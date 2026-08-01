---
document_type: Meeting Minutes
version: "1.0"
status: Final
author: "PO Persona"
created: "2026-08-02"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
meeting_type: "PO → Channel Owner — Thai PDPA Owner Readiness Handoff"
participants: ["PO Persona", "Deer_NGO / Channel Owner", "Developer / Host"]
classification: "Internal"
tags: [meeting-minutes, pdpa, privacy, owner, compliance, vrm, scoreboard]
standard_ref:
  - Thailand Personal Data Protection Act B.E. 2562 (2019)
  - PDPC consent, security, breach, and erasure guidance
---

# Meeting Minutes — PO → Channel Owner: Thai PDPA Owner Readiness

> **Date:** 2026-08-02
> **Type:** Privacy/product-readiness handoff
> **From:** PO Persona
> **To:** Deer_NGO / Channel Owner, Developer / Host
> **Status:** ✅ Final — owner checklist added; legal confirmation remains outstanding

---

## 1. Purpose

Add a practical Thai PDPA owner-readiness checklist to the Deerngo Bot overview and security documentation. The checklist explains what the channel owner should do and avoid before operating a public viewer scoreboard.

This document is not a legal opinion or a certification of PDPA compliance.

## 2. Key Privacy Position

The following may be personal data when they identify or can be linked to a natural person:

- YouTube/streamer.bot user ID;
- normalized YouTube handle;
- EasyDonate donor name;
- donation event and amount;
- points balance and public ranking; and
- operational logs/backups containing those fields.

A public handle does not automatically fall outside Thai PDPA merely because it is visible on YouTube. Normalizing a handle does not anonymize it.

## 3. Owner Responsibilities

### Must do

- Document each processing purpose separately.
- Publish a clear privacy notice and owner contact channel.
- Decide and document the lawful basis for registration, matching, points, and public display.
- Treat public handle/score publication as a separate optional disclosure purpose where appropriate.
- Provide hide, withdrawal, correction, access, and deletion/removal procedures.
- Define field-level retention and deletion for members, donor data, logs, caches, exports, and backups.
- Record written instructions and data-processing responsibilities with the developer/host.
- Prepare a breach escalation procedure.
- Obtain qualified Thai privacy counsel review before public launch if the owner needs legal certainty.

### Must not do

- Do not publish raw donor names, messages, payment identifiers, or exact donation records.
- Do not expose stable user IDs publicly.
- Do not store display names that the MVP does not need.
- Do not assume public handles are PDPA-free.
- Do not treat normalization as anonymization.
- Do not use fuzzy matching to guess identity.
- Do not change production points without backup, transaction, reason, and before/after record.
- Do not retain raw donor data forever without a documented purpose.
- Do not publish personal data or secrets in GitHub/issues/logs/fixtures.
- Do not claim “PDPA compliant” solely because `:deer: private` exists.

## 4. Documents Updated

| Document | Change |
|----------|--------|
| `external_overview/Deer Ngo Bot.md` | Added “Thai PDPA — Owner Responsibilities” section and official PDPC links |
| `06_security/063_thai_pdpa_owner_checklist.md` | Added detailed owner must/must-not checklist, launch checklist, retention, roles, breach, and rights guidance |
| `06_security/061_security_test_report.md` | Existing privacy/security release gates remain referenced |
| `07_pm/071_risk_register.md` | Existing R-006 privacy/PDPA risk remains a release gate |
| `external_plan/phase1-deerngo-bot-mvp.md` | Existing privacy gate and owner approval remain in Definition of Done/release gates |

## 5. Release Gate

The public scoreboard must not be considered production-ready until the owner has:

- [ ] approved the privacy notice;
- [ ] approved the public display behavior and lawful-basis decision;
- [ ] approved retention/deletion rules;
- [ ] confirmed the hide/correction/access/removal route;
- [ ] recorded developer/host processing instructions;
- [ ] reviewed breach escalation; and
- [ ] obtained Thai legal advice if legal certainty is required.

## 6. Official Starting Sources

- [PDPC — Personal Data Protection Act, B.E. 2562 (2019)](https://www.pdpc.or.th/en/23134/)
- [PDPC — Consent guidance](https://www.pdpc.or.th/14/)
- [PDPC — Security Measures](https://www.pdpc.or.th/en/22758/)
- [PDPC — Breach notification criteria](https://www.pdpc.or.th/en/22783/)
- [PDPC — Erasure/destruction/de-identification criteria](https://www.pdpc.or.th/en/22864/)

> Official Thai-language legal/Gazette text controls over unofficial translations. Re-check current regulator materials before launch.

---

> **Handoff status:** Owner-readiness checklist is documented. The checklist is guidance and a release gate, not a legal certification.
---

