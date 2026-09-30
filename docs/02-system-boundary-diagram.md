# Meterwise: System Boundary Diagram

**Version 0.1 · Phase 1a · 30 September 2026**

This is the core evidence for the Phase 1a outcome: it shows where every piece of Meterwise runs (browser, server, model, data, tools) **before any frontend code is written**, and where trust boundaries sit.

Related documents: [Product brief](01-product-brief.md) · [Risk register](03-risk-register.md) · [README](../README.md)

> GitHub renders the Mermaid diagrams below automatically. If you view this file somewhere that does not, paste each code block into [mermaid.live](https://mermaid.live).

---

## 1. Boundary diagram

Solid arrows are data flows. The dashed arrow is an action the user takes themselves, outside the app.

```mermaid
flowchart LR
  subgraph B["BROWSER - untrusted zone"]
    UI["Capture and upload UI<br/>compress image, strip location metadata"]
    CONF["Confirm extracted fields<br/>edit or approve"]
    VIEW["Streaming results UI<br/>finding cards, checklist, case chat"]
    DRAFT["Draft editor<br/>fills placeholders from in-memory data"]
    PREF["UI preferences only<br/>language, text size"]
  end

  subgraph S["SERVER - Next.js route handlers on Vercel - trusted zone"]
    GATE["Input gate<br/>file type and size checks, rate limits, session"]
    MASK["Privacy layer<br/>mask token codes, IDs, phone numbers"]
    ORCH["Orchestrator<br/>prompts, tool calls, output schemas"]
    GUARD["Output guard<br/>sources required, no legal conclusions"]
  end

  subgraph T["TOOLS - called only by the orchestrator"]
    CALC["Unit calculator<br/>all money and energy maths in code"]
    LOOK["Reference lookup<br/>band and tariff by DisCo and date"]
  end

  subgraph M["MODEL LAYER - via Vercel AI SDK, provider swappable"]
    LLM["AI model<br/>vision extraction, explanation, drafting, Q and A"]
  end

  subgraph D["DATA LAYER - server side only"]
    REF[("Reference data<br/>bands and tariffs with source and effective date")]
    TPL[("Complaint templates and checklists<br/>versioned")]
    CASE[("Case store<br/>temporary with expiry, no raw images by default")]
  end

  subgraph X["EXTERNAL"]
    PROV["Model provider API"]
    OFFICIAL["DisCo and regulator websites<br/>link out only"]
  end

  UI -->|"image over HTTPS"| GATE
  GATE --> MASK
  MASK --> ORCH
  ORCH -->|"image and schema for extraction"| LLM
  LLM -->|"provider API call"| PROV
  LLM -->|"structured fields with confidence"| ORCH
  ORCH --> GUARD
  GUARD -->|"fields to confirm, token masked"| CONF
  CONF -->|"confirmed fields"| GATE
  ORCH -->|"tool calls"| CALC
  ORCH -->|"tool calls"| LOOK
  LOOK --> REF
  ORCH --> TPL
  ORCH <--> CASE
  ORCH -->|"numbers supplied, not computed by model"| LLM
  GUARD -->|"streamed findings, sources, confidence"| VIEW
  VIEW --> DRAFT
  DRAFT -.->|"user copies and sends it themselves"| OFFICIAL
  PREF -.- UI

  classDef client fill:#dbe4ff,stroke:#1f3fd1,color:#0b1220;
  classDef server fill:#e3f3e6,stroke:#2f7d44,color:#0b1220;
  classDef tool fill:#fff2c2,stroke:#a87b00,color:#0b1220;
  classDef model fill:#f3e1ff,stroke:#7b2fa8,color:#0b1220;
  classDef data fill:#e8e8e8,stroke:#555555,color:#0b1220;
  classDef ext fill:#ffe0dc,stroke:#b03a2e,color:#0b1220;
  class UI,CONF,VIEW,DRAFT,PREF client;
  class GATE,MASK,ORCH,GUARD server;
  class CALC,LOOK tool;
  class LLM model;
  class REF,TPL,CASE data;
  class PROV,OFFICIAL ext;
```

---

## 2. What runs where

| Piece | Runs in | Reason it lives there |
|---|---|---|
| Image capture, compression, location-metadata removal | **Browser** | Reduces upload size on poor connections and removes data before it leaves the device |
| Confirmation and edit forms for extracted fields | **Browser** | Human review gate (G1) |
| Streaming results, finding cards, evidence checklist, case chat UI | **Browser** | Presentation and interaction only |
| Draft editor and placeholder filling | **Browser** (in memory) | Identifiers are inserted locally so the model never needs to see the full values when drafting |
| UI preferences (language, text size, plain-language mode) | **Browser** storage | Non-sensitive |
| Upload validation, rate limiting, session handling | **Server** | Cannot be trusted to the client |
| Masking of token codes, IDs, and phone numbers | **Server** | Must happen before any text reaches the model or logs |
| Prompt assembly, tool orchestration, output schemas | **Server** | Keeps prompts and tool definitions private |
| Output guard (sources present, no legal conclusions, no accusations) | **Server** | Enforced independently of the model |
| Vision extraction, explanation, drafting, Q and A | **Model** | What the model is good at |
| Unit and cost calculations | **Tool (code)** | Deterministic and testable; the model never does arithmetic |
| Band and tariff lookup | **Tool + reference data** | Facts with a source and an effective date |
| Case store, templates, reference data | **Data layer** (server side) | Never delivered wholesale to the client |
| Sending a complaint | **User, outside the app** | Human control; the app never sends on the user's behalf |

---

## 3. Trust boundaries and what never crosses them

| Boundary | Never crosses it |
|---|---|
| **Browser to Server** | Nothing from the client is trusted without validation (file type, size, field ranges, schema) |
| **Server to Browser** | API keys, system prompts, tool definitions, other users' data, unmasked token codes, raw provider responses, internal logs |
| **Server to Model** | Unmasked identifiers beyond what extraction from the image requires, secrets, other users' data |
| **Model to Server** | Model output is treated as untrusted input: it is validated against a schema, checked for sources, and figures are compared with tool output |
| **Server to External** | Only masked or minimal data; no images stored by the provider integration by default |

---

## 4. Sequence: "Check my token"

```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant B as Browser
  participant S as Server
  participant M as AI model
  participant T as Calculator and reference data

  U->>B: Choose receipt photo
  B->>B: Compress to 1 MB or less and strip location metadata
  B->>S: Upload image over HTTPS
  S->>S: Validate type and size, apply rate limit
  S->>M: Extract fields from image using a fixed schema
  M-->>S: Structured fields with confidence
  S->>S: Validate schema, mask and drop token code
  S-->>B: Fields to confirm, token masked
  U->>B: Confirm or edit fields
  B->>S: Confirmed fields
  S->>T: Look up band and tariff, compute expected units
  T-->>S: Expected units with source and effective date
  S->>M: Explain the result, figures supplied not computed
  M-->>S: Plain language explanation, streamed
  S->>S: Output guard checks sources and wording
  S-->>B: Stream findings, sources, and confidence
  U->>B: Build my case and approve draft
  B->>B: Fill identifier placeholders locally from in-memory data
  U->>U: Copy the draft and send it outside the app
```

---

## 5. Design rules this diagram enforces

1. **The browser never talks to the model provider.** Every model call goes through the server.
2. **The model never does arithmetic.** Numbers come from the calculator tool and are passed to the model to explain.
3. **Every factual claim about bands, tariffs, or procedures carries a source and an effective date**, or the product says it cannot confirm.
4. **Nothing leaves the app without the user approving it**, and the app never sends anything on their behalf.
5. **Sensitive identifiers are masked after extraction and are never persisted in the browser.**
6. **Provider-specific code lives in one server module**, so switching between Anthropic and OpenAI does not touch the UI.
