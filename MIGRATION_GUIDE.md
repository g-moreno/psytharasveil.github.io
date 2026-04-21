# React + TypeScript Migration Guide
## Psythara's Veil Character Sheet

This guide documents the complete process to convert from vanilla JavaScript to React + TypeScript, including Foundry VTT integration.

---

## Table of Contents
1. [Phase 1: Project Setup](#phase-1-project-setup)
2. [Phase 2: Data Layer & TypeScript Interfaces](#phase-2-data-layer--typescript-interfaces)
3. [Phase 3: Core Calculation Hooks](#phase-3-core-calculation-hooks)
4. [Phase 4: Component Architecture](#phase-4-component-architecture)
5. [Phase 5: Foundry VTT Integration](#phase-5-foundry-vtt-integration)
6. [Phase 6: Class-by-Class Migration](#phase-6-class-by-class-migration)
7. [Troubleshooting & Tips](#troubleshooting--tips)

---

## Phase 1: Project Setup

### Step 1.1: Initialize React + Vite + TypeScript
```bash
cd c:\Users\gabri\Documents\VS Code\PsytharasVeil
npm create vite@latest psythara-veil-react -- --template react-ts
cd psythara-veil-react
npm install
```

### Step 1.2: Install Required Dependencies
```bash
npm install axios react-hook-form zustand
# axios: HTTP client for Foundry API bridge
# react-hook-form: Form state management (stat inputs, selections)
# zustand: Lightweight state management (character data)
```

### Step 1.3: Project Structure
Create the following directory structure:
```
src/
├── components/
│   ├── CharacterIdentity.tsx
│   ├── StatBuyControl.tsx
│   ├── Loadout.tsx
│   ├── ACCalculator.tsx
│   ├── DiceRoller.tsx
│   ├── FeatureDisplay.tsx
│   ├── CharacterPreview.tsx
│   └── index.ts
├── hooks/
│   ├── useCharacter.ts
│   ├── useStatCalculations.ts
│   ├── useACCalculator.ts
│   ├── useFoundryRoller.ts
│   ├── useLocalStorage.ts
│   └── index.ts
├── data/
│   ├── types.ts              (TypeScript interfaces)
│   ├── classes.ts            (CLASS data)
│   ├── species.ts            (SPECIES data)
│   ├── roles.ts              (ROLES data)
│   └── constants.ts          (STAT_COSTS, BASE_STATS, etc)
├── services/
│   ├── foundryApi.ts         (Foundry API integration)
│   └── calculations.ts       (Game logic: AC, HP, mods, etc)
├── styles/
│   ├── index.css             (Global styles from original style.css)
│   └── variables.css         (CSS variables)
├── utils/
│   └── helpers.ts            (Utility functions)
├── App.tsx
└── main.tsx
```

### Step 1.4: Copy & Adapt Styling
Copy `style.css` from original project:
```bash
cp ../Psytharas-Veil-Character-Sheet/style.css src/styles/index.css
```

Refactor CSS for React component naming (optional - can use BEM or CSS-in-JS):
- Keep existing class names where possible for minimal refactoring
- Consider moving to Tailwind or styled-components later

---

## Phase 2: Data Layer & TypeScript Interfaces

### Step 2.1: Create `data/types.ts`
Define all TypeScript interfaces extracted from the original `script.js` object structures:

```typescript
// data/types.ts

export type StatKey = 'STR' | 'AGI' | 'CON' | 'INT' | 'WIS' | 'CHA';

export interface StatBonuses {
  [key: string]: number;
  any?: [string, string];
  choice2?: string[];
  amt?: number[];
}

export interface NaturalArmor {
  base: number;
  uses: 'DEX' | 'STR';
}

export interface Species {
  key: string;
  bonuses: StatBonuses;
  naturalArmor?: NaturalArmor;
  note: string;
}

export interface WeaponSpecial {
  name: string;
  text: string;
}

export interface Weapon {
  name: string;
  dmg: string;
  type: string;
  special: WeaponSpecial;
}

export interface Armor {
  name: string;
  type: 'Light' | 'Medium' | 'Heavy' | 'Natural';
  ac: number;
  perk: string;
}

export interface Class {
  key: string;
  role: string;
  startHP: number;
  weapons: Weapon[];
  armor: Armor[];
  utils: string[];
}

export interface Character {
  name: string;
  level: 1 | 2 | 3 | 4 | 5;
  species: string; // key reference
  clazz: string;   // key reference
  baseStats: Record<StatKey, number>;
  finalStats: Record<StatKey, number>;
  weapon: string;
  armor: string;
  utilA: string;
  utilB: string;
  useNamedArmor: boolean;
  hasShield: boolean;
  profilePic: string | null;
  lockedStats: boolean;
  speciesChoices?: {
    any1?: string;
    any2?: string;
    choice2?: string;
  };
}

export interface ACResult {
  ac: number;
  notes: string;
  armType: string;
}

export interface Role {
  hpPerLevel: number;
}
```

### Step 2.2: Create `data/constants.ts`
```typescript
// data/constants.ts
import { StatKey } from './types';

export const BASE_STATS: StatKey[] = ['STR', 'AGI', 'CON', 'INT', 'WIS', 'CHA'];

export const STAT_COSTS: Record<number, number> = {
  8: 0, 9: 1, 10: 2, 11: 3, 12: 4, 13: 5, 14: 7, 15: 9
};

export const ROLES: Record<string, { hpPerLevel: number }> = {
  "Tank": { hpPerLevel: 7 },
  "Balanced Frontline": { hpPerLevel: 7 },
  "Anti-Supernatural Defender": { hpPerLevel: 7 },
  "Melee DPS": { hpPerLevel: 6 },
  // ... etc (copy all from original ROLES)
};
```

### Step 2.3: Create `data/species.ts`
Extract SPECIES array from original script, convert to TypeScript:
```typescript
// data/species.ts
import { Species } from './types';

export const SPECIES: Species[] = [
  {
    key: "Human",
    bonuses: { any: ["Any", "Any"], amt: [1, 1] },
    naturalArmor: undefined,
    note: "Versatility — 1/long rest, advantage on any skill check.",
  },
  // ... copy all remaining species from original script.js
];
```

### Step 2.4: Create `data/classes.ts`
Extract CLASSES array, convert to TypeScript:
```typescript
// data/classes.ts
import { Class } from './types';

export const CLASSES: Class[] = [
  {
    key: "Juggernaut",
    role: "Tank",
    startHP: 14,
    weapons: [
      {
        name: "Titan Maul",
        dmg: "1d10+3",
        type: "Melee",
        special: { name: "Shockwave Slam", text: "Cone knockdown (once/short rest)." },
      },
      // ... copy remaining weapons
    ],
    armor: [
      // ... copy armor
    ],
    utils: [
      // ... copy utilities
    ],
  },
  // ... copy all remaining classes
];
```

### Step 2.5: Create `data/roles.ts`
```typescript
// data/roles.ts
export const ROLES = {
  // Directly export all roles from original ROLES object
};
```

---

## Phase 3: Core Calculation Hooks

### Step 3.1: Create `hooks/useCharacter.ts`
Global character state management using Zustand:

```typescript
// hooks/useCharacter.ts
import { create } from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';
import { Character, StatKey } from '../data/types';

const initialCharacter: Character = {
  name: '',
  level: 1,
  species: 'Human',
  clazz: 'Soldier',
  baseStats: { STR: 8, AGI: 8, CON: 8, INT: 8, WIS: 8, CHA: 8 },
  finalStats: { STR: 8, AGI: 8, CON: 8, INT: 8, WIS: 8, CHA: 8 },
  weapon: '',
  armor: '',
  utilA: '',
  utilB: '',
  useNamedArmor: true,
  hasShield: false,
  profilePic: null,
  lockedStats: false,
  speciesChoices: {},
};

interface CharacterStore {
  character: Character;
  updateCharacter: (updates: Partial<Character>) => void;
  updateStat: (stat: StatKey, value: number) => void;
  updateSpeciesChoice: (key: string, value: string) => void;
  lockStats: () => void;
  resetCharacter: () => void;
  load: () => void;
  save: () => void;
}

export const useCharacterStore = create<CharacterStore>()(
  subscribeWithSelector((set, get) => ({
    character: initialCharacter,

    updateCharacter: (updates) => {
      set((state) => ({
        character: { ...state.character, ...updates },
      }));
      get().save();
    },

    updateStat: (stat: StatKey, value: number) => {
      const newValue = Math.max(8, Math.min(15, value));
      set((state) => ({
        character: {
          ...state.character,
          baseStats: { ...state.character.baseStats, [stat]: newValue },
        },
      }));
      get().save();
    },

    updateSpeciesChoice: (key: string, value: string) => {
      set((state) => ({
        character: {
          ...state.character,
          speciesChoices: { ...state.character.speciesChoices, [key]: value },
        },
      }));
      get().save();
    },

    lockStats: () => {
      set((state) => ({
        character: { ...state.character, lockedStats: !state.character.lockedStats },
      }));
      get().save();
    },

    resetCharacter: () => {
      set({ character: initialCharacter });
      localStorage.removeItem('psythara_character');
    },

    load: () => {
      const saved = localStorage.getItem('psythara_character');
      if (saved) {
        try {
          set({ character: JSON.parse(saved) });
        } catch (e) {
          console.error('Failed to load character:', e);
        }
      }
    },

    save: () => {
      const { character } = get();
      localStorage.setItem('psythara_character', JSON.stringify(character));
    },
  }))
);
```

### Step 3.2: Create `hooks/useStatCalculations.ts`
Calculation logic for ability modifiers, point buy costs, final stats:

```typescript
// hooks/useStatCalculations.ts
import { useMemo } from 'react';
import { Character, StatKey } from '../data/types';
import { BASE_STATS, STAT_COSTS } from '../data/constants';
import { SPECIES } from '../data/species';

export function useStatCalculations(character: Character) {
  const finalStats = useMemo(() => {
    const base = character.baseStats;
    const species = SPECIES.find((s) => s.key === character.species);
    if (!species) return base;

    const out = { ...base };
    const bonuses = species.bonuses || {};

    // Apply fixed bonuses
    Object.entries(bonuses).forEach(([key, value]) => {
      if (key !== 'any' && key !== 'choice2' && key !== 'amt') {
        out[key as StatKey] = (out[key as StatKey] || 0) + (value as number);
      }
    });

    // Apply flexible bonuses
    if (bonuses.any && character.speciesChoices) {
      if (character.speciesChoices.any1) {
        out[character.speciesChoices.any1 as StatKey] += 1;
      }
      if (character.speciesChoices.any2) {
        out[character.speciesChoices.any2 as StatKey] += 1;
      }
    }

    if (bonuses.choice2 && character.speciesChoices?.choice2) {
      out[character.speciesChoices.choice2 as StatKey] = 
        (out[character.speciesChoices.choice2 as StatKey] || 0) + 2;
    }

    return out;
  }, [character.baseStats, character.species, character.speciesChoices]);

  const abilityMod = (score: number): number => {
    return Math.floor((score - 10) / 2);
  };

  const getPointCost = (score: number): number => {
    if (score >= 8 && score <= 13) return score - 8;
    if (score === 14) return 7;
    if (score === 15) return 9;
    return 0;
  };

  const pointsSpent = useMemo(() => {
    return BASE_STATS.reduce((sum, stat) => sum + getPointCost(character.baseStats[stat]), 0);
  }, [character.baseStats]);

  const pointsLeft = 27 - pointsSpent;

  return { finalStats, abilityMod, getPointCost, pointsSpent, pointsLeft };
}
```

### Step 3.3: Create `hooks/useACCalculator.ts`
```typescript
// hooks/useACCalculator.ts
import { Character, ACResult } from '../data/types';
import { SPECIES } from '../data/species';
import { CLASSES } from '../data/classes';
import { useStatCalculations } from './useStatCalculations';

export function useACCalculator(character: Character): ACResult {
  const { finalStats, abilityMod } = useStatCalculations(character);
  const species = SPECIES.find((s) => s.key === character.species);
  const clazz = CLASSES.find((c) => c.key === character.clazz);

  if (!clazz) return { ac: 10, notes: '', armType: '—' };

  const agiMod = abilityMod(finalStats.AGI);
  const armor = clazz.armor.find((a) => a.name === character.armor) || clazz.armor[0];
  const notes: string[] = [];
  let ac = 10;
  let armType = armor?.type || '—';

  if (character.useNamedArmor && armor) {
    ac = armor.ac;
    notes.push(armor.perk);
  } else {
    if (armType === 'Light') ac = 12 + agiMod;
    else if (armType === 'Medium') ac = 14 + Math.min(2, Math.max(0, agiMod));
    else if (armType === 'Heavy') ac = 16;
  }

  if (species?.naturalArmor?.base) {
    const nat = species.naturalArmor.base + (species.naturalArmor.uses === 'DEX' ? agiMod : 0);
    if (nat > ac) {
      notes.push(`Natural armor active (${nat})`);
      ac = nat;
      armType = 'Natural';
    }
  }

  if (character.hasShield) ac += 1;

  return { ac, notes: notes.filter(Boolean).join(' '), armType };
}
```

### Step 3.4: Create `services/calculations.ts`
Game logic utilities:

```typescript
// services/calculations.ts
import { Character, StatKey } from '../data/types';
import { ROLES } from '../data/roles';
import { CLASSES } from '../data/classes';

export function calculateHP(character: Character, finalStats: Record<StatKey, number>): {
  startHP: number;
  hpPerLevel: number;
  totalHP: number;
} {
  const clazz = CLASSES.find((c) => c.key === character.clazz);
  if (!clazz) return { startHP: 0, hpPerLevel: 0, totalHP: 0 };

  const role = ROLES[clazz.role];
  const conMod = Math.floor((finalStats.CON - 10) / 2);
  const startHP = clazz.startHP + conMod;
  const hpPerLevel = (role?.hpPerLevel || 5) + conMod;
  const totalHP = startHP + (character.level - 1) * hpPerLevel;

  return { startHP, hpPerLevel, totalHP };
}
```

---

## Phase 4: Component Architecture

### Step 4.1: Create `components/CharacterIdentity.tsx`
```typescript
// components/CharacterIdentity.tsx
import React from 'react';
import { useCharacterStore } from '../hooks/useCharacter';
import { SPECIES, CLASSES } from '../data';

export function CharacterIdentity() {
  const character = useCharacterStore((s) => s.character);
  const updateCharacter = useCharacterStore((s) => s.updateCharacter);

  return (
    <div className="panel">
      <h2>// Identity</h2>
      {/* Profile picture frame */}
      <div className="profile-pic-container">
        <div className="profile-pic-frame">
          {character.profilePic && <img src={character.profilePic} alt="Profile" />}
        </div>
        <div className="name-input-large">
          <label>Character Name</label>
          <input
            type="text"
            value={character.name}
            onChange={(e) => updateCharacter({ name: e.target.value })}
            placeholder="e.g., Kessa Vox"
          />
          <hr />
          <div className="flex">
            <div>
              <label>Level</label>
              <input
                type="number"
                min="1"
                max="5"
                value={character.level}
                onChange={(e) => updateCharacter({ level: parseInt(e.target.value) as 1 | 2 | 3 | 4 | 5 })}
              />
            </div>
            <label htmlFor="profile-pic-input" className="btn secondary">
              Upload Pic
            </label>
            <input
              type="file"
              id="profile-pic-input"
              accept="image/*"
              onChange={(e) => {
                // Handle file upload to base64 or URL
              }}
            />
          </div>
        </div>
      </div>

      {/* Species & Class selectors */}
      <div className="row">
        <div>
          <label>Species</label>
          <select
            value={character.species}
            onChange={(e) => updateCharacter({ species: e.target.value })}
          >
            {SPECIES.map((s) => (
              <option key={s.key} value={s.key}>
                {s.key}
              </option>
            ))}
          </select>
        </div>
        <div>
          <label>Class</label>
          <select
            value={character.clazz}
            onChange={(e) => updateCharacter({ clazz: e.target.value })}
          >
            {CLASSES.map((c) => (
              <option key={c.key} value={c.key}>
                {c.key}
              </option>
            ))}
          </select>
        </div>
      </div>
      {/* HP display, Focus, etc */}
    </div>
  );
}
```

### Step 4.2: Create `components/StatBuyControl.tsx`
```typescript
// components/StatBuyControl.tsx
import React from 'react';
import { useCharacterStore } from '../hooks/useCharacter';
import { useStatCalculations } from '../hooks/useStatCalculations';
import { BASE_STATS } from '../data/constants';

export function StatBuyControl() {
  const character = useCharacterStore((s) => s.character);
  const updateStat = useCharacterStore((s) => s.updateStat);
  const lockStats = useCharacterStore((s) => s.lockStats);
  const { finalStats, abilityMod, pointsLeft } = useStatCalculations(character);

  return (
    <div className="panel">
      <h2>// Attributes (Point Buy)</h2>
      <div className="muted">Start with 27 points. All stats begin at 8. Cost increases for higher scores.</div>
      <div className="points-left" style={{ color: pointsLeft < 0 ? '#ff6b6b' : '#9efff4' }}>
        Points Left: {pointsLeft}
      </div>

      <div className="stat-cost-grid">
        {BASE_STATS.map((stat) => (
          <div key={stat} className="stat-cost">
            <h3>{stat}</h3>
            <input
              type="number"
              min="8"
              max="15"
              value={character.baseStats[stat]}
              onChange={(e) => updateStat(stat, parseInt(e.target.value))}
              disabled={character.lockedStats}
            />
            <div className="stat-cost-controls">
              <button
                className="btn secondary"
                onClick={() => updateStat(stat, character.baseStats[stat] - 1)}
                disabled={character.lockedStats}
              >
                -
              </button>
              <button
                className="btn secondary"
                onClick={() => updateStat(stat, character.baseStats[stat] + 1)}
                disabled={character.lockedStats}
              >
                +
              </button>
            </div>
            <div className="muted">
              mod: <span className="mono">{abilityMod(finalStats[stat])}</span>
            </div>
          </div>
        ))}
      </div>

      <button className="btn" onClick={() => lockStats()}>
        {character.lockedStats ? 'Unlock Stats' : 'Lock Stats'}
      </button>
    </div>
  );
}
```

### Step 4.3: Create Remaining Components
Create similar component structure for:
- `DiceRoller.tsx`
- `Loadout.tsx` (Weapon/Armor selection)
- `ACCalculator.tsx`
- `FeatureDisplay.tsx`
- `CharacterPreview.tsx`

Each component should:
- Use `useCharacterStore` to read/write character data
- Use relevant calculation hooks
- Be focused on a single responsibility
- Handle its own local state only (inputs, dropdowns)

---

## Phase 5: Foundry VTT Integration

### Step 5.1: Install Foundry API Bridge Module
Users must install in their Foundry instance:
1. Download: https://github.com/alexivenkov/foundry-api-bridge-module
2. Extract to `Data/modules/foundry-api-bridge-module`
3. Enable in Foundry module settings

### Step 5.2: Create `services/foundryApi.ts`
```typescript
// services/foundryApi.ts
import axios from 'axios';

const FOUNDRY_API_BASE = 'http://localhost:30000/api';

interface FoundryRollRequest {
  formula: string;
  speaker?: { name: string };
}

interface FoundryRollResult {
  total: number;
  formula: string;
  rolls: number[];
}

export async function rollInFoundry(formula: string): Promise<FoundryRollResult | null> {
  try {
    const response = await axios.post(`${FOUNDRY_API_BASE}/roll`, {
      formula,
      speaker: { name: 'Character Sheet' },
    } as FoundryRollRequest);

    return response.data as FoundryRollResult;
  } catch (error) {
    console.warn('Foundry not available, rolling locally', error);
    return null;
  }
}

export function buildDiceFormula(
  sides: number,
  count: number,
  advantage?: boolean,
  disadvantage?: boolean
): string {
  if (sides === 20 && advantage) return '2d20kh';
  if (sides === 20 && disadvantage) return '2d20kl';
  return `${count}d${sides}`;
}
```

### Step 5.3: Create `hooks/useFoundryRoller.ts`
```typescript
// hooks/useFoundryRoller.ts
import { useState } from 'react';
import { rollInFoundry, buildDiceFormula } from '../services/foundryApi';

export interface RollResult {
  total: number;
  formula: string;
  isLocal: boolean; // true if rolled locally (Foundry unavailable)
}

export function useFoundryRoller() {
  const [isLoading, setIsLoading] = useState(false);
  const [lastResult, setLastResult] = useState<RollResult | null>(null);
  const [error, setError] = useState<string | null>(null);

  const roll = async (
    sides: number,
    count: number = 1,
    advantage: boolean = false,
    disadvantage: boolean = false
  ): Promise<RollResult | null> => {
    setIsLoading(true);
    setError(null);

    const formula = buildDiceFormula(sides, count, advantage, disadvantage);

    try {
      // Try Foundry first
      const foundryResult = await rollInFoundry(formula);
      if (foundryResult) {
        const result: RollResult = {
          total: foundryResult.total,
          formula: foundryResult.formula,
          isLocal: false,
        };
        setLastResult(result);
        return result;
      }

      // Fallback to local roll
      const result = rollLocal(formula);
      setLastResult(result);
      return result;
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Unknown error';
      setError(message);
      return null;
    } finally {
      setIsLoading(false);
    }
  };

  return { roll, lastResult, isLoading, error };
}

function rollLocal(formula: string): RollResult {
  // Parse formula and execute locally
  // Example: "2d20kh" -> roll 2d20, keep highest
  const result = parseAndRoll(formula);
  return {
    total: result,
    formula,
    isLocal: true,
  };
}

function parseAndRoll(formula: string): number {
  // Simple dice parser
  // Support: XdY, XdYkh, XdYkl
  const match = formula.match(/^(\d+)d(\d+)([kl][hl])?$/);
  if (!match) return 0;

  const [, countStr, sidesStr, modifier] = match;
  const count = parseInt(countStr);
  const sides = parseInt(sidesStr);

  const rolls = Array(count)
    .fill(0)
    .map(() => Math.floor(Math.random() * sides) + 1);

  if (modifier === 'kh') return Math.max(...rolls); // keep highest
  if (modifier === 'kl') return Math.min(...rolls); // keep lowest
  return rolls.reduce((a, b) => a + b, 0);
}
```

### Step 5.4: Update `components/DiceRoller.tsx`
```typescript
// components/DiceRoller.tsx
import React, { useState } from 'react';
import { useFoundryRoller } from '../hooks/useFoundryRoller';

export function DiceRoller() {
  const [selectedDice, setSelectedDice] = useState(20);
  const [numRolls, setNumRolls] = useState(1);
  const [advantage, setAdvantage] = useState(false);
  const [disadvantage, setDisadvantage] = useState(false);
  const { roll, lastResult, isLoading } = useFoundryRoller();

  const handleRoll = async () => {
    await roll(selectedDice, numRolls, advantage, disadvantage);
  };

  return (
    <div className="panel">
      <h2>// Dice Roller</h2>
      <div className="dice-container">
        {[4, 6, 8, 10, 12, 20, 100].map((sides) => (
          <button
            key={sides}
            className={`dice-btn ${selectedDice === sides ? 'selected-die' : ''}`}
            onClick={() => setSelectedDice(sides)}
          >
            D{sides}
          </button>
        ))}
      </div>

      <div className="dice-roll-controls">
        <label>Rolls:</label>
        <input
          type="number"
          value={numRolls}
          onChange={(e) => setNumRolls(Math.max(1, Math.min(10, parseInt(e.target.value))))}
          min="1"
          max="10"
        />
        <button
          className={`btn secondary ${advantage ? '' : ''}`}
          onClick={() => {
            setAdvantage(!advantage);
            setDisadvantage(false);
          }}
        >
          Advantage
        </button>
        <button
          className={`btn secondary ${disadvantage ? '' : ''}`}
          onClick={() => {
            setDisadvantage(!disadvantage);
            setAdvantage(false);
          }}
        >
          Disadvantage
        </button>
        <button className="btn" onClick={handleRoll} disabled={isLoading}>
          {isLoading ? 'Rolling...' : 'Roll Dice'}
        </button>
      </div>

      {lastResult && (
        <div className="dice-results">
          <div>
            D{selectedDice}: {lastResult.total}
            {lastResult.isLocal && <span className="muted"> (local)</span>}
            {!lastResult.isLocal && <span className="muted"> (Foundry)</span>}
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Phase 6: Class-by-Class Migration

### Strategy: Migrate One Class at a Time

### Step 6.1: Verify Base Components Work
1. Ensure `CharacterIdentity`, `StatBuyControl`, `Loadout`, `ACCalculator` are fully functional
2. Test with simple class (e.g., **Soldier**)
3. Verify all calculations match original implementation

### Step 6.2: Start with Soldier Class
```typescript
// Verify these calculations:
// - HP calculation
// - AC calculation
// - Weapon/Armor options rendering
// - Utility display
// - Ability modifier calculations
```

### Step 6.3: Compare Original vs React
1. Open original in browser DevTools
2. Open React version in separate tab
3. Create identical character in both
4. Compare all computed values (HP, AC, mods, etc)
5. Use console to verify calculations

### Step 6.4: Move to Next Class After Verification
Once one class is verified:
1. Test with 2-3 other diverse classes (Tank, Healer, Caster)
2. Ensure species bonuses work correctly
3. Verify point buy constraints
4. Test edge cases (max level, min stats, shield bonus)

### Step 6.5: Full Class List (In Recommended Order)
```
1. Soldier (balanced, good baseline)
2. Juggernaut (tank, simple)
3. Warp Mage (caster, spellcasting mechanics)
4. Medic (healer, buff mechanics)
5. Bounty Hunter (ranged DPS)
6. Smuggler (rogue, skill-focused)
7. [Continue for remaining 14 classes]
```

---

## Phase 5: Integration Checklist

### Pre-Launch Checklist
- [ ] All 20 classes tested with identical character creation
- [ ] Foundry API bridge integration tested
- [ ] localStorage persistence working
- [ ] Export JSON working
- [ ] Print functionality working
- [ ] Profile picture upload working
- [ ] All stat calculations verified
- [ ] AC formula matches original (both named and formula-based)
- [ ] HP calculations accurate
- [ ] Point buy constraints enforced
- [ ] Species bonuses apply correctly

### Foundry Setup Checklist
- [ ] User has `foundry-api-bridge-module` installed
- [ ] Foundry API listening on `http://localhost:30000`
- [ ] Dice rolls appear in Foundry chat
- [ ] Roll formulas correctly built (2d20kh for advantage, etc)
- [ ] Fallback to local rolling if Foundry unavailable

---

## Troubleshooting & Tips

### Common Issues

#### "Foundry connection failed"
- **Cause**: Foundry API bridge not installed or not running
- **Fix**: Install module, enable in Foundry, restart
- **Fallback**: App automatically rolls locally

#### "Calculations don't match original"
- **Cause**: Rounding differences in ability mods or AC formulas
- **Fix**: Use original script.js as reference, trace through calculation step-by-step
- **Debug**: Log intermediate values in calculation hooks

#### "Stats not saving to localStorage"
- **Cause**: `useCharacterStore.save()` not called or localStorage quota exceeded
- **Fix**: Ensure `updateCharacter` calls `save()`, check localStorage quota
- **Verify**: Check DevTools > Application > Local Storage

#### "Species bonuses not applying"
- **Cause**: `speciesChoices` not initialized or UI not rendering
- **Fix**: Initialize empty object in `Character` type, render choice dropdowns for species with flexible bonuses
- **Test**: Create Human or Cyborgian character, verify stat bonuses

### Performance Tips
- Use `useMemo` for expensive calculations (AC, HP, final stats)
- Memoize components with `React.memo` to prevent unnecessary renders
- Use Zustand subscriptions to update only affected components

### Testing Strategy
1. **Unit tests**: Test calculation functions (`abilityMod`, `getPointCost`, `calculateHP`)
2. **Integration tests**: Test character creation flow with all classes
3. **Snapshot tests**: Compare original vs React for 10 random character combinations
4. **Manual testing**: Create characters, verify all displays, test Foundry integration

### Debugging Tools
```typescript
// Log character state on change
useCharacterStore.subscribe(
  (state) => console.log('Character updated:', state.character)
);

// Compare calculations
const original = originalCalculationFunction(data);
const react = useCalculationHook(data);
console.assert(original === react, `Mismatch: ${original} vs ${react}`);
```

---

## Commit Strategy

After each phase, commit to Git:
```bash
git add .
git commit -m "Phase 1: React + Vite setup"
git commit -m "Phase 2: TypeScript data layer"
git commit -m "Phase 3: Calculation hooks"
git commit -m "Phase 4: Component architecture (base)"
git commit -m "Phase 5: Foundry VTT integration"
git commit -m "Phase 6: Migrate Soldier class (verified)"
git commit -m "Phase 6: Migrate [Class Name] (verified)"
# ... etc
```

---

## Timeline Estimate

- **Phase 1**: 1-2 hours (setup, dependencies)
- **Phase 2**: 3-4 hours (extract & type data)
- **Phase 3**: 4-5 hours (implement hooks)
- **Phase 4**: 6-8 hours (build & test components)
- **Phase 5**: 2-3 hours (Foundry integration)
- **Phase 6**: 3-5 hours per class batch (1-2 weeks for all 20)

**Total: 3-4 weeks** for complete migration with full testing

---

## Next Steps

1. Run: `npm create vite@latest psythara-veil-react -- --template react-ts`
2. Follow Phase 1 setup
3. Begin Phase 2: Extract data to TypeScript
4. Reference this guide as you build each phase

Good luck with the migration! 🚀

