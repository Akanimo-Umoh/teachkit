# Meterwise: Product Brief

**Version 0.1 · Phase 1a (AI-Native Product Strategy & Frontend Architecture) · 30 September 2026**
**Author:** Akanimo Umoh · **Track:** Flexisaf AI Native Frontend (Advanced)

> **How to read this document.** Items marked **[VERIFY]** come from press coverage or older regulatory documents and must be confirmed against current NERC / DisCo sources before they are stated publicly or built into the product. Numbers in the acceptance criteria are initial targets and will be recalibrated after the first evaluation baseline.

Related documents: [System boundary diagram](02-system-boundary-diagram.md) · [Risk register](03-risk-register.md) · [README](../README.md)

---

## 1. Summary

**Meterwise is an AI-native copilot that helps Nigerian electricity customers understand what they were charged and know what to do if it looks wrong.**

A customer photographs a prepaid token receipt or an electricity bill. Meterwise reads it, asks the customer to confirm what it read, checks the figures against the applicable band and tariff using deterministic code, explains each charge in plain language, and, if something does not match, helps the customer build an evidence checklist and a complaint they review and approve before anything leaves the app.

**One-line pitch:** *Understand what you were charged. Know what to do if it's wrong.*

**What makes it AI-native rather than a chatbot bolted onto a calculator:**
- Multimodal input: the primary input is a photo of a receipt, not typed text.
- Structured, streamed output: findings appear as cards with confidence and sources, not a wall of chat text.
- Human-in-the-loop by design: extraction must be confirmed, and drafts must be approved.
- The model reasons and explains; **code does the arithmetic** and reference data provides the facts.

---

## 2. Problem and evidence

Electricity charges in Nigeria are hard for customers to verify, and the route to redress is unclear. Customers can end up paying silently for errors or giving up on disputes.

| Signal | What it shows | Source |
|---|---|---|
| Customers of a DisCo questioning whether the units credited match what they paid, and whether they are on the correct band | The verification problem is live and current | [ThisDay, 10 Sep 2026](https://thisdaylive.com/2026/09/10/bedc-electricity-why-are-customers-paying-more-for-less) |
| NERC ordered DisCos to refund customers wrongly billed at the new Band A rate (April 2024) | Billing errors are real enough for a regulator to order refunds | [Channels TV, 6 Apr 2024](https://www.channelstv.com/2024/04/06/new-tariff-fg-mandates-11-discos-to-refund-customers-wrongly-billed/) |
| Many consumers are uninformed about how to enforce their rights; the Electricity Act 2023 is reported to require exhausting regulatory dispute resolution before going to court | The redress path is poorly understood | [The Guardian (Nigeria)](https://guardian.ng/energy/how-electricity-consumers-can-get-reprieve-for-injustices-complaints/) **[VERIFY]** |
| Bands are defined by hours of supply (A: 20 to 24 h, B: 16 to 20 h, C: 12 to 16 h, D: 8 to 12 h, E: 4 to 8 h) | A customer's band determines their tariff and is itself a source of disputes | [VTpass explainer](https://vtpass.com/blog/how-to-check-electricity-band-understanding-electricity-tariff-in-nigeria-2026/) **[VERIFY]** |
| Band A tariff was reported to rise from ₦66 to ₦225 per kWh in 2024 | Tariffs change, so any reference data needs effective dates | [The Guardian (Nigeria)](https://guardian.ng/energy/how-electricity-consumers-can-get-reprieve-for-injustices-complaints/) **[VERIFY]** |

Estimated billing for unmetered customers has also been a long-running complaint in press and regulator documents **[VERIFY current figures before quoting any statistic]**.

---

## 3. Existing options and the gap

| Option | What it does | Why it is not enough |
|---|---|---|
| NERC outage reporting app | Lets customers report outages and track complaint resolution ([The Guardian](https://guardian.ng/news/electricity-consumers-empowered-to-monitor-discos-with-app/)) | Covers supply problems, not billing or token disputes |
| AfroTools calculators | Prepaid unit calculator and a tariff / bill reconciliation tool where the user enters the rate ([prepaid](https://afrotools.com/tools/prepaid-meter/nigeria/), [tariff](https://afrotools.com/tools/electricity-tariff/)) | The user must find and type in the tariff; no receipt reading; no guidance on dispute steps |
| Band and tariff explainers | Blog posts explaining how to check your band | Static; not personalised to the customer's receipt |
| DisCo portals and customer care | Official complaint channels | Not designed to help a customer decide whether to challenge a charge or assemble evidence |
| General-purpose AI chat | Can explain concepts | Not grounded in current tariff orders, so it can invent rates or rules |

**Gap:** I did not find a tool that reads the customer's own receipt or bill, checks it against sourced reference data, and then guides the customer through evidence and complaint steps. This is based on a limited search and is not proof that none exists. **[VERIFY]** by re-running the search before making any uniqueness claim. Also check the name "Meterwise" for existing products and domain availability.

---

## 4. Target users

| User | Situation | Needs and constraints |
|---|---|---|
| **Prepaid household customer** (primary) | Buys tokens through an app, USSD, or an agent. Sees fewer units than expected. | Mid-range Android phone, limited data, limited time. Plain language. Wants a quick answer: "is this normal?" |
| **Small business owner** (primary) | Runs a shop, salon, or cold room on prepaid or postpaid supply. Bills or unit counts swing without explanation. | Needs evidence organised for a complaint. Time-poor. Cares about cost. |
| **Helper** (secondary) | A family member or community volunteer helping someone less confident with technology. | Needs a flow simple enough to run on another person's phone without saving their details. |

Language for the first release: English, written at a plain reading level. Nigerian Pidgin and other languages are a later extension.

---

## 5. Jobs to be done

| ID | Job |
|---|---|
| J1 | When I buy a token and the units on my meter look low, I want to know whether they match what I paid, so I can decide whether it is worth challenging. |
| J2 | When I get a bill I do not understand, I want each charge explained in plain language, so I can tell what is normal and what is not. |
| J3 | When something looks wrong, I want to know what to do, who to contact, and what evidence to gather, so my complaint is not ignored. |
| J4 | When I have complained, I want to track deadlines and next escalation steps, so I do not lose track. |
| J5 | On my phone, on a poor connection, I want to do all of this without giving away more personal data than necessary. |

---

## 6. Core AI use cases

| ID | Use case | Model's role | Non-model parts | Human check |
|---|---|---|---|---|
| A1 | **Read a receipt or bill** | Vision extraction into a structured schema, with per-field confidence | Schema validation, range checks, masking of sensitive fields | User confirms or edits every extracted value before analysis |
| A2 | **Explain charges in plain language** | Generate an explanation grounded in confirmed fields and reference data | Reference lookup supplies band, tariff, source, and effective date | Sources and confidence are visible on every finding |
| A3 | **Detect discrepancies** | Explain what a mismatch might mean and what to ask | A **calculator tool** computes expected units and differences; the model never does the arithmetic | Findings are worded as "does not match X", never as accusations |
| A4 | **Build an evidence checklist** | Tailor the checklist to the finding | Rules and templates define the baseline checklist | User ticks items as done |
| A5 | **Draft a complaint** | Draft from a template using confirmed facts | Placeholders for identifiers are filled in the browser, not by the model | User edits and explicitly approves; nothing is sent automatically |
| A6 | **Answer questions about the case** | Grounded Q&A using the case, reference data, and tools | Refusal rules for legal or out-of-scope questions | Answers cite sources or say "I can't confirm that" |

---

## 7. Non-goals

- No legal advice, representation, or statements that a DisCo acted unlawfully.
- No automatic sending of complaints to a DisCo or regulator. Delivery is always user-initiated and outside the app.
- No token purchase, bill payment, or wallet.
- No smart-meter or DisCo system integration, and no scraping of DisCo portals (link out only).
- No storing of meter numbers, account numbers, token codes, or receipt images in the browser.
- No public naming or ranking of DisCos or individual customers.
- First release is English only.

---

## 8. MVP scope and how later modules extend it

**MVP: "Check my token."** Upload a prepaid token receipt (and optionally a photo of the meter's displayed units), confirm the extracted fields, get a deterministic expected-units check with a plain-language explanation and sources, then follow guided next steps and an approved complaint draft if there is a mismatch.

**Pilot DisCo:** to be chosen in Phase 1b based on which DisCo's real sample receipts are available for testing. The data model is DisCo-agnostic from day one.

**Planned extension points** (curriculum module names to be confirmed; this list describes what the architecture is designed to absorb):

| Extension | What it adds | Where it plugs in |
|---|---|---|
| Streaming responses | Progressive findings and explanations | Server orchestrator streams to result cards |
| Tool calling | Model-initiated calculator and reference lookups | Tool layer (already separated from the model) |
| Generative UI | Finding cards, checklists, and cost breakdowns rendered from structured output | Client result components |
| Retrieval | Search over tariff orders and complaint procedures | Data layer behind the reference lookup tool |
| Accounts and persistence | Saved cases, reminders, follow-ups | Case store and auth |
| Evaluations and guardrails | Automated accuracy, safety, and injection tests | Test harness against the acceptance criteria |
| Observability | Latency, cost, and failure monitoring without personal data | Server logging |

Later scope (not MVP): postpaid bill analysis (J2), more DisCos, Nigerian Pidgin, saved cases with reminders (J4).

---

## 9. User journeys

**Journey A: Check a token (primary)**
1. User opens Meterwise on their phone and chooses "Check my token".
2. A short notice explains that the photo will be sent to an AI service, and offers **manual entry** as an alternative that sends no image.
3. User takes or selects a receipt photo. The browser compresses it and strips location metadata.
4. Meterwise shows what it read, with the token code masked. Low-confidence fields are highlighted.
5. User confirms or edits the fields. **Analysis does not start until they confirm.**
6. Meterwise looks up the band and tariff, computes expected units in code, and streams a plain-language explanation with sources and effective dates.
7. If it matches: a reassuring summary and how to keep an eye on future tokens.
8. If it does not match: a finding card, "Build my case", an evidence checklist, and a complaint draft the user edits and approves.
9. User copies or exports the approved draft and sends it themselves.

**Journey B: Understand a bill (later)**: upload a bill, confirm fields, get line-item explanations and a check for estimated readings and band consistency.

**Journey C: Ask a question**: the user asks "why did my units drop?" in the case chat. The answer uses the case data and reference data with citations, or says it cannot confirm.

**Journey D: Follow up**: the user returns to a saved case, logs the DisCo's response, and gets the next escalation step and deadline **[VERIFY escalation route and timelines; the 15-day window and Forum Office appeal come from an older NERC document]**.

---

## 10. Interaction flows

### 10.1 Chat
Case-scoped chat, always visible after the first upload. Suggestion chips offer common questions. Answers stream in and cite sources.

### 10.2 Assistive actions (buttons that act on the case)
"Check my units", "Explain this charge", "Build my case", "Draft complaint", "Make it simpler".

### 10.3 Review and approval gates

| Gate | What the user reviews | Rule |
|---|---|---|
| G1 Extraction confirmation | Every field read from the image | Mandatory before analysis |
| G2 Finding review | The finding, its sources, and its confidence | Sources visible within one tap |
| G3 Draft approval | The complaint text with identifiers shown | Explicit "Approve" action; nothing is sent by the app |
| G4 Export | Final text or file | User-initiated copy or download |

### 10.4 Fallback states

| Trigger | What the user sees | Recovery |
|---|---|---|
| Image unreadable or not a receipt | "I couldn't read this photo." with tips on lighting and framing | Retake, or switch to manual entry |
| Low-confidence or invalid field | Field highlighted with the reason | Edit the value; analysis is blocked until resolved |
| DisCo or band not supported or unknown | "I don't have current tariff data for this DisCo." | General explanation only, plus a link to the official source |
| No tariff with a valid effective date | "I can't confirm the rate for this date." | User can enter the rate from their own source; the finding is labelled as user-supplied |
| Legal or out-of-scope question | A short answer of general information and where to ask a professional | Link to regulator contact information |
| Suspected prompt injection in a document | The document text is ignored as instructions; a neutral notice is shown | User can continue with manual entry |
| Model or network error | Plain message with what was saved | Retry; progress is preserved; manual entry remains available |
| Rate limit reached | Message with when they can try again | Wait, or continue with manual entry |
| Slow connection | Progress indicator and resumable upload | Automatic retry; the user is never asked to start over |

---

## 11. System boundaries (summary)

The full diagram is in [02-system-boundary-diagram.md](02-system-boundary-diagram.md).

| Layer | Responsibilities | Must not |
|---|---|---|
| **Frontend** (Next.js client components) | Capture and compress images, strip EXIF, render streaming results, collect confirmations, hold UI preferences, fill draft placeholders from in-memory data | Hold API keys, call the model provider directly, persist sensitive fields, perform tariff maths, decide findings |
| **Backend** (Next.js route handlers on Vercel) | Validate uploads, rate-limit, mask sensitive data, orchestrate model calls and tools, enforce output schemas, require sources, log without personal data | Trust client-supplied values without validation, log images or token codes |
| **Model layer** (via Vercel AI SDK) | Vision extraction, plain-language explanation, drafting, grounded Q&A | Perform arithmetic on money or energy, invent tariffs, state legal conclusions |
| **Data layer** (server-side) | Reference data (bands and tariffs with source and effective date), complaint templates, temporary case store | Store raw images by default, expose one user's case to another |
| **External tools and services** | Model provider API, later a database or key-value store; DisCo and regulator sites as link-outs only | Receive unmasked identifiers beyond what extraction requires |

---

## 12. Privacy: data that must not be exposed to the browser

| Data | Rule |
|---|---|
| Provider API keys and secrets | Server-only environment variables. Never in the client bundle. Never in variables prefixed for browser exposure. |
| System prompts, guardrail rules, tool definitions | Server-only. |
| Prepaid token code on a receipt | Treated as a secret, since it can be loaded onto a meter by whoever holds it **[VERIFY]**. Masked on extraction, never stored, never repeated in model output, never logged. |
| Meter and account numbers | Masked (last four digits) in the UI after extraction. Full values exist only in server memory for the request. Drafts use placeholders that the browser fills from in-memory data and never writes to storage. |
| Names, addresses, phone numbers | Same handling as meter numbers. |
| Receipt and bill images | Sent over HTTPS for extraction. Not persisted by default. Location metadata removed in the browser and re-checked on the server. |
| Case data | Held server-side under an opaque, unguessable ID with an expiry. Never accessible across sessions. |
| Other users' data | Never returned to any client. |
| Logs | Structured events only; no images, identifiers, or token codes. |

**Consent and control:** a notice appears before the first upload explaining that the photo goes to an AI service, with manual entry available as an alternative. Applicable obligations under the Nigeria Data Protection Act 2023 (lawful basis, retention, cross-border transfer) must be confirmed **[VERIFY]**.

**Known limit:** a receipt image necessarily contains identifiers when it reaches the model provider for extraction. Mitigations are consent, manual entry, provider data-retention terms, no image persistence, and masking of everything after extraction. See risk R05 in the [risk register](03-risk-register.md).

---

## 13. Acceptance criteria

Targets are initial and will be recalibrated after the first baseline evaluation.

### Accuracy
| ID | Criterion | Target | How measured |
|---|---|---|---|
| AC-A1 | Extraction accuracy for amount paid, units credited, and date | At least 95% exact match | 30-receipt evaluation set across lighting, angles, and DisCo formats |
| AC-A2 | No silently wrong figures | 100% of fields that fail validation or fall below the confidence threshold are flagged for confirmation | Automated tests with seeded errors |
| AC-A3 | Arithmetic integrity | 100% of money and energy figures shown come from the calculator tool; the shown value equals the tool value | Unit tests plus an automatic check on model output |
| AC-A4 | Grounding | 100% of tariff, band, and procedure claims show a source and effective date, or say "I can't confirm" | Prompt test suite |
| AC-A5 | Finding language | Zero uses of accusatory or legal-conclusion wording (for example "overcharged", "illegal", "fraud") | Red-team suite of at least 20 cases |

### Latency
| ID | Criterion | Target | How measured |
|---|---|---|---|
| AC-L1 | Immediate feedback after any action | Within 300 ms | Browser performance trace |
| AC-L2 | First streamed explanation text after confirmation | 2.5 s or less at p75 on a throttled slow-4G profile | Synthetic test |
| AC-L3 | Extraction result for a compressed image | 10 s or less at p90 | Synthetic test |
| AC-L4 | Upload size | 1 MB or less after client-side compression | Automated check |
| AC-L5 | Initial JavaScript for the main flow | 150 KB gzipped or less | Build analysis |

### Accessibility
| ID | Criterion | Target | How measured |
|---|---|---|---|
| AC-X1 | Conformance | WCAG 2.2 AA for the core flow | Automated audit with zero critical issues, plus manual review |
| AC-X2 | Keyboard and focus | Every action operable by keyboard with visible focus | Manual test |
| AC-X3 | Touch targets | At least 44 px | Design review |
| AC-X4 | Reflow and zoom | Usable at 320 px width and 200% text size | Manual test |
| AC-X5 | Dynamic content | Streaming updates announced politely to screen readers; status never conveyed by colour alone | Screen reader check |
| AC-X6 | Plain language | Explanations at about grade 8 reading level or lower | Readability tool |

### Safety
| ID | Criterion | Target | How measured |
|---|---|---|---|
| AC-S1 | No legal advice | Legal questions receive general information and a pointer to a professional or the regulator | Test suite |
| AC-S2 | Prompt injection | 0 instruction-following from document content across at least 20 adversarial samples | Adversarial suite |
| AC-S3 | Secrets and personal data | None in the client bundle, unrelated network responses, or logs | Automated scan |
| AC-S4 | Human approval | 100% of outbound drafts require explicit approval; nothing is sent by the app | Flow test |
| AC-S5 | Abuse limits | Per-session and per-IP rate limits enforced | Load test |

### UX quality
| ID | Criterion | Target | How measured |
|---|---|---|---|
| AC-U1 | Task success | At least 80% of at least 5 target users complete "Check my token" unaided in 3 minutes | Moderated usability test |
| AC-U2 | Error messages | Every fallback in section 10.4 says what happened and what to do next | Content review |
| AC-U3 | Correction effort | Any extracted value can be corrected in 2 taps or fewer | Flow test |
| AC-U4 | Transparency | Sources and confidence reachable within 1 tap from any finding | Flow test |
| AC-U5 | Resilience | Progress survives a dropped connection; works on a 360 px Android browser and iOS Safari | Device test |

---

## 14. Early state-management decisions

| State | Decision | Why | Impact on later modules |
|---|---|---|---|
| **Unit of state** | A **Case** (evidence, confirmed fields, findings, drafts, timeline). Chat is a view over a case, not the source of truth. | Users think in cases, not conversations | Saved cases, reminders, and follow-ups build directly on this |
| **Conversation history** | The server holds the authoritative history per case under an opaque ID. The client keeps rendered messages in memory using the AI SDK's chat state and rehydrates from the server. | Keeps sensitive content out of browser storage | Streaming and tool-calling modules plug into the same message shape |
| **Session state** | Anonymous session via an httpOnly, secure, same-site cookie holding an opaque ID and no personal data. Case data expires (initially 24 hours) unless the user saves it. | Privacy by default | Auth later replaces the anonymous session for saved cases |
| **Saved preferences** | Non-sensitive UI preferences only (language, plain-language mode, text size, chosen DisCo) in browser storage | Useful and low risk | Preference sync when accounts arrive |
| **Saved cases** | Opt-in, requires sign-in. Storage provider (database or key-value store) decided in Phase 1b. Field-level encryption for identifiers. | Vercel functions are stateless, so durable state needs an external store | Persistence module |
| **In-flight upload state** | Client-only, cleared on completion or cancel | Avoids lingering images | None |
| **Drafts** | Autosaved server-side in the case with placeholders, not in browser storage | Drafts may contain personal text | Version history later |
| **Client state tooling** | React state and context first, plus the AI SDK chat state. Introduce a store only when distant components must share state. | Avoids premature complexity | Revisit at the generative UI module |

**Data model sketch (not final):**

```ts
type Case = {
  id: string;                  // opaque, unguessable
  discoId?: string;
  bandId?: string;
  evidence: { id: string; kind: "token_receipt" | "bill" | "meter_photo"; stored: boolean }[];
  fields: Record<string, { value: string | number; confidence: number; confirmed: boolean }>;
  findings: { id: string; summary: string; sources: { title: string; url: string; effectiveDate: string }[]; confidence: number }[];
  drafts: { id: string; body: string; approved: boolean }[];
  timeline: { at: string; event: string }[];
  expiresAt: string;
};
```

---

## 15. AI platform decision

- **Building against:** Anthropic (Claude) through the **Vercel AI SDK**.
- **Why for this product:** strong document and image understanding for receipts, reliable structured output, and cautious behaviour on legal and safety questions.
- **This choice is swappable.** The Vercel AI SDK keeps frontend code the same regardless of provider, so moving to OpenAI or another provider means changing the provider package and configuration, not rewriting the UI. All provider-specific code will live in a single server module, and the model identifier will come from an environment variable rather than being hard-coded.
- **Before finalising**, run the same evaluation set against both Anthropic and OpenAI and keep the provider that best meets AC-A1 to AC-A5 and AC-L2 to AC-L3.
- Reference docs: [Anthropic Messages API](https://platform.claude.com/docs/en/api/messages), [OpenAI Responses API](https://platform.openai.com/docs/api-reference/responses), [Vercel AI SDK](https://ai-sdk.dev/docs/introduction), [AI SDK providers](https://ai-sdk.dev/providers/ai-sdk-providers).

---

## 16. Success metrics

| Type | Metric | Initial target |
|---|---|---|
| Outcome | Pilot users (10 to 20 real customers) who say they understand their charge better after a check | 70% or more |
| Quality | Extractions confirmed without edits | Tracked as a baseline, then improved |
| Quality | Acceptance criteria passing | All safety criteria (AC-S1 to AC-S5) at 100% before any public release |
| Engagement | Checks that reach a finding or "all clear" | Tracked as a baseline |
| Action | Findings that lead to an approved draft | Tracked as a baseline |
| Delivery | A working Vercel deployment at the end of every module | Every module |

---

## 17. Assumptions, open questions, and items to verify

| # | Item | Status |
|---|---|---|
| 1 | Pilot DisCo | Open, decided in Phase 1b by availability of real sample receipts |
| 2 | Complaint timelines and escalation route, and whether state regulators change the process | **[VERIFY]** |
| 3 | Current tariff and band data for the pilot DisCo, with source and effective date | **[VERIFY]** |
| 4 | Whether token codes can be loaded by anyone holding them (drives masking rules) | **[VERIFY]** |
| 5 | Nigeria Data Protection Act 2023 obligations for storing, retaining, and transferring images and identifiers | **[VERIFY]** |
| 6 | How consistently DisCos show deductions (fixed charges, tax, debt recovery) on receipts, which decides the extraction schema | Open, checked with sample receipts |
| 7 | Competitor re-check and name / domain availability for "Meterwise" | **[VERIFY]** before any public claim |
| 8 | Which provider (Anthropic or OpenAI) performs better on the evaluation set | Open, decided after baseline evaluation |

---

## Sources

- ThisDay: [BEDC electricity: why are customers paying more for less](https://thisdaylive.com/2026/09/10/bedc-electricity-why-are-customers-paying-more-for-less)
- Channels TV: [New tariff: FG mandates DisCos to refund customers wrongly billed](https://www.channelstv.com/2024/04/06/new-tariff-fg-mandates-11-discos-to-refund-customers-wrongly-billed/)
- The Guardian (Nigeria): [How electricity consumers can get reprieve for injustices, complaints](https://guardian.ng/energy/how-electricity-consumers-can-get-reprieve-for-injustices-complaints/)
- The Guardian (Nigeria): [Electricity consumers empowered to monitor DisCos with app](https://guardian.ng/news/electricity-consumers-empowered-to-monitor-discos-with-app/)
- VTpass: [Electricity tariff in Nigeria and how to check your band](https://vtpass.com/blog/how-to-check-electricity-band-understanding-electricity-tariff-in-nigeria-2026/)
- AfroTools: [Nigeria prepaid meter calculator](https://afrotools.com/tools/prepaid-meter/nigeria/) and [electricity tariff calculator](https://afrotools.com/tools/electricity-tariff/)
