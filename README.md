# Document Template

A personal software development document library for persona-driven, spec-driven development.

## What this is

- 361 markdown templates, one per document type
- Every template has YAML frontmatter (`schema_version: 2`) with tier, applicability, and form metadata
- Placeholders only — no fake example data

## Folders

| Folder | Content |
|---|---|
| `01`–`22` | Templates by category (business analysis, requirements, design, testing, security, data, ops, …) |
| `23_Project_Size` | 7 tier checklists — pick your project tier, use that checklist |
| `_governance` | Index, matrix, audit and quality reports (generated — do not hand-edit) |
| `99_Archive` | Superseded files, kept for history |

## How to use

1. Start a project: copy `01_Business_Analysis_and_strategy/Product-Brief.md`, fill it roughly (one page).
2. Pick your tier (1–7) with the decision flow in `checklist/release-checklist/release.md` ("Which Tier Am I?").
   - 1 POC · 2 Prototype · 3 Internal · 4 Small Prod · 5 Medium Prod · 6 Production Grade · 7 Mission-Critical
3. Open `23_Project_Size/Tier-N-...-Checklist.md` and create only the documents listed there. Skip rows whose trigger doesn't apply; note skips in Tailoring-Justification.
4. Copy each template into your project spec folder and fill it in.

## Rules

- Frontmatter is the source of truth. `_governance/TEMPLATE-INDEX.md`, `7-Tier Applicability Matrix.md`, and the tier checklists are generated from it — regenerate, never hand-edit.
- Keep templates placeholder-only. Examples are generated on demand, not stored here.
- Mermaid only: `flowchart` for architecture, `treeView-beta` for trees, never `mindmap`.
- When two templates cover the same concept, `source_of_truth` in frontmatter says which one is canonical; the others are views.

## Governance docs

| File | What |
|---|---|
| `_governance/TEMPLATE-INDEX.md` | Master index, counts computed from disk |
| `_governance/7-Tier Applicability Matrix.md` | Every template × tier × applicability |
| `_governance/Template Restructure Plan - 2026-09-23.md` | How this library was rebuilt (decisions D1–D4) |
| `_governance/Phase 5 Quality Gate Report - 2026-09-24.md` | Verification results |
| `_governance/Transmatter Retrospective - 2026-09-25.md` | Comparison against a real senior project |

## Standards

Templates cite ISO/IEC/IEEE standards where applicable (29148, 42010, 29119, 828, 730, 1012, 25010, 27001:2022, …). Project-level process standard is selected per tier via the `standards_profile` frontmatter field: tiers 1–4 default to ISO/IEC 29110 (VSE), 5+ consider 12207, 7 adds domain regulations.
