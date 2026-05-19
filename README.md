# FLUX Constraint Engine — Documentation

## Navigate

| Page | What |
|------|------|
| [Index](index.md) | What is FLUX? Start here. |
| [Getting Started](getting-started.md) | 5-minute tutorial in Python, Rust, JS, and C. |

### Concepts

| Page | What |
|------|------|
| [Error Masks](concepts/error-mask.md) | 1 bit per constraint. The fundamental data structure. |
| [NaN Trap](concepts/nan-trap.md) | IEEE 754's dirty secret and why NaN always violates. |
| [Fracture-Coalesce](concepts/fracture-coalesce.md) | Independent constraints → parallel blocks → bitwise OR merge. |
| [Sediment](concepts/sediment.md) | Immutable correction layers. Correctness only grows. |
| [Thermodynamics](concepts/thermodynamics.md) | Constraints as ideal gases. The partition function factorizes. |

### Languages

| Page | What |
|------|------|
| [96 Languages](languages/index.md) | What each language taught us. |
| [Old Architecture](languages/old-architecture.md) | Why COBOL reveals the optimal constraint engine shape. |

### GPU

| Page | What |
|------|------|
| [GPU Benchmarks](gpu/index.md) | 24.9B checks/sec. Why error masks are the ideal GPU workload. |

### API Reference

| Page | What |
|------|------|
| [Python](api/python.md) | `flux-lib` — `pip install flux-lib` |
| [Rust](api/rust.md) | `flux-fracture` — `cargo add flux-fracture` |
| [JavaScript](api/javascript.md) | `@flux/check` — `npm install @flux/check` |
| [C](api/c.md) | `flux_fracture.h` — single-header, no dependencies |

### Research

| Page | What |
|------|------|
| [31 Modules](research/index.md) | What we asked, what died, what survived, what's new. |
| [Grand Synthesis](research/grand-synthesis.md) | Full analysis: 7 agents, 4 models, 2 rounds. |

## Build

```bash
# Serve raw markdown
cd flux-docs
python3 -m http.server 8080

# Or convert to HTML
make html
```

## Reading Order

1. [Index](index.md) → [Getting Started](getting-started.md)
2. [Error Masks](concepts/error-mask.md) → [NaN Trap](concepts/nan-trap.md) → [Fracture-Coalesce](concepts/fracture-coalesce.md)
3. [Sediment](concepts/sediment.md) → [Thermodynamics](concepts/thermodynamics.md)
4. [96 Languages](languages/index.md) → [Old Architecture](languages/old-architecture.md)
5. [GPU Benchmarks](gpu/index.md)
6. API for your language
7. [Research](research/index.md) → [Grand Synthesis](research/grand-synthesis.md)

## License

MIT — Part of the [SuperInstance](https://github.com/SuperInstance) constraint-theory ecosystem.
