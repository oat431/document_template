---
document_type: Thai PDPA Owner Checklist
version: "0.1"
status: Draft
author: "PO"
created: "2026-08-02"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
owner: "Deer_NGO / Channel Owner"
classification: "Internal"
tags: [thai-pdpa, privacy, owner-checklist, compliance, vrm, members, easydonate]
standard_ref:
  - Thailand Personal Data Protection Act B.E. 2562 (2019)
  - PDPC security, breach, erasure, and consent guidance
parent_project: "Deerngo Bot — VRM"
---

# Thai PDPA Owner Checklist — Deerngo Bot

> **Important:** This is a practical product/operations checklist, not legal advice or a legal certification. The channel owner should confirm the final position with qualified Thai privacy counsel. The Thai-language law and official Gazette text control over unofficial translations.

## 1. What This Means for the Owner

The channel owner normally decides:

- why viewers register;
- why donations are matched to handles;
- how points are calculated;
- whether a handle and score are published;
- how long data is retained; and
- how privacy requests are handled.

That normally makes the channel owner the likely **Data Controller** for this processing.

The developer/host may be a **Data Processor** only when operating under the owner's documented instructions and not reusing the data for an independent purpose. The actual facts matter more than the label in a contract.

A YouTube handle, stable YouTube/streamer.bot user ID, EasyDonate donor name, donation event, and points balance may be personal data when they identify or can be linked to a natural person. A public handle is not automatically exempt just because it is already visible on YouTube.

### PDPA topics the owner should verify

These are practical signposts to discuss with Thai counsel and to verify against the current Thai-language Act/official Gazette:

| Topic | Act sections commonly relevant | Deerngo implication |
|------|:------------------------------:|---------------------|
| Scope, personal data, controller/processor | Sections 5–6 | Handle, user ID, donor name, score, and linked events may be in scope; roles depend on who decides purposes/means. |
| Consent and lawful bases | Sections 19–24 | Registration, internal scoring, and public publication may need separate purpose/basis analysis. |
| Notice and indirect collection | Section 23 and Section 25 | Explain collection/use and address data received from EasyDonate or platform events. |
| Use/disclosure | Section 27 | Public scoreboard publication is a disclosure, not merely internal storage. |
| Data-subject rights | Sections 30–36 | Prepare access, correction, objection, withdrawal, restriction, and erasure/removal handling. |
| Security and breach | Section 37 | Use safeguards, deletion controls, incident escalation, and counsel-led breach assessment. |
| Records, processor agreement, DPO triggers | Sections 39–41 | Record processing responsibilities and document owner/developer instructions; confirm exemptions/obligations with counsel. |

The section mapping is a product-planning aid, not a legal conclusion. The Thai-language statutory text controls over unofficial translations.

---

## 2. Owner MUST Do Before Public Release

### 🔴 A. Decide and document the purpose

Write down the purposes separately:

- member registration through `:deer: register`;
- donation matching and points calculation;
- answering `:deer: point`;
- public scoreboard publication;
- security, fraud prevention, troubleshooting, and backups.

Do not silently reuse the data for unrelated marketing, profiling, resale, or another channel/project.

### 🔴 B. Approve a privacy notice

The notice should explain, in clear Thai/English appropriate to the audience:

- the channel owner/controller identity and contact method;
- what data is collected;
- that the actual streamer.bot identity is used for registration;
- that EasyDonate donor data is used for matching;
- why each category is processed;
- whether a field is required or optional;
- who receives the data, including the developer/host and providers;
- hosting/transfer locations where relevant;
- retention periods or deletion criteria;
- public scoreboard behavior;
- how to request access, correction, hiding, withdrawal, or deletion;
- what happens if a viewer does not register or does not want public display.

The existing registration message is only a short notice. It is not a complete privacy notice.

### 🔴 C. Choose a lawful basis for each purpose

Do not use one generic basis for everything. The owner, with counsel, should document the basis for:

| Purpose | Product requirement |
|---------|---------------------|
| Registration/membership | State the legal-basis decision and notice wording |
| Internal matching/points | Explain necessity and why the processing is reasonable |
| Public handle + score | Treat as a separate optional publication purpose; prefer explicit opt-in or document a defensible legitimate-interest assessment |
| Security logs | Define legitimate purpose, access, and retention |
| Legal/accounting records | Keep only if required and document the obligation |

Typing `:deer: register` is not automatically valid consent to every later use. If consent is used, it must be specific, informed, distinguishable, freely given, recorded, and easy to withdraw.

### 🔴 D. Provide a privacy control

The owner must provide a practical way for a member to:

- hide from the scoreboard;
- request correction of a wrong handle/point match;
- withdraw optional public display;
- request access to their stored member/donation-related data;
- request deletion/erasure where applicable; and
- contact the owner outside YouTube chat if the chat command is unavailable.

The Phase 1 product supports `:deer: private`, but the owner still needs a documented contact/removal process. A future admin console is not a reason to ignore requests now.

### 🔴 E. Set retention rules

Define and approve a field-level schedule before production:

| Data | Suggested product question |
|------|----------------------------|
| Active member record | How long is membership retained after inactivity or a removal request? |
| Inactive old member | When is it archived/deleted after manual point correction? |
| Raw EasyDonate donor name/message | How long is it needed for reconciliation/correction? Keep it only as long as necessary. |
| Donation reference/amount/time | Is there a legal/accounting reason to retain it? |
| Public scoreboard/cache | How quickly is hidden/deleted data removed from projections and caches? |
| Logs | When are logs rotated and securely deleted? |
| Backups | When do expired backups containing personal data disappear? |
| Consent/notice evidence | How is the notice version and time recorded? |

Do not write “keep forever” by default. A deletion schedule needs triggers, an owner, and a verification method.

### 🔴 F. Control the developer/host relationship

Create a written owner-to-developer data-processing agreement or equivalent instruction record covering:

- permitted purposes and fields;
- no independent data reuse;
- confidentiality and least privilege;
- security controls;
- subprocessors and hosting locations;
- assistance with access/correction/deletion requests;
- incident/breach escalation;
- retention, backup deletion, return, and destruction;
- audit/cooperation responsibilities.

Never send API keys, OAuth tokens, webhook secrets, or database passwords through ordinary chat or GitHub issues.

### 🔴 G. Prepare a breach response

The owner needs an incident contact and escalation procedure. The developer/host should notify the owner immediately after discovering a suspected incident, not wait for a statutory deadline. The owner and counsel determine whether regulator/data-subject notification is required.

At minimum, preserve evidence and assess:

- what data was exposed;
- whose data was affected;
- whether donor/payment information was involved;
- when the incident happened/discovered;
- containment and reset/rotation actions;
- notification obligations.

---

## 3. Owner MUST NOT Do

- ❌ Do not publish raw EasyDonate donor names on the scoreboard.
- ❌ Do not publish donation messages, payment identifiers, or exact donation records by default.
- ❌ Do not expose stable YouTube/streamer.bot user IDs publicly.
- ❌ Do not store YouTube display names when the MVP does not need them.
- ❌ Do not assume a public YouTube handle is automatically free of PDPA obligations.
- ❌ Do not treat lowercasing/removing `@` as anonymization.
- ❌ Do not use fuzzy matching to guess who donated; false attribution can harm a viewer.
- ❌ Do not transfer points automatically when a handle changes.
- ❌ Do not edit production points without a backup, transaction, reason, and before/after record.
- ❌ Do not use the data for another purpose without a new purpose/basis review and notice update.
- ❌ Do not keep raw donor names/messages forever without a documented need.
- ❌ Do not place personal data or secrets in GitHub issues, logs, screenshots, test fixtures, or public documents.
- ❌ Do not promise viewers that the system is “PDPA compliant” solely because it has a `:deer: private` command.

---

## 4. Minimum Privacy-Safe MVP Configuration

The owner should verify that the production configuration follows this model:

```text
Registration:
  actual streamer.bot identity → active member

Stored privately:
  stable user ID
  normalized current handle
  status/visibility
  registration timestamp
  points
  necessary donation/reconciliation record

Public scoreboard:
  active AND public_visibility=true AND total_points>0
  rank + normalized handle + total_points only

Private member:
  continues earning points
  excluded from scoreboard
  exact score not shown in public chat; 100-point band only

Handle change:
  old record inactive with old points
  new active record starts at 0
  owner/back office corrects points manually if appropriate
```

## 5. Owner Operational Checklist

### Before launch

- [ ] Controller/contact identity decided.
- [ ] Privacy notice approved and published in an accessible place.
- [ ] Purpose/legal-basis decision documented for registration, matching, scoring, and publication.
- [ ] Public scoreboard choice approved as separate optional purpose or documented legitimate-interest decision.
- [ ] `:deer: private` behavior tested.
- [ ] Contact/removal/correction route tested.
- [ ] Retention schedule approved for members, donations, logs, projections, and backups.
- [ ] Developer/host instructions and processor terms recorded.
- [ ] EasyDonate payload/authentication verified without exposing secrets.
- [ ] Security tests and response allowlist pass.
- [ ] Breach escalation contact and procedure documented.

### Per handle change

- [ ] Confirm the new registration came from the actual chat identity.
- [ ] Verify the old member is inactive and hidden.
- [ ] Verify the new member starts at 0.
- [ ] Decide manually whether a point correction is justified.
- [ ] If correcting, back up, transact, record before/after/reason/operator, and verify.
- [ ] Do not create an automatic alias or fuzzy match.

### When a viewer requests privacy

- [ ] Verify the request sufficiently to avoid changing the wrong member.
- [ ] Set `public_visibility=false` promptly when appropriate.
- [ ] Remove the member from scoreboard responses and caches.
- [ ] Assess whether logs, exports, backups, and provider-linked records need action.
- [ ] Record the request and outcome without unnecessary personal detail.
- [ ] Escalate legal exceptions/retention disputes to counsel.

---

## 6. Cautious Legal Notes

- This checklist is product guidance, not legal advice.
- Whether a specific field or processing activity is covered depends on facts and context.
- The owner should confirm the final lawful basis, notice, rights process, retention, international-transfer, controller/processor, and breach position with qualified Thai privacy counsel.
- Official PDPC pages and Thai-language legal/Gazette texts should be rechecked before launch; unofficial English translations do not override the Thai source.

## 7. Official Starting Sources

- [PDPC — Personal Data Protection Act, B.E. 2562 (2019)](https://www.pdpc.or.th/en/23134/)
- [PDPC — Consent guidance](https://www.pdpc.or.th/14/)
- [PDPC — Security Measures of the Data Controller](https://www.pdpc.or.th/en/22758/)
- [PDPC — Personal Data Breach notification criteria](https://www.pdpc.or.th/en/22783/)
- [PDPC — Erasure, destruction, or de-identification criteria](https://www.pdpc.or.th/en/22864/)
- [PDPC — English subordinate-legislation index](https://www.pdpc.or.th/en/notifications-of-the-personal-data-protection-committee/)
- [Official research notes used for this project](https://www.pdpc.or.th/en/23134/)

---

> **Owner sign-off:** ____________________  **Date:** ____________________
> **Legal review (if obtained):** ____________________  **Date:** ____________________
---
