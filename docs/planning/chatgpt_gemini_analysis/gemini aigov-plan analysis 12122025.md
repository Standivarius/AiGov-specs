# **AIGov Implementation Plan \- Deep Consistency Analysis**

## **Part 1: Executive Summary**

The AIGov implementation plan is built on a sophisticated, modern stack leveraging the **UK AISI Inspect** framework and **Anthropic’s Petri** auditing tool. This choice significantly reduces the engineering burden for Phase 0 by offloading the complex orchestration of "Auditor" and "Target" agents to proven open-source tooling. The architecture is sound, aligning well with the goal of automated governance audits.

However, the execution risk is elevated by **documentation lag** and **scope ambiguity**. The primary repository still references outdated concepts like "Mock Target LLM" (which Petri replaces) and lists critical components like the IntakeAgent as "Not Started" with only 3 weeks remaining. While the core auditing engine (Petri) is powerful, the wrapper systems (Intake, Reporting) are under-specified.

**Top 3 Recommendations:**

1. **Purge "Mock Target" References**: Explicitly update all docs to define "Target" as a model endpoint within the Inspect/Petri configuration, removing the need for a standalone "Mock Target" component.  
2. **Scope "IntakeAgent" Immediately**: This is the highest implementation risk. Define a "Wizard of Oz" version for Phase 0 (manual intake) to unblock the audit pipeline if the AI agent cannot be built in Week 4\.  
3. **Clarify "Judge" Role**: Distinguish between the *Petri Judge* (scoring interactions) and the *AIGov Judge* (compliance mapping). If they are the same, the "Not Started" status is misleading (it's configuration, not code). If different, the integration point is undefined.

---

## **Part 2: Cross-Document Inconsistencies**

| Inconsistency | Description | Impact | Resolution |
| :---- | :---- | :---- | :---- |
| **Target Architecture** | **README**: Lists "Mock Target LLM" as a Phase 0 goal. **Status**: User states "Adopted Inspect AI \+ Petri". Petri handles target interactions natively. | Wasted effort planning/building a mock target that is already solved by Petri. | **Remove "Mock Target" task.** Replace with "Configure Petri Target Endpoint". |
| **Eval-App Location** | **Structure**: projects/eval-app folder exists in file tree. **README**: Links Eval-app to external repo Standivarius/Aigov-eval. | Confusion on where testing code lives. Is it a submodule or a broken link? | **Use Git Submodules** or purely link externally. Delete empty folder if unused. |
| **"Petri" Scope** | **Naming**: Repo has Codex-Petri and CC-Petri. **Tooling**: "Petri" is now the Anthropic agent tool. | Ambiguity: Does "Petri" refer to the Anthropic tool or the AIGov components? | Rename internal components to Codex-Graph and Legal-Corpus to avoid overloading "Petri". |

---

## **Part 3: Architecture Integration Analysis**

### **1\. Core Engine: Inspect AI \+ Petri**

* **Purpose**: Orchestrate the "Auditor" (attacker) vs "Target" (SUT) interaction.  
* **Status**: Installed.  
* **Integration Gap**: The repository structure (projects/judge, projects/scenario-forge) implies custom tooling. It is unclear if ScenarioForge generates **Petri Seed Instructions** (JSON format) or if it's a separate pipeline.  
* **Recommendation**: Explicitly define ScenarioForge output as Petri Seed JSON compatible with Inspect.

### **2\. Judge (Multilingual)**

* **Purpose**: Analyze transcripts for EU AI Act violations.  
* **Ambiguity**: Petri includes a "Judge" agent (usually Claude Opus).  
  * *Scenario A*: You use Petri's Judge. (Task \= Prompt Engineering).  
  * *Scenario B*: You run a post-process custom Judge. (Task \= Python Dev).  
* **Risk**: If "Multilingual" capability is required, the default Petri Judge prompt may fail.  
* **Missing Spec**: Input format (Petri JSON logs) → Output format (behaviour\_json\_v1 mapping to EU/ISO standards).

### **3\. IntakeAgent**

* **Purpose**: Onboard clients and extract context.  
* **Status**: 🔴 Not Started.  
* **Disconnect**: The output of Intake (Context) must feed into **ScenarioForge** to customize the audit. This data flow (Intake JSON → Scenario Parametrization) is undocumented.

### **4\. ReportGen**

* **Purpose**: Generate L1/L2/L3 reports.  
* **Dependency**: Needs structured data from Judge.  
* **Status**: 🟡 In Progress.  
* **Verification**: Ensure ReportGen templates match the specific JSON output schema of the Petri/Inspect log exports.

---

## **Part 4: Phase 0 Feasibility Deep-Dive**

**Current Date**: Dec 12 | **Target**: Jan 3 (approx 3 working weeks)

* **Week 1: Setup & Config (Feasible)**  
  * *Task*: Configure Inspect/Petri.  
  * *Status*: "Inspect installed, Petri installed" (Done).  
  * *Focus*: Connect projects/akg and projects/rag if they are to be used as "Tools" for the Petri Auditor.  
* **Week 2: Scenarios & Judging (High Risk)**  
  * *Task*: ScenarioForge and Judge.  
  * *Bottleneck*: Creating high-quality "Seed Instructions" for Petri that are relevant to EU governance is a creative/legal task, not just coding.  
  * *Mitigation*: Use Petri's default seeds for the first "Tracer Bullet" to prove the pipeline, ignore custom scenarios for now.  
* **Week 3: Reporting & Dashboard (Medium Risk)**  
  * *Task*: ReportGen and Dashboard.  
  * *Feasibility*: Doable if the data format from Week 2 is stable. If the Judge output changes, this breaks.  
* **Week 4: Intake Agent (Critical Risk)**  
  * *Task*: Build Intake Agent.  
  * *Reality Check*: Building a robust "AI-dynamic questionnaire" in 1 week alongside holidays is unrealistic for a solo founder.  
  * *Pivot*: **Cut from Phase 0**. Use a static YAML config file for the "Client Context" instead of an interactive agent.

---

## **Part 5: Terminology & Naming Issues**

| Term Variation | Found In | Recommended Standard |
| :---- | :---- | :---- |
| **Mock Target / Target LLM** | README / Prompt | **System Under Test (SUT)** (Inspect standard) |
| **Petri** | Tool Name / Component Names | **Petri** (The Tool) vs **AIGov-Graph/Corpus** |
| **Judge / Judge LLM** | Projects / Specs | **Compliance Scorer** (to differentiate from Petri's "Safety Judge") |
| **IntakeAgent / Intake Agent** | README / Tables | **Intake Module** (Keep it simple) |

---

## **Part 6: Gap Analysis**

### **Technical Gaps**

1. **Petri Tool Definitions**: No documentation on how AKG and RAG are exposed to the Petri Auditor agent. Are they MCP (Model Context Protocol) tools? Python functions?  
2. **Data Schema Hand-off**: The JSON schema for the *handoff* between Petri's raw transcript log and the ReportGen engine is missing.  
3. **Multilingual Strategy**: No defined approach for forcing the Petri Auditor to test in non-English languages (e.g., system prompt modifications).

### **Process Gaps**

1. **Test Data**: "Mock Target" is deprecated, but what is the *actual* target? (e.g., gpt-4o or a local Llama 3?). Need a specific target model defined in config to run the first test.  
2. **Success Criteria**: "Phase 0 Complete" is defined by deliverables, but not by quality. Does the system need to *find* a violation, or just *run* without crashing?

---

## **Part 7: Prioritized Recommendations**

### **Tier 1: Fix Before Execution (Critical)**

1. **Define Phase 0 Target**: Set the target model to a publicly available endpoint (e.g., claude-3-sonnet) in inspect\_evals config. Don't build a mock.  
2. **Standardize Data Flow**: Write a specs/data-flow.md defining exactly what the Judge outputs (e.g., {"violation\_id": "EU\_ACT\_05", "severity": "High", "evidence": "..."}).  
3. **Rename Components**: Rename Codex-Petri to AIGov-Graph to avoid confusion with the Petri tool.

### **Tier 2: Fix During Phase 0 (Important)**

1. **Simplify Intake**: Downgrade IntakeAgent to "Static Config File" for the first iteration.  
2. **Leverage Inspect Evals**: Instead of building ScenarioForge from scratch, fork an existing inspect\_eval dataset and modify the "Seed Instructions" for governance topics.

### **Tier 3: Document for Later**

1. **Dashboard Integration**: Connect the Dashboard to read directly from Inspect's SQLite log database (standard output) rather than building a custom API layer immediately.

The adoption of **Petri/Inspect** is a massive accelerant. Your "Solo Founder" risk is manageable *only if* you lean entirely on these tools and stop trying to build parallel custom components (like the Mock Target or a complex Intake Agent) in Phase 0\.

