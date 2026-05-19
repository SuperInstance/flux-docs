# Constraints as Thermodynamics

A constraint system *is* an ideal gas. The math works. Here's why.

## The Partition Function

In statistical mechanics, the partition function describes a system's thermodynamic behavior:

```
Z = Σ exp(-Eᵢ / kT)
```

For a Boolean constraint system with N independent constraints, each constraint has two states: pass (E=0) or violate (E=w, where w is the violation weight). The partition function factorizes:

```
Z = Πᵢ (1 + e^(-wᵢ/kT))
```

This factorization is **exact** for independent constraints. No approximation. No perturbation theory. The partition function factorizes because the constraint events are independent — exactly what [fracture-coalesce](fracture-coalesce.md) exploits.

## Temperature = How Strict

The "temperature" T controls how strictly violations are penalized:

| T → 0 (strict) | T → ∞ (lenient) |
|:---------------|:----------------|
| All violations are fatal | Violations are tolerated |
| `e^(-w/kT) → 0` | `e^(-w/kT) → 1` |
| Z → N (only passing states count) | Z → 2N (all states count equally) |

At T=0, the system is a hard constraint engine. Violation = death. At T=∞, the system is permissive. Violations are noted but don't affect behavior.

**This is a knob you can turn.** Start with T=∞ during testing (see all violations). Decrease T in production (enforce strictly). The math handles the interpolation.

## Energy = Total Violations

```
E = -∂(ln Z)/∂β    where β = 1/kT
```

For independent constraints, this simplifies to:

```
E = Σᵢ wᵢ · σ(vᵢ)
```

where `σ(vᵢ) = e^(-wᵢ/kT) / (1 + e^(-wᵢ/kT))` is the Fermi-Dirac occupancy — the "probability" that constraint `i` is violated at temperature T.

The total energy is the weighted sum of violations. At T=0, this is just `Σ violated weights`. At T>0, each violation contributes a fraction between 0 and its full weight.

## Entropy = Information Content

```
S = k · ln Z + E/T
```

High entropy = many constraint states are accessible = the system is "loose." Low entropy = few states = the system is "tight."

This is useful for monitoring: if your constraint system's entropy suddenly spikes, something changed. New violation patterns are emerging. The entropy is an alarm bell.

## Why This Works

Boolean constraint systems are *cleaner* than most physical systems:

1. **No interactions** — Independent constraints have no coupling terms. The Hamiltonian is diagonal. Real physics has interactions (van der Waals, spin-orbit coupling, etc.). We don't.
2. **Discrete states** — Each constraint is binary (pass/fail). No continuous energy spectrum. The partition function sums over exactly 2N states.
3. **Factorization is exact** — In physics, factorization is an approximation (mean-field theory). For independent constraints, it's a theorem.
4. **No renormalization needed** — Physical systems have divergences at short distances. Boolean constraints don't. The theory is well-defined at all scales.

This is why the thermodynamic analogy is more than a metaphor — it's a mathematical isomorphism for independent constraint systems.

## What This Enables

### Soft Constraints
Instead of hard pass/fail, use temperature to soften constraints:
```
strictness = 1 - e^(-violation_weight / temperature)
```
High violation at low temperature → strictness ≈ 1 → hard fail. Low violation at high temperature → strictness ≈ 0 → pass.

### Anomaly Detection
Monitor the entropy of your constraint system over time. Sudden entropy changes = something changed in the underlying system. No need to know *what* changed — the entropy tells you *something* changed.

### Resource Allocation
The free energy `F = E - TS` tells you the "cost" of running at a given strictness level. At low T, F ≈ E (pure violation cost). At high T, F is dominated by -TS (the cost of permissiveness). Minimize F to find the optimal operating strictness.

## The Precision Class Connection

FLUX's precision classes (INT8, FP16, FP32, FP64) are *renormalization-like* scale transitions:

- INT8: 76% of FP64 violations are missed. This IS a phase transition — the system changes character at this boundary.
- FP16: ~99% accuracy. Most of the physics is captured.
- FP32: ~99.99% accuracy. For most applications, indistinguishable from FP64.
- FP64: Ground truth.

This is the RG flow of constraint checking: coarse-grain (lower precision) and the effective theory changes. The critical point (where accuracy drops sharply) is between INT8 and FP16.

**Next:** See all 96 language implementations → [Languages](../languages/index.md)
