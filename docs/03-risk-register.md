# Meterwise: Risk Register

**Version 0.1 · Phase 1a · 30 September 2026 · Review at the start of every module**

Related documents: [Product brief](01-product-brief.md) · [System boundary diagram](02-system-boundary-diagram.md) · [README](../README.md)

## How to read this register

- **Likelihood (L)** and **Impact (I)** are each scored 1 (low) to 5 (high). **Score = L × I.**
- **Level:** High is 15 or above, Medium is 8 to 14, Low is 7 or below.
- Scores are judgement calls made before any code exists. Re-score after each module and after the first evaluation baseline.
- Mitigations reference acceptance criteria (AC-...) in the [product brief](01-product-brief.md).

## 1. Summary

| ID | Risk | Category | L | I | Score | Level |
|---|---|---|---|---|---|---|
| R02 | Extraction misreads digits or amounts | Accuracy | 4 | 4 | 16 | High |
| R03 | Stale or incorrect tariff and band data | Data | 4 | 4 | 16 | High |
| R01 | A false or misleading discrepancy finding | Accuracy and harm | 3 | 5 | 15 | High |
| R05 | Personal data exposure to the model provider, logs, or browser | Privacy | 3 | 5 | 15 | High |
| R04 | Output treated as legal advice, or over-reliance on it | Legal and safety | 3 | 4 | 12 | Medium |
| R07 | Prompt injection through a photographed or uploaded document | Security | 3 | 4 | 12 | Medium |
| R08 | API key exposure, abuse, or runaway cost | Security and cost | 3 | 4 | 12 | Medium |
| R09 | Wrong regulator or outdated complaint process | Regulatory | 3 | 4 | 12 | Medium |
| R10 | Exclusion of low-bandwidth, low-literacy, or disabled users | Accessibility | 4 | 3 | 12 | Medium |
| R12 | Data protection non-compliance | Legal | 3 | 4 | 12 | Medium |
| R13 | Automation bias: users trust AI output as fact | Safety and UX | 4 | 3 | 12 | Medium |
| R17 | Scope too large to sustain across the track | Delivery | 4 | 3 | 12 | Medium |
| R19 | Access to another user's case | Security | 2 | 5 | 10 | Medium |
| R11 | Model provider outage, rate limits, or latency spikes | Reliability | 3 | 3 | 9 | Medium |
| R14 | Inaccurate or inflammatory complaint drafts | Safety | 3 | 3 | 9 | Medium |
| R18 | Malicious or oversized uploads | Security | 3 | 3 | 9 | Medium |
| R06 | Prepaid token code leaked or misused | Privacy and fraud | 2 | 4 | 8 | Medium |
| R16 | Vendor lock-in to one AI provider | Platform | 2 | 3 | 6 | Low |
| R20 | Uniqueness claim proves wrong, or a competitor appears | Strategy | 3 | 2 | 6 | Low |
| R15 | Users submit edited or fabricated receipts | Abuse | 2 | 2 | 4 | Low |

## 2. Detail: mitigations, verification, and residual risk

| ID | Mitigation (design) | How it is verified | Residual risk |
|---|---|---|---|
| **R02** | Mandatory confirmation of every extracted field before analysis (gate G1). Schema and range validation. Per-field confidence with low-confidence fields highlighted. Manual entry always available. Client-side guidance for good photos. | AC-A1, AC-A2 on a 30-receipt evaluation set with seeded errors | Medium: poor photos will still cause errors, but they are caught by the user gate |
| **R03** | Reference data stores source URL and effective date for every band and tariff. The calculator refuses to run without a valid effective date. When data is missing, the product says "I can't confirm" and offers user-supplied rates labelled as such. A named owner reviews the data before each release. | AC-A4, unit tests for date handling, a release checklist item | Medium: data will still age between reviews, so the effective date is always shown |
| **R01** | All arithmetic in code. Findings worded as "does not match X", never as accusations. The output guard blocks accusatory and legal-conclusion wording. Every finding shows sources and confidence, plus the reasons a mismatch might be innocent (deductions, tariff changes, timing). | AC-A3, AC-A5 red-team suite | Medium: an unusual receipt format could still produce a wrong finding, so the user-facing framing stays cautious |
| **R05** | Consent notice before the first upload with a manual-entry alternative. Images are not persisted by default. Location metadata is removed in the browser and re-checked on the server. Server-side masking after extraction. Placeholders for identifiers are filled in the browser. No sensitive data in browser storage, logs, or URLs. Provider data-retention terms reviewed before choosing a provider. | AC-S3 automated scan of the bundle, network responses, and logs | **Medium-High:** the image must reach the provider for extraction, so a residual exposure remains. Manual entry is the privacy-preserving path |
| **R04** | Persistent "information, not legal advice" framing in the UI. Legal questions get general information plus a pointer to the regulator or a professional. No statements about lawfulness or liability. | AC-S1 test suite | Low-Medium |
| **R07** | Text from images and documents is treated strictly as data, never as instructions. Model output is validated against a schema. The model has no tools that can act outside the case (no sending, no browsing). The output guard checks for unexpected content. | AC-S2 adversarial suite of at least 20 samples | Medium: injection defences are never complete, so the blast radius is kept small by removing dangerous tools |
| **R08** | Provider keys live only in server environment variables and never in browser-exposed variables. Per-session and per-IP rate limits. Upload size caps. Spending limits and alerts set with the provider. | AC-S3, AC-S5 load test | Low-Medium |
| **R09** | Product copy avoids assuming a single national process. Procedure guidance is stored as sourced, dated data. Unknown DisCo or state leads to general guidance and a link to the official source. The complaint timeline and escalation route are re-verified before launch. | Release checklist item, content review | Medium until the process is confirmed for the pilot DisCo |
| **R10** | Plain-language writing. Compressed uploads and resumable transfers. Small JavaScript budget. WCAG 2.2 AA targets. Status never conveyed by colour alone. Manual entry as an alternative to photos. Language extension planned. | AC-X1 to AC-X6, AC-L4, AC-L5, AC-U5 | Medium: first release is English only |
| **R12** | Data minimisation, short case expiry, no image persistence by default, consent notice, and an opt-in for saved cases. Obligations under the Nigeria Data Protection Act 2023 confirmed before saved cases launch. | Legal review checklist item **[VERIFY]** | Medium until reviewed |
| **R13** | Confidence and sources on every finding. Language that invites checking ("this doesn't match", "worth asking about"). Mandatory review gates G1 to G3. Explicit statement of what the tool could not confirm. | AC-U4, usability testing | Medium: some users will still over-trust the output |
| **R17** | MVP limited to one journey ("Check my token") and one pilot DisCo. Extension points documented, not built. Each module adds one capability. | Review of scope at the start of each module | Medium |
| **R19** | Opaque, unguessable case IDs. Session-bound access checks on every request. Expiry. No case listing endpoint. | Access-control tests | Low |
| **R11** | Timeouts and bounded retries. Preserved progress. Manual entry continues to work without the model. Provider can be swapped through the AI SDK. | Failure-injection test | Low-Medium |
| **R14** | Drafts use neutral, factual templates and label facts as user-provided. The user edits and approves. Nothing is sent by the app. | AC-S4, AC-A5 | Low |
| **R18** | Server-side validation of type and size, image decoding limits, and no execution of uploaded content. | Upload abuse tests | Low |
| **R06** | Token code is masked on extraction, dropped from model input and output after extraction, never stored or logged, and never shown in full again. | AC-S3 scan for token-shaped strings | Medium: the code is visible in the image during extraction |
| **R16** | All provider code in one server module. Model identifier from configuration. Same evaluation set run against more than one provider. | Provider swap test in Phase 1b | Low |
| **R20** | Positioning based on the workflow (read receipt, verify, guide, draft), not on a claim of being the only tool. Re-run the competitor search before public claims. | Periodic search | Low |
| **R15** | The tool does not authenticate receipts and says so. Drafts label figures as user-provided. | Copy review | Low |

## 3. Top risks to watch first

1. **R02 and R03 (accuracy of inputs)**: everything downstream depends on correct fields and correct reference data.
2. **R01 (false finding)**: the highest-harm outcome. The design answer is code-based arithmetic plus cautious wording plus visible sources.
3. **R05 (personal data)**: the design answer is consent, no persistence, masking, and manual entry as an alternative path.

## 4. Open items that could change the scores

- Confirmation of the complaint process and regulator for the pilot DisCo (R09).
- Provider data-retention terms for image inputs (R05, R06, R12).
- First evaluation baseline for extraction accuracy (R02).
- Data protection review under the Nigeria Data Protection Act 2023 (R12).
