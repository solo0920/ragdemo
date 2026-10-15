# AGENTS.md

## Repo state
- Spec-first repo; **no application code exists yet**: no `pyproject.toml`, `uv.lock`,
  tests, or Docker files. Do not run `uv`/`pytest`/compose until they actually exist.
- `README.md` (Traditional Chinese) is the only product description: 司法院 open data
  → rar 解壓縮 → duckdb 清洗金融相關實例 → parquet.
- Features live in `specs/NNN-<name>/` (not created yet). The active feature pointer is
  `.specify/feature.json` — gitignored, machine-local, rewritten by the scripts.

## Constitution is binding — read it before planning
- `.specify/memory/constitution.md` supersedes other practices. It is currently
  `v2.0.0-DRAFT.4`, not ratified; the ratification date is a TODO — do not fill it in.
- Non-Negotiable principles (cannot be waived by Complexity Tracking): CM1-CM4, E1-E3,
  G1-G3. Violations mean stop and report; only a formal G3 amendment can change them.
- G2 checkpoints are workflow-event based: before plan research/design, after design
  artifacts (before tasks), and after implementation. Record deviations in the
  Complexity Tracking table.
- Stay inside approved spec/task scope (G1, G5); stop and ask before changing behavior
  or scope that the spec did not authorize.

## Evidence rules (apply to artifacts: reports, PRs, commits, docs)
- Label factual claims `[verified]` or `[inferred]`; measurements need the exact
  command, archived output location, and denominator (E1).
- Confirm files, functions, and field names exist before citing them (E5); never
  fabricate evidence or mark work complete without verification output (E3).
- Implementation notes for the future byte-level contracts (CM3): offsets are
  zero-based bytes into canonical UTF-8, half-open `[start, end)`, never character
  indices; chunk/hash tests must operate on `bytes` objects.

## Workflow tooling
- OpenCode commands: `/speckit.specify`, `/speckit.clarify`, `/speckit.plan`,
  `/speckit.tasks`, `/speckit.analyze`, `/speckit.implement`, `/speckit.checklist`,
  `/speckit.constitution`. Gate order: specify → plan → tasks → implement.
- Underlying scripts, run from repo root with `--json`:
  `.specify/scripts/bash/create-new-feature.sh "<desc>"`,
  `setup-plan.sh`, `setup-tasks.sh`,
  `check-prerequisites.sh --require-spec --require-tasks --include-tasks`,
  `resolve-template.sh <template-name>`.
- `/speckit.constitution` writes **only** `.specify/memory/constitution.md`. Do not
  hand-edit `.opencode/commands/`, `.specify/templates/`, or `.specify/scripts/` to
  change behavior; those are managed by Spec Kit (speckit_version 1.1.3).

## Fixed stack
- Python + uv + FastAPI; PostgreSQL + Qdrant; Docker Compose. Stack changes are
  architectural and require constitutional justification (A1, A2).

## Known gaps
- `docs/ENVIRONMENTS.md` is referenced by the constitution (D3) but does not exist
  yet; create it when supported environments are defined.
