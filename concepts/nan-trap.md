# The NaN Trap

The bug that started FLUX.

## IEEE 754's Dirty Secret

In IEEE 754 floating-point arithmetic, **every comparison with NaN returns false**:

```python
NaN < 5    # False
NaN > 5    # False
NaN == NaN # False
NaN != NaN # True (the ONLY true comparison)
NaN <= 5   # False
NaN >= 5   # False
```

This means a standard bounds check silently passes NaN values:

```python
def check_broken(value, lo, hi):
    if value < lo or value > hi:
        return False  # violated
    return True  # passed
```

What happens when `value = NaN`? `NaN < lo` is `False`. `NaN > hi` is `False`. So the `or` is `False`. The function returns `True` — **passed**. A NaN just snuck through your constraint check.

## Why This Matters

NaN isn't a theoretical edge case. It shows up in real systems:

- **Sensor failure** — A temperature probe disconnects. The ADC reads `0xFFFFFFFF`. Your parsing code produces NaN.
- **Division by zero** — A flow rate calculation divides by elapsed time. First iteration: elapsed time is 0. NaN propagates.
- **Overflow** — An intermediate calculation exceeds `DBL_MAX`. IEEE 754 produces `Inf`, which then contaminates downstream values as NaN.
- **Uninitialized memory** — In C, an uninitialized `double` on the stack is often `NaN` (or `0x7FF8000000000000`).

In any of these cases, a broken bounds check says "passed." The system proceeds with garbage data. The failure propagates silently through every downstream calculation until something *else* breaks — far from the root cause.

## The Fix

One explicit check, before the comparison:

```c
uint8_t flux_check(double val, double lo, double hi) {
    return (isnan(val) || val < lo || val > hi) ? 1 : 0;
}
```

The `isnan()` check is not optional. It's not a nice-to-have. It's the difference between a constraint engine that catches failures and one that silently endorses them.

**In FLUX, NaN always violates.** Every implementation — Python, Rust, JavaScript, C, Fortran, COBOL — checks NaN first. There is no flag to disable it. There is no "strict mode" you have to opt into. It's the default and only behavior.

## The Pattern

This is a general pattern in numerical systems:

> **If your comparison doesn't handle NaN, it's not a comparison — it's a lie.**

The same pattern applies to:

- `NULL` in SQL (three-valued logic)
- `undefined` in JavaScript (loose equality is broken)
- Sentinel values in embedded systems (`0xFFFF` thermocouple readings)
- Missing data in data science (`np.nan` in pandas)

FLUX's approach is the same for all of them: **check for the invalid value explicitly, before the domain comparison.** Don't rely on the comparison operator to reject it. It won't.

## Why NaN Detection is Bit 0

In FLUX's internal implementation, the NaN check happens before the bounds check. Conceptually:

```
if isnan(value):     → violate (no bounds check needed)
elif value < lo:     → violate
elif value > hi:     → violate
else:                → pass
```

Short-circuit evaluation means NaN values skip the bounds comparison entirely. On GPU, this compiles to a predicated instruction sequence with zero divergence — the warp doesn't branch, it just sets the bit.

**This is why "zero false negatives" is a theorem, not a claim.** The NaN check is part of the theorem's proof. Without it, the proof doesn't hold.

**Next:** How we split independent constraints for parallel checking → [Fracture-Coalesce](fracture-coalesce.md)
