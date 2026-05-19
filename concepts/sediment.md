# Sediment Layers

Accumulated correctness. Every edge-case fix is an immutable layer. Nothing is deleted.

## The Problem

You deploy a constraint engine with bounds [0, 100] on some sensor. It runs for months. Then someone finds an edge case: sensor reads 0.001 when it's disconnected. The value is technically within bounds, but it's wrong.

Options:
1. Change the bound to [0.5, 100]. But now you've lost the original intent (0 *is* a valid reading sometimes).
2. Add a special-case `if` in the checking code. But now you're patching production code for a specific sensor.
3. **Add a sediment layer.**

## How Sediment Works

A sediment layer is an immutable correction applied *after* the base check:

```
Base check:   value in [0, 100]    → mask bit 0
Sediment L1:  if value < 0.5 AND sensor_type == "pressure": set bit 0 (violation)
Sediment L2:  if value == 0.0 AND last_reading > 50: set bit 0 (violation — sensor drop)
Sediment L3:  if value > 99.9 AND sensor_type == "flow": clear bit 0 (false positive at max)
```

Each layer is a narrow correction. It doesn't rewrite the base check. It sits on top, handling one specific edge case.

## Key Properties

### Monotonic Correctness

Each layer can only increase the set of correct answers. It can:
- **Set a bit** (mark a passing value as violated — catch a missed failure)
- **Clear a bit** (mark a violated value as passed — correct a false positive)

But the *knowledge* only grows. Layer 3 doesn't undo Layer 1 — it adds a new correction on top. The complete picture is the base mask + all layers applied in order.

### Immutability

Once written, a sediment layer never changes. If Layer 2 was wrong, you don't edit Layer 2 — you add Layer 4 that corrects it. This is the COBOL pattern: code is frozen, corrections are additive.

### Bounded Size

Sediment layers are fixed-size arrays (like COBOL `OCCURS 50 TIMES`). When the stack is full, the oldest layer is superseded — not deleted, but marked as inactive. The newest correction wins.

## The Frozen Core + Open Edges Pattern

```
┌──────────────────────────────────┐
│  FROZEN CORE (never changes)     │
│                                  │
│  Constraint definitions          │
│  Base bounds                     │
│  Dependency graph                │
│                                  │
│  This is the hot path.           │
│  It runs 25 billion times/sec.   │
│  It is provably correct.         │
└──────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────┐
│  OPEN EDGES (only grows)         │
│                                  │
│  Sediment Layer 1: NaN edge      │
│  Sediment Layer 2: sensor drop   │
│  Sediment Layer 3: overflow fix  │
│  Sediment Layer 4: ...           │
│                                  │
│  This runs AFTER the hot path.   │
│  Only on previously-violated     │
│  constraints. (~5% overhead)     │
└──────────────────────────────────┘
```

The hot path never changes. It's the frozen core — optimized, verified, locked down. The sediment layers are the open edges — the mutable, growing body of corrections.

This is why COBOL systems still run after 40 years. The core is frozen. The corrections accumulate. The system is never "redeployed" — it *grows*.

## On GPU

Sediment layers on GPU are surprisingly efficient. The key insight: only re-check constraints that *already violated* in the base check. If the base mask says constraint 3 passed, no sediment layer can affect it (layers only operate on violated constraints to potentially clear them, or on passed constraints that should have failed).

In practice, sediment adds ~5% overhead on top of the base check. The GPU sediment kernel processes 2.95 billion values/sec — still astronomical.

## Why "Sediment"?

Geological metaphor:

- **Bedrock** — the constraint definitions. Fixed. Hard. Millions of years old.
- **Sediment** — the layers of corrections. Each one is thin. Each one represents an event (a bug found, an edge case discovered). They accumulate over time.
- **Fossil record** — you can read the history of the system by reading the sediment layers. Layer 1 was the first bug. Layer 2 was the second. Each one tells a story.

The constraint engine's sediment stack IS the fossil record of its correctness. Never delete. Only supersede. COBOL computes. MUMPS remembers.

**Next:** What physics has to do with constraint systems → [Thermodynamics](thermodynamics.md)
