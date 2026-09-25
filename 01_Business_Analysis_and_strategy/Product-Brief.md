---
document_type: Product Brief
schema_version: 2
canonical_name: Product Brief
doc_form: light
applicability: universal
min_project_tier: 1  # POC
minimum_form: "This one page — fill what you know, leave [?] where you don't"
standards_profile: "[ISO/IEC 29110 VSE | ISO/IEC/IEEE 12207 | none]"  # set per project tier: T1-T4 default 29110, T5+ consider 12207, T7 adds domain regs
version: "1.0"
status: Active
author: "[Human — the idea owner]"
created: "[YYYY-MM-DD]"
last_updated: "[YYYY-MM-DD]"
project_name: "[Project Name]"
classification: "Internal"
tags: [product-brief, intake, idea-seed, spec-driven, entry-point]
---

# Product Brief — [Project Name]

> **What this is:** the human's one-page seed for a new project. Write it rough — half-answers and `[?]` marks are fine. You fill this; the PO grills you on it; together you grow it into the real spec chain (Business Objectives → Stakeholder Needs → User Stories → Acceptance Criteria → …).
>
> **Rule:** this document stays *short*. If a section grows beyond a few lines, it belongs in a downstream template — the PO will move it during the grill session.

---

## 1. The Idea (one line)

> What do you want to build? One sentence, no jargon.

[I want to create …]

## 2. Why — The Problem / Motivation

> Why does this need to exist? Whose pain is it (yours? users'? a client's?)? What happens today without it?

- Problem: [describe the pain or gap]
- Today's workaround: [how it's handled now, if at all]
- Why now: [trigger — deadline, opportunity, annoyance threshold]

## 3. Who — Users

> Who actually uses it? Be specific; "everyone" is a red flag the PO will grill.

| User | What they need from it |
|---|---|
| [Primary user] | [their goal] |
| [Secondary user] | [their goal] |

## 4. Core Features (rough list)

> What must it do? Bullet points, no priority yet — the PO helps you cut and order these.

- [Feature 1]
- [Feature 2]
- [Feature 3]

**Explicitly NOT wanted (if known):** [out-of-scope ideas]

## 5. Simple Data

> What information does it hold? A rough list is enough — field names, not types.

```
[entity-1]: [field], [field], [field]
[entity-2]: [field], [field]
```

## 6. Simple Response / Shape

> If it's an API or has screens: what does "it worked" look like? A rough JSON sketch or a described screen is fine.

```json
{
  "[field]": "[value]",
  "[field]": "[value]"
}
```

or: [describe what the user sees on success]

## 7. Picked Tech (and why)

> What stack are you leaning to? Honesty about constraints matters more than openness — the PO only challenges choices that fight your existing infra or the problem.

| Concern | Pick | Why |
|---|---|---|
| Language / framework | [e.g., Go + Fiber v3] | [reason] |
| Storage | [e.g., PostgreSQL / MongoDB / file] | [reason] |
| Frontend | [e.g., Next.js / none / CLI] | [reason] |
| Deployment | [e.g., homelab Docker + Cloudflare Tunnel] | [reason] |
| External services | [APIs, OAuth providers, …] | [reason] |

## 8. Constraints & Context

> Budget: [free / $X] · Time: [days / weeks / no deadline] · Must reuse: [existing infra, accounts, domains] · Privacy notes: [personal data involved?]

## 9. What Does Success Look Like?

> One or two observable outcomes. Not metrics theater — what changes in your life/business when this works?

- [Outcome 1]
- [Outcome 2]

## 10. Open Questions / Unknowns

> Everything you're unsure about. The PO's grill session starts here.

- [?] [question 1]
- [?] [question 2]

## 11. My Tier Guess (optional)

> 🧪 POC · 🔧 Prototype · 🏠 Internal · 🟢 Small-Prod · 🔵 Medium-Prod · 🟣 Prod-Grade · 🔴 Mission-Critical
> The PO confirms this with the decision flow in `release.md` §"Which Tier Am I?" — it determines how much documentation the project needs.

Guess: [tier]

---

## What Happens Next (PO workflow — do not fill)

| Step | Output | Template |
|---|---|---|
| 1. Grill session on this brief | Confirmed problem, users, scope, tier | — |
| 2. Business direction | Objectives + success measures | [[Business-Objectives]] |
| 3. Needs & stakeholders | Who needs what, evidence | [[Stakeholder-Needs-Document]] / [[Stakeholder-Analysis]] |
| 4. Requirements | Stories + acceptance criteria | [[User-Stories]], [[Acceptance-Criteria]] |
| 5. Tailored doc set | Tier checklist cut to real scope | [[Tailoring-Justification]] + `23_Project_Size/Tier-N` |
| 6. Design/QA/Dev handoffs | Per persona, via meeting minutes | Meeting-minute pattern (PO skill: `templates/designer-handoff-meeting-minute.md`) |

> This brief is **kept, not discarded** — it is the provenance record of the original human intent. Later documents link back to it.

## Related Documents

| Document | Relationship |
|---|---|
| [[Business-Objectives]] | First formal output grown from this brief |
| [[Business-Case]] | Heavy alternative when investment approval is needed |
| [[Tailoring-Justification]] | Records which tier checklist items are skipped and why |

---

> **Template Standard:** intake seed for spec-driven development; aligns with BABOK elicitation (confirmed vs unconfirmed results) and ISO/IEC 29110 VSE agreement process
> **Usage:** fill sections 1–10 roughly, hand to the PO, say "grill me". Keep it under two pages.
