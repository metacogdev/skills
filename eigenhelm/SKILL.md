---
name: eigenhelm
description: Evaluate code quality using eigenhelm aesthetic scoring. Run after writing or modifying code files to get actionable improvement directives with automatic iteration limits.
---

# Eigenhelm Code Quality Evaluation

## Running Evaluation

After writing or modifying a code file, evaluate it:

```bash
eh evaluate --classify --format human <file>
```

If `eh` is not on PATH, use `eigenhelm evaluate` or `uv run eh evaluate`.

## Interpreting Output

**Score**: 0.0 (perfect) to 1.0 (worst). Lower is better.

**Decisions** (thresholds vary by config/model):
- **accept** (score ≤ 0.4): Good. Move on.
- **marginal** (0.4 < score < 0.6): Review high-severity directives.
- **reject** (score ≥ 0.6): Must address high-severity directives.

**Percentile**: "p50" = better than 50% of training corpus.

**Five scoring dimensions**:
1. `manifold_drift` — distance from learned code manifold
2. `manifold_alignment` — alignment with principal quality axes
3. `token_entropy` — information density of token stream
4. `compression_structure` — compressibility / structural regularity
5. `ncd_exemplar_distance` — normalized compression distance to nearest exemplar

**Directives** have severity `[high]`, `[medium]`, or `[low]`. Only act on `[high]` items.

## Iteration Protocol

Follow this strictly to avoid infinite refinement loops.

1. Write or modify a file, then run `eh evaluate --classify --format human <file>`.
2. **reject**: Address `[high]` severity directives only. Re-evaluate. Max **3 attempts**.
3. **marginal**: Address `[high]` directives if straightforward. Max **2 attempts**.
4. **accept**: Done. Move on.

**STOP RULE**: If the score does not improve by ≥ 0.03 between attempts, stop immediately. The remaining issues are structural and cannot be fixed by refactoring.

**Small files** (< 80 lines): `improve_compression` and `review_structure` directives are expected and capped at `[medium]` severity. Focus only on PCA-derived directives (`reduce_complexity`, `extract_repeated_logic`).

## Common Flags

| Flag | Purpose |
|---|---|
| `--model <path>` | Specific trained model (.npz) |
| `--format json` | Machine-readable JSON output |
| `--format sarif` | SARIF 2.1.0 for CI integration |
| `--diff <range>` | Only files changed in git revision range |
| `--scorecard` | Per-repo scorecard (M1-M5, Q1-Q5) |
| `--strict` | Treat marginal as reject |
| `--lenient` | Treat marginal as accept |

## Setup

Project config lives in `.eigenhelm.toml`. Run `eh init` to generate defaults. Key settings:
- `model`: path to trained .npz model
- `thresholds.accept` / `thresholds.reject`: classification boundaries
- Threshold hierarchy: CLI flags > config file > model calibration > hardcoded defaults
