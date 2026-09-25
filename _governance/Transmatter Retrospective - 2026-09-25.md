---
tags: [retrospective, provenance, senior-project, transmatter, iso-29110, harvest]
status: Complete
created: 2026-09-25
author: PO (product-owner persona)
source_document: "F:\\books\\Senior_Project\\Transmatter_Platform.pdf (521 pages, Nov 2022)"
related_plan: "[[Template Restructure Plan - 2026-09-23]]"
---

# Transmatter Retrospective — Senior Project vs Template Library

> **Purpose:** Compare the Founder's real senior-project document set (Chiang Mai University, BSc Software Engineering, Nov 2022 — a full lifecycle from Proposal to end-of-service, ISO 29110 VSE-based) against the restructured template library, and record what was harvested vs what was deliberately not adopted.
> **Verdict:** The library is **not outdated relative to the senior project — it is a superset**. The senior project validated the library's core design and contributed two improvements (applied 2026-09-25).

---

## 1. What the Transmatter Platform was

A content-reader platform for the visually impaired: admin subsystem (content management/verification, fetching from Sanook/ThaiRath/Dek-D + NewsAPI), accessible web UI (aria labels, audio feedback, shortcut keys, spell correction, WCAG principles), and a physical touchpad UI with speech-to-text. 2-person team, ~8 months, advisor-gated releases.

## 2. Document set produced (chapter map from the PDF)

| Ch | Document | Pages (PDF) | Library equivalent | Coverage verdict |
|---|---|---|---|---|
| 1 | Project Proposal (motivation, aims/objectives, deliverables & limits, business/literature/technology review, quality standard, milestones) | 1–30 | [[Business-Case]] + [[Business-Objectives]] + [[Solution-Scope]] + [[Market-Analysis-Technology-Assessment]] | ✅ Covered (split across finer templates) |
| 2 | Project Management Plan (identification, overview, work products, infrastructure/dev tools, team structure, monitoring & controlling, meeting mechanism, ISO 29110 VSE processes, quality planning/reviews, naming convention, change management, project repository, risk identification, schedule) | 31–42 | [[Project-Management-Plan]] + [[Risk-Register]] + [[RACI-Matrix]] + [[Coding-Standards]] + [[SCMP]] + [[Project-Schedule]] | ✅ Covered |
| 3 | Change Request Document (dated change log: Update/Draft/Remove per chapter, Viewable/Editable/Responsible per entry) | 43 | [[Change-Request]] + [[Requirements-Change-Log]] | ✅ Covered |
| 4 | Software Requirement Specification (purpose per ISO 29110, objective, feature descriptions, use-case diagrams, URS, SyRS) | 44–104 | [[Software-Requirements-Specification]] + [[Use-Case-Specifications]] + [[System-Requirements-Specification]] | ✅ Covered |
| 5 | Software Design Development (system architecture, class diagrams per subsystem, function descriptions, ERD/data modeling, sequence diagrams, UI design) | 105–196 | [[Software-Architecture-Document]] + [[Class-Diagrams]] + [[ERD]] + [[Sequence-Diagrams]] + 11_UX_UI_Design family | ✅ Covered |
| 6 | Test Plan (purpose, objectives, scope, responsibility, per-class unit tests, system tests) | 197–282 | [[Test-Plan]] + [[Test-Cases]] + [[Test-Strategy]] | ✅ Covered |
| 7 | Test Record (unit test case records UTC-xx with inputs/expected/actual, system test records with tester sign-off) | 283–461 | [[Test-Report]] + [[Defect-Report]] + (execution logs) | ✅ Covered |
| 8 | Traceability Record (requirement ↔ test linkage) | 462–467 | [[Requirements-Traceability-Matrix]] + [[Traceability-Matrix-Req-Tests]] | ✅ Covered |
| 9 | Executive Summary (project perspective, progress summaries, acronyms) | 468–end | [[Final-Report]] + [[Verified-Deliverables]] | ✅ Covered |

**Conclusion: every chapter of a real, advisor-approved, ISO-29110-aligned lifecycle has a template equivalent (usually several, at finer granularity). The library omits nothing the senior project needed — and adds operations, security, data-management, retirement, and tier tailoring the 2022 project did not cover.**

## 3. Genuinely good patterns harvested ✅ (applied 2026-09-25)

### A. Document History & Access matrix (from Ch1–3 "Document History" tables)
Every Transmatter chapter doc carried a per-revision table with **Status (draft/review/release), Date, Viewable, Editable, Responsible** — a real document-access/change-control matrix our Revision History tables lacked (they track *what changed*, not *who may see/edit/approve*).

→ **Applied:** all 73 heavy templates (those with a `## Document Control` section) now include a *Document History & Access* matrix with the draft→review→release flow and an editable/responsible enforcement note linking to [[Change-Request]].

### B. `standards_profile` frontmatter hook (from Ch1 §3 "ISO 29110 for Very Small Entity")
Transmatter explicitly declared its process standard (ISO/IEC 29110 VSE) and mapped PM + implementation processes to it. Our templates cited ISO/IEEE standards per-document but had **no project-level process-profile selector** — exactly the "tailoring is declared, not assumed" principle from ED-A02/ED-A05.

→ **Applied:** all 360 templates now carry `standards_profile: "[ISO/IEC 29110 VSE | ISO/IEC/IEEE 12207 | none]"` with tier guidance (T1–T4 default 29110 VSE — it was literally written for very small entities like a 2-person team; T5+ consider 12207; T7 adds domain regulations like HIPAA/PCI-DSS).

## 4. Deliberately NOT adopted ❌

| Senior-project trait | Why not |
|---|---|
| Monolithic 521-page single PDF with 9 chapters | Contradicts spec-driven modularity: AI personas and tiered tailoring need per-artifact files; the PDF itself shows the pain (Chapter 4 SRS rewritten across Progress 1/2 with whole-chapter change entries) |
| Duplicated boilerplate per chapter (each chapter re-states project objective/scope verbatim) | The library's umbrella+service pattern and `source_of_truth` overlap fields exist precisely to prevent this drift |
| Hand-maintained page counts / printed "Document name/type/owner/release date/page/print date" footers | Replaced by frontmatter (`status`, `version`, `last_updated`) — machine-readable, and counts are generated (RC-03 lesson) |
| Test records as 180 pages of per-case screenshots/tables inside the doc | Records belong in trackers/reports; library classifies these as `doc_form: record` with `minimum_form` guidance |
| WCAG **3.0** citations (draft standard in 2022; still not final) | Library keeps accessibility targets as project-set values; do not pin to a draft — cite WCAG 2.2 (or current) per jurisdiction |

## 5. Sentimental ≠ childish

The senior project is the only artifact in this ecosystem with **advisor-signed release gates and a real end-of-service story** — it is evidence that the documentation chain (proposal → plan → change control → SRS → design → test plan → test record → traceability → executive summary) survives contact with an actual deadline-driven 2-person team. That is exactly the Tier 2–3 workload the new checklists describe, and it passed.

## Related

- [[TEMPLATE-INDEX]] — regenerated master index (360 templates)
- [[Phase 5 Quality Gate Report - 2026-09-24]] — pre-harvest quality baseline
- [[Template Restructure Plan - 2026-09-23]] — governing plan
- Source: `F:\books\Senior_Project\Transmatter_Platform.pdf` — Chiluek & Tippimwong, CMU CAMT, advisor Asst. Prof. Noppon Choosri, 11 Nov 2022
