# The Error Mask

The fundamental data structure of FLUX. Everything else is built on top of this.

## One Bit Per Constraint

8 constraints = 1 byte. 64 constraints = 1 `uint64`. No arrays. No heap. No indirection.

```
Constraint:  7    6    5    4    3    2    1    0
Value:       151  50   3.2  6    NaN  2000 240  0.1
Bound:       150  100  10   5    15   3600 240  15
Result:      1    0    0    1    1    0    0    0

Error mask:  0b00011001 = 0x19 = 25
```

Bit `i` = 1 means constraint `i` was violated. Bit `i` = 0 means it passed. That's it.

## Why Not a List of Booleans?

```python
violations = [True, False, False, True, True, False, False, False]
```

This is 8 Python objects. Each is a heap allocation with a reference count. To check "did anything fail?" you iterate. To check "did constraint 3 fail?" you index. To merge two results, you zip and OR elementwise. Every operation is O(n) with overhead.

The error mask does all of these in one CPU instruction:

| Operation | Boolean list | Error mask |
|-----------|:------------|:----------|
| Any violation? | `any(violations)` | `mask != 0` |
| Constraint 3? | `violations[3]` | `mask & (1 << 3)` |
| Count violations | `sum(violations)` | `popcount(mask)` |
| Merge two results | `[a or b for a, b in zip(x, y)]` | `mask_a \| mask_b` |
| All passed? | `not any(violations)` | `mask == 0` |

The bitwise OR merge is the key that makes [fracture-coalesce](fracture-coalesce.md) work. You can't OR two lists of booleans and get a correct result without allocating a third list. With masks, it's one instruction.

## Why Not a Set of Violated Indices?

```python
violated = {0, 3, 4}  # which constraints failed
```

Better than a list — checking membership is O(1), merging is union. But:

- A set is a hash table. Heap-allocated, variable-size, cache-unfriendly.
- For 8 constraints, a `Set<int>` is ~80 bytes. The error mask is 1 byte.
- On GPU, there are no hash tables. There *are* bitwise OR instructions.
- You can't do `popcount` on a set to count violations. You call `.len()` which walks the internal structure.

## Why Not a Result Enum?

```rust
enum CheckResult {
    Pass,
    Fail { violated: Vec<usize>, details: Vec<String> },
}
```

This is what most constraint libraries return. It's ergonomic. It's also:

- Variable-size (heap allocation)
- Branching on every check (enum discriminant)
- Impossible to merge with one instruction
- Completely unsuitable for GPU

The error mask is the *answer*, not a description of the computation. You don't need to know *why* constraint 3 failed in the hot path. You need one bit: did it fail? If you need details, look them up after — outside the hot path.

## The Bit Is the Answer

This is the philosophical point. Most systems compute a violation check and then *store the result* in a data structure. FLUX computes the violation check and the bit *is* the result. There's no intermediate representation.

```c
mask |= (isnan(val) || val < lo || val > hi) << i;
```

One line. One bit. No allocation. This compiles to 3-4 instructions on any CPU: compare, compare, OR, shift-OR. On GPU, it compiles to predicated instructions with zero warp divergence.

**This is why FLUX hits 24.9 billion checks per second.** The data structure was chosen to match the hardware, not the programmer's intuition.

## What About More Than 64 Constraints?

Use `uint64` chunks. 256 constraints = 4 `uint64`s. All the same operations work with SIMD (OR four lanes at once). The principle doesn't change — it just gets wider.

**Next:** The bug that proved why the NaN check matters → [NaN Trap](nan-trap.md)
