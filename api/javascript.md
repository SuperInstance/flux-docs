# JavaScript API — `@flux/check`

```bash
npm install @flux/check
```

## Core

```js
import { ConstraintEngine, Severity } from "@flux/check";

const engine = new ConstraintEngine();
engine.addConstraint("coolant_temp", -40, 150);
engine.addConstraint("pressure", 0, 100);

const result = engine.check({ coolant_temp: 151, pressure: 50 });
// result.errorMask === 0b001
// result.severity === Severity.CAUTION
// result.violatedNames === ["coolant_temp"]
```

### `ConstraintEngine`

| Method | Returns | Description |
|--------|---------|-------------|
| `addConstraint(name, lo, hi, dims?)` | — | Add a named constraint |
| `check(values)` | `CheckResult` | Check all constraints |
| `use(strategy)` | — | Enable "fracture" or "sediment" |
| `fracture()` | `FractureResult` | Split into independent blocks |
| `addSedimentLayer(context, corrections)` | — | Add correction layer |
| `checkWithSediment(values)` | `SedimentResult` | Check with sediment |

### `CheckResult`

| Field | Type | Description |
|-------|------|-------------|
| `errorMask` | `number` | Bitmask of violations |
| `violations` | `Uint8Array` | Per-constraint violation array |
| `severity` | `Severity` | PASS / CAUTION / WARNING / CRITICAL |
| `violationCount` | `number` | Number violated |
| `violatedNames` | `string[]` | Names of violated constraints |

## Exact Checking (`src/core.ts`)

```js
import { checkExact, checkOne, errorMask, severityFromMask } from "@flux/check";

// Batch
const violations = checkExact(
    [151, 50, 3.2, 6, 50, 2000, 240, NaN],
    [[-40, 150], [0, 100], [0.5, 10], [0, 5],
     [10, 95], [800, 3600], [110, 240], [0.1, 15]]
);
const mask = errorMask(violations);  // 0b10001001

// Single
checkOne(NaN, 0, 100);  // 1 (NaN always violates)

// Severity from mask
severityFromMask(0b10001001);  // Severity.WARNING
```

| Function | Description |
|----------|-------------|
| `checkExact(values, bounds)` | Batch exact check. NaN always violates. Bounds `<=`. |
| `checkOne(value, lo, hi)` | Single value check → `0 \| 1` |
| `errorMask(violations)` | Bitmask from violation array |
| `severityFromMask(mask)` | PASS / CAUTION / WARNING / CRITICAL |

## Fracture-Coalesce (`src/fracture.ts`)

```js
import { DependencyGraph, fracture, coalesce } from "@flux/check";

const graph = DependencyGraph.fromMasks([[0], [1], [2, 3], [4]]);
const result = fracture(graph);
// result.nBlocks === 4, result.speedupPotential === 4.0

const final = coalesce([0b0001, 0b0000, 0b0100, 0b0000]);  // 0b0101
```

| Function | Description |
|----------|-------------|
| `DependencyGraph.fromMasks(masks)` | Build bipartite graph |
| `fracture(graph)` | BFS connected components |
| `coalesce(blockMasks)` | Bitwise OR coalescence |

## Sediment (`src/sediment.ts`)

```js
import { SedimentStack } from "@flux/check";

const stack = new SedimentStack();
stack.addLayer("Sensor dropout", [
    { constraint: "pressure", action: "violate", ifValue: 0 }
]);
```

| Class/Method | Description |
|-------------|-------------|
| `SedimentStack` | Immutable correction layers |
| `addLayer(context, corrections)` | Add correction |
| `apply(errorMask, names, values, defs)` | Simplified post-processing |

## Invariants

1. **Zero false negatives** — always detected.
2. **NaN always violates** — `Number.isNaN()` check, no opt-in.
3. **Bounds checked with `<=`** — boundary values pass.
4. **No external dependencies** — pure TypeScript, runs anywhere (Node, browser, Deno, Bun).

## Build & Test

```bash
npm install
npx tsc
node tests/core.test.mjs
```
