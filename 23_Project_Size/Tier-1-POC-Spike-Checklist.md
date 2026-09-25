---
document_type: Project Tier Checklist — Tier 1 POC / Spike
schema_version: 2
canonical_name: Tier 1 POC / Spike Checklist
doc_form: record
applicability: evidence
min_project_tier: 1
minimum_form: This checklist itself
version: "1.0"
status: Active
created: "2026-09-24"
last_updated: "2026-09-25"
tags: [tier-checklist, tier-1, project-size, generated]
generator: Generated from template frontmatter — do not hand-edit; regenerate instead
---

# 🧪 Tier 1 — POC / Spike — Document Checklist

> **Description:** Validate an idea. Throwaway code. `git push` is fine.
> **Team:** 1 dev | **Users:** Internal only | **Lifespan:** Days–weeks
> **Tier source:** `checklist/release-checklist/release.md` §Project Tier Scoping Matrix
>
> **Key principle:** Everything a Tier-1 project needs = every template with `min_project_tier ≤ 1` whose trigger applies. Anything above tier 1 is over-engineering for this context.
>
> **Scope:** 2 artifacts (2 new at this tier, 0 inherited from lower tiers).
> **Every project starts with** [[Product-Brief]] — the human's one-page idea seed.

## How to use

1. Confirm your tier with the decision flow in `release.md` §"Which Tier Am I?".
2. Tick ☐ → ✅ as each document is created. **Skip rows whose trigger condition doesn't apply** and note the omission in [[Tailoring-Justification]].
3. `Form` column: **heavy** = full controlled document · **light** = lean doc · **record** = produced by an activity (create when the activity happens, not upfront).
4. Priority: 🔴 universal · 🟡 conditional/evidence (check trigger) · 🟢 technique (embed in parent docs, never standalone).

## Tier ladder

| Tier | Checklist | Artifacts |
|---|---|---|
| 1 | **→ this tier** | 2 |
| 2 | [[Tier-2-Prototype-MVP-Checklist|🔧 Prototype / MVP]] | 7 |
| 3 | [[Tier-3-Internal-Tool-Checklist|🏠 Internal Tool]] | 22 |
| 4 | [[Tier-4-Small-Production-Checklist|🟢 Small Production]] | 229 |
| 5 | [[Tier-5-Medium-Production-Checklist|🔵 Medium Production]] | 322 |
| 6 | [[Tier-6-Production-Grade-Checklist|🟣 Production Grade]] | 343 |
| 7 | [[Tier-7-Mission-Critical-Checklist|🔴 Mission-Critical / Regulated]] | 361 |

## Business Analysis and strategy

> **Owner:** PO / BA · 1 artifacts

| Document | Priority | Form | Applies at tier | Trigger / Note | Status |
|---|---|---|---|---|---|
| [[Product-Brief]] | 🔴 | light | T1 🆕 | always | ☐ |

## SE Cross Cutting

> **Owner:** Systems Engineer · 1 artifacts

| Document | Priority | Form | Applies at tier | Trigger / Note | Status |
|---|---|---|---|---|---|
| [[Tailoring-Justification]] | 🔴 | light | T1 🆕 | always | ☐ |

---

## Tailoring record

| # | Document omitted / combined | Reason | Approved by |
|---|---|---|---|
| 1 | [Document] | [Why not needed at this tier] | [Name] |

> Copy omissions into the project's [[Tailoring-Justification]].

## Related

- [[7-Tier Applicability Matrix]] — full matrix view over all 361 templates
- [[release]] — tier model source (Project Tier Scoping Matrix)
- [[TEMPLATE-INDEX]] — regenerated master index
- [[Product-Brief]] — where every project starts
