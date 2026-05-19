# C API — `flux_fracture.h`

Single-header C99 library. No dependencies. No heap allocation.

## Installation

Download [flux_fracture.h](https://github.com/SuperInstance/flux-fracture-c). In exactly one source file:

```c
#define FRACTURE_IMPLEMENTATION
#include "flux_fracture.h"
```

All other files just `#include "flux_fracture.h"` without the define.

## Quick Start

```c
#include <stdio.h>
#include <math.h>

#define FRACTURE_IMPLEMENTATION
#include "flux_fracture.h"

int main() {
    /* 8 independent constraints */
    frac_edge edges[8];
    for (int i = 0; i < 8; i++)
        edges[i] = (frac_edge){i, i};

    /* Fracture */
    frac_result result = frac_fracture_from_edges(edges, 8, 8, 8);
    printf("Blocks: %d, Largest: %d, Speedup: %.1fx\n",
           result.n_blocks, result.largest_block, result.speedup_potential);

    /* Check some values */
    double lo[] = {-40, 0, 0.5, 0, 10, 800, 110, 0.1};
    double hi[] = {150, 100, 10.0, 5, 95, 3600, 240, 15};
    double vals[] = {151, 50, 3.2, 6, 50, 2000, 240, NAN};

    uint8_t mask = 0;
    for (int i = 0; i < 8; i++) {
        uint8_t bit = (isnan(vals[i]) || vals[i] < lo[i] || vals[i] > hi[i]) ? 1 : 0;
        mask |= (bit << i);
    }
    printf("Error mask: %02x\n", mask);

    /* Coalesce block masks */
    uint64_t block_masks[8] = {1, 0, 1, 0, 1, 0, 1, 0};
    frac_coalesce_result cr = frac_coalesce(block_masks, result.blocks, 8, 8);

    /* Verify */
    bool ok = frac_coalesce_verify(cr.error_mask, 0x55, 8);

    frac_result_free(&result);
    return 0;
}
```

## Data Structures

| Type | Description |
|------|-------------|
| `frac_edge` | Bipartite edge: `{constraint_idx, dim_idx}` |
| `frac_block` | Independent block: constraint/dimension index arrays |
| `frac_result` | Fracture result: blocks, stats, speedup |
| `frac_coalesce_result` | Coalesced error mask with verification |
| `frac_adjacency` | Bipartite adjacency matrix |
| `frac_delta` | Change between two fracture results |

## Functions

### Graph Construction

| Function | Description |
|----------|-------------|
| `frac_graph_build(edges, n_edges, n_c, n_d)` | Build adjacency from edge list |
| `frac_graph_from_masks(masks, n_c, max_dims)` | Build from per-constraint dim masks |

### Fracture

| Function | Description |
|----------|-------------|
| `frac_fracture(adj, n_c, n_d)` | BFS connected components → blocks |
| `frac_fracture_from_edges(edges, n_e, n_c, n_d)` | One-step: edges → result |
| `frac_result_free(result)` | Free block memory |

### Coalesce

| Function | Description |
|----------|-------------|
| `frac_coalesce(masks, blocks, n_blocks, n_c)` | Bitwise OR coalescence |
| `frac_coalesce_verify(coalesced, expected, n_c)` | Verify correctness |

### Adaptive

| Function | Description |
|----------|-------------|
| `frac_adaptive_init(adj)` | Initialize adaptive fracture |
| `frac_adaptive_update(adj)` | Re-fracture if changed |
| `frac_adaptive_free()` | Release resources |

## Memory

- **Stack allocation only** in the hot path.
- BFS queue is fixed-size: `uint32_t queue[MAX_CONSTRAINTS]`.
- Blocks allocated with `malloc` — call `frac_result_free()` when done.
- Total working memory for 256 constraints: ~2KB.

## Invariants

1. **Zero false negatives** — bitwise OR coalescence is provably correct.
2. **NaN always violates** — `isnan()` check before comparison.
3. **C99 compatible** — works with any C99 compiler, no extensions.
4. **Single header** — `#define FRACTURE_IMPLEMENTATION` + `#include`.
