# **Comprehensive Inspect AI \+ Petri Research Report**

**For:** Codex (GPT-5.2), Gemini 3.0, Claude Code (Opus 4.5)  
**Date:** Dec 12, 2025  
**Subject:** AIGov Architecture Research Findings  
---

## **Executive Summary**

**The Bottom Line:** Inspect AI \+ Petri provides **85%** of the required "Engine" infrastructure (orchestration, execution, logging, basic scoring). The "Critical Gap" is **Reporting** and **Domain-Specific Configuration** (Scenarios/Dimensions).  
**Key Strategic Pivot:** Stop thinking about "building an audit engine." The engine exists. Shift engineering focus entirely to:

1. **Input Layer:** Porting GDPR/ISO logic into Inspect-compatible formats (Datasets/Prompts).  
2. **Output Layer:** Building the "Translation Layer" that converts Inspect's raw JSON logs into the L1/L2/L3 PDF reports.

**Feasibility:** The 3-4 week Phase 0 timeline is **highly feasible** if you avoid rebuilding the execution layer. A Phase 0 MVP can essentially be: `Inspect CLI` \+ `Custom GDPR Scenarios` \+ `Python Report Script`.  
---

## **Deliverable 1: Architecture Decision Matrix**

| Component | Assessment | Approach | Rationale & Implementation Details | Effort (Solo) |
| :---- | :---- | :---- | :---- | :---- |
| **Audit Execution** | **Perfect Fit** | **Leverage Inspect Core** | Inspect handles async loops, retries, provider rate limits, and tool execution natively. *Action:* Use `inspect eval` CLI for Phase 0\. | 0 Days |
| **Scenario Library** | **Key Asset** | **Leverage Petri Structure** | Petri's 111 scenarios are just a python list (`AUDITOR_INSTRUCTIONS`). We can inject our own. *Action:* Create `gdpr_scenarios.json` and pass via `-T special_instructions`. | 1 Day (Config) |
| **Transcript Analysis** | **Strong Fit** | **Leverage Petri Judge** | Petri's  alignment\_judge is sophisticated (XML-based citation parsing). *Action:* Reuse the class, but swap `DIMENSIONS` in  prompts.py for GDPR criteria. | 2 Days (Prompting) |
| **Legal Validation** | **Gap** | **Custom Tool (Solver)** | Inspect allows Solvers to call tools. *Action:* Build a simple Python `Tool` that queries your AKG/RAG, and give it to the `Judge` model. | 3-5 Days |
| **Report Generation** | **CRITICAL GAP** | **Custom Build** | Inspect outputs JSON/Log View. It does NOT make PDFs. *Action:* Build `report_gen.py` using Jinja2 \+ WeasyPrint/vpdf to consume `eval_log.json`. | **5-7 Days** |
| **Audit Supervision** | **Good Enough** | **Leverage VS Code Ext** | The Inspect VS Code extension allows real-time viewing of logs. *Action:* Use this for manual supervision in Phase 0\. | 0 Days |
| **Agent Testing** | **Native** | **Leverage Inspect Tools** | Inspect supports tool use natively. Petri's  auditor\_agent already implements the attack loop. *Action:* Use existing patterns. | 0 Days |
| **Multilingual** | **Native** | **Leverage Model Config** | Just configure the `system_prompt` to force the language. Inspect doesn't care. | \< 1 Day |

---

## **Deliverable 2: Optimal Data Flow Diagram**

**Philosophy:** Inspect is the "Running Engine". AIGov is the "Wrapper" (Inputs \+ Outputs).  
⚠️ Failed to render Mermaid diagram: Parse error on line 8  
`flowchart TD`  
    `subgraph Phase 0: Manual Intake`  
        `A[Client Requirement] --> B[Excel/JSON Config]`  
    `end`

    `subgraph "AIGov Input Layer"`  
        `B --> C{Scenario Selector}`  
        `C --> D[GDPR Scenarios (JSON)]`  
        `C --> E[ISO Scenarios (JSON)]`  
        `B --> F[Client Context Prompt]`  
    `end`

    `subgraph "Inspect AI Engine (The Black Box)"`  
        `G[inspect evaluate]`  
        `D & E & F --> G`  
        `H[Auditor Agent (Petri)] <--> I[Target Model (API)]`  
        `G --> J[Raw Transcript (Memory)]`  
        `J --> K[Alignment Judge (Petri Logic)]`  
        `K --> L[RAG/AKG Tool (Custom)]`  
        `K --> M[Scored Log (JSON)]`  
    `end`

    `subgraph "AIGov Output Layer (Value Add)"`  
        `M --> N[Log Parser / Mapper]`  
        `N --> O[Findings Database]`  
        `O --> P[L2: Legal Report (Jinja2)]`  
        `O --> Q[L3: Technical Report (Jinja2)]`  
        `O --> R[GDPR Annex Generator]`  
    `end`

    `subgraph Deliverables`  
        `P & Q & R --> S[Final Audit Pack (PDF/Docx)]`  
    `end`

**Key Data Transitions:**

1. **Config \-\> Inspect:** Passed via generic task arguments (`-T params=...`).  
2. **Judge \-\> Findings:** The Judge outputs `Scores` \+ `Highlights` (XML). The `Log Parser` extracts these highlights to populate the "Evidence" column in reports.

---

## **Deliverable 3: Extension Relevance Ranking**

| Extension | Rating | Relevance | Recommendation |
| :---- | :---- | :---- | :---- |
| **inspect-viz** | **7/10** | **High Utility for Dashboards.** Great for validitating model comparisons or aggregating results across multiple runs. Less useful for single-audit PDF generation. | Use for internal analytics or Phase 1 "Client Dashboard". |
| **inspect-transluce** | **5/10** | **Human-in-the-loop.** Provides a UI ("Docent") for annotating transcripts. | Keep on radar for "Analyst Review" interface in Phase 1\. |
| **inspect-cyber** | **4/10** | **Agent Sandboxing.** specialized for CTF/Cyber tasks. | Too complex for Phase 0 compliance audits unless you are testing code-generating agents specifically. |
| **inspect-swe** | **2/10** | **Benchmarks.** for software engineering. | Not relevant to AIGov. |

---

## **Deliverable 4: Phase 0 Revised Scope**

**Objective:** Validate the business model (sell the report), not the tech stack.  
**MUST Build (The "Thin" Layer):**

1. **GDPR Scenario Pack (JSON):** 10-15 prompts that specifically test GDPR Art 5 (Confidentiality) and Art 15 (Access).  
   * *Why:* Petri scenarios are safety-focused (bio-weapons, self-harm). We need compliance-focused.  
2. **GDPR Scoring Dimensions (Python):** A dictionary mapping GDPR principles to   
3. alignment\_judge criteria.  
   * *Why:* We need to score "Data Minimization Failure" not "Sycophancy".  
4. **The Reporter (Python):** A script that takes `eval_results.json` \-\> `L2_Report.pdf`.  
   * *Why:* This is what the client buys.

**MUST NOT Build (The "Trap"):**

1. **Custom Orchestrator:** Do not write python loops to call LLMs. Use `inspect eval`.  
2. **Custom UI:** Do not build a React frontend. Use VS Code \+ CLI.  
3. **Complex RAG (Phase 0):** Hardcode the legal references in the Judge prompt or Report template for the first 5 audits. Don't build a dynamic vector DB integration yet.

---

## **Deliverable 5: Critical Questions Answered**

**1\. What does Inspect actually provide beyond "run eval"?** It provides the entire plumbing: state management, concurrency, retry logic, provider standardization (switch OpenAI to Anthropic with 1 string change), and tool definitions. It handles the "boring" parts of LLM ops.  
**2\. Does Inspect generate professional reports?** **No.** It generates deeply technical JSON logs and a developer-centric HTML view (`inspect view`). It does not produce "Board Ready" PDFs. This is your core value-add build.  
**3\. Are we missing Inspect capabilities?** Yes. **Tools**. You can give the Judge (Scorer) tools. This allows the Judge to query your Legal Knowledge Graph (AKG) *during* scoring to validate if a response violates a specific article. This is a game-changer for automation.  
**4\. Multilingual Support?** Inspect is language-agnostic. Support is achieved by:

1. Configuring the Target Model with a Romanian System Prompt.  
2. Ensuring the Auditor (Petri) prompts are in Romanian (or finding that English prompts work against Romanian targets, which is often true).  
3. Configuring the Judge to output reasoning in the desired language (or English for the report).

**5\. Wrappers?** Inspect has native support for: OpenAI, Anthropic, Google, Mistral, AzureAI, Bedrock, Groq, Together, vLLM. You do not need to build wrappers.  
---

## **Deliverable 6: Risks & Mitigations**

**Risk 1: Prompt/Judge Tuning (The "Vibe" Problem)**

* *Issue:* The Judge might flag a valid medical disclaimer as a "Refusal" or fail to catch a subtle GDPR breach.  
* *Mitigation:* Dedicate 50% of Phase 0 time to *tuning* the Judge prompts (  
* prompts.py). This is the product quality bottleneck.

**Risk 2: Report Formatting is Hard**

* *Issue:* Converting JSON to a polished, paginated PDF with tables is tedious.  
* *Mitigation:* Use a robust template engine (Jinja2) and a headless browser renderer (WeasyPrint or Chrome headless) rather than building a PDF from scratch.

**Risk 3: Cost of Eval**

* *Issue:* Petri uses `Claude Opus` as the default Judge. This is expensive (€10-20 per audit run).  
* *Mitigation:* Switch Judge to `GPT-4o` or `Claude 3.5 Sonnet` for Phase 0\. They are capable enough for compliance classification and 10x cheaper.

---

## **Next Steps**

1. **Install:** Ensure `inspect-ai` and `petri` are working locally (Done).  
2. **Config:** Create `src/aigov/scenarios/gdpr.json` and `src/aigov/judge/gdpr_dimensions.py`.  
3. **Run:** Run a test eval with these custom configs.  
4. **Report:** Prototype the `json_to_pdf.py` script.

