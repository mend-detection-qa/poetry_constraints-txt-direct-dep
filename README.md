# poetry_constraints-txt-direct-dep

**What this probe proves:** Mend's `constraints.txt` support applies to **direct dependencies** on a Poetry project, not only transitives. When `pyproject.toml` declares a direct dep with a wide range, the `constraints.txt` entry narrows the direct dep version emitted by Mend — even though Poetry itself does not read constraints.txt natively.

## Why this probe exists

All other Poetry constraints.txt probes pin a transitive (`urllib3`). The direct-dep axis was uncovered — this probe closes it.

## Files

- `pyproject.toml` — declares `requests = ">=2.0"` (wide range) so the resolver has room to pick any 2.x; poetry.lock will land at the latest 2.32.x
- `poetry.lock` — **NOT YET GENERATED.** Run `poetry lock` on a real machine. Will reflect Poetry's unconstrained resolution (requests at latest 2.32.x). Do NOT hand-craft.
- `constraints.txt` — `requests==2.30.0` (older version, so we can prove the constraint did something — latest would be ≥2.32.5)
- `whitesource.config` — `python.applyConstraints=true`

## Expected scan behavior (post SCA-5154)

Mend's dep tree reports:
- Direct: `requests` at **`2.30.0`** (constrained, even though `poetry.lock` shows the latest 2.32.x)
- Transitives at the resolver defaults for requests 2.30.0

The divergence between `poetry.lock` (requests=latest, Poetry ignores constraints.txt natively) and Mend's scan output (requests=2.30.0) is the load-bearing signal — Mend applied the constraint independently of the lockfile.

## Load-bearing assertion

`expected_dependency_pairs` pins `requests@2.30.0`. The direct dep version is the load-bearing signal.
