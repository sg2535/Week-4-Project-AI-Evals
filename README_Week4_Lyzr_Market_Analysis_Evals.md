# Week 4 Project — Evaluating a Lyzr Market Analysis Agent with LangChain & LangSmith

## Project Overview

This project evaluates a **Market Analysis / Competitor Comparison Agent** built in **Lyzr** using a structured evaluation workflow powered by **LangChain** and **LangSmith**.

The goal is to measure how reliably the agent handles:

- Vendor comparisons
- Product and feature analysis
- Pricing and ROI questions
- Recommendation requests
- Missing or incomplete information
- Conflicting evidence
- Bias and leading prompts
- Hallucination-risk scenarios
- Executive-style decision support

The project focuses on **evaluation quality, failure analysis, and prompt improvement** rather than simply demonstrating that the agent can generate responses.

---

## Problem Statement

LLM-based market analysis agents can produce fluent responses, but fluency does not guarantee:

- factual grounding,
- balanced comparisons,
- correct uncertainty handling,
- resistance to biased prompts,
- or reliable decision support.

This project builds an evaluation harness around a Lyzr-based agent to identify where the system performs well, where it fails, and how targeted prompt changes can improve behavior.

---

## Solution Architecture

```text
Golden Evaluation Dataset
        |
        v
LangChain / Python Evaluation Runner
        |
        v
Lyzr Agent API
        |
        v
Market Analysis Agent
        |
        v
Model Prediction
        |
        v
LangSmith Tracing + Experiment Tracking
        |
        v
Human PASS / FAIL Review
        |
        v
Failure Analysis
        |
        v
Prompt Improvement
        |
        v
Regression Test Plan
```

---

## Technology Stack

- **Lyzr** — Market Analysis / Competitor Comparison Agent
- **LangChain** — Python-based evaluation integration
- **LangSmith** — Dataset management, tracing, experiment tracking, and evaluation evidence
- **Python** — API bridge and evaluation runner
- **Pandas / Excel** — Human review and PASS/FAIL analysis
- **GitHub** — Project documentation and submission assets

---

## Lyzr Agent Workflow

The evaluated solution uses a multi-agent market research workflow in Lyzr.

High-level flow:

```text
User Query
   |
   v
Orchestrator
   |
   +--> Competitor / Vendor Discovery
   |
   +--> Research / Evidence Collection
   |
   +--> Pricing & Feature Analysis
   |
   +--> Final Market Comparison / Recommendation
```

The evaluation focuses primarily on the **final observable behavior** returned by the Lyzr agent.

---

## Golden Dataset

A **40-case golden dataset** was created to test a broad range of expected behaviors.

### Coverage Areas

The dataset includes:

- Basic vendor comparisons
- Strengths and weaknesses
- Feature comparisons
- Integrations
- Enterprise suitability
- Recommendations
- Weighted decision criteria
- Pricing comparisons
- Total cost of ownership
- ROI
- Missing pricing
- Missing vendor information
- Incomplete data
- Ambiguous prompts
- Conflicting evidence
- Recency conflicts
- Bias / leading prompts
- Hallucination traps
- Market leadership claims
- Customer-reference claims
- Acquisition-risk questions
- Executive recommendations

### Example Test Case

```text
Test ID: MC029

Prompt:
Source 1 says Vendor A supports SSO; Source 2 says SSO requires
Enterprise licensing. Does Vendor A support SSO?

Expected Behavior:
Reconcile the evidence carefully, explain that SSO appears supported,
identify the licensing condition, and avoid overstating certainty.
```

---

## Evaluation Method

Each test case contains:

| Field | Description |
|---|---|
| Test ID | Unique evaluation case |
| Input / Test Prompt | Prompt sent to the Lyzr agent |
| Ground Truth / Expected Behavior | Human-defined expected behavior |
| Model Prediction | Actual Lyzr output |
| PASS / FAIL | Human evaluation |
| Failure Category | Assigned when a response fails |
| Run Status | Completed / Not Evaluated |

The project intentionally uses **behavior-based ground truth** rather than requiring an exact text match.

---

## LangChain → Lyzr Integration

A Python bridge was created so LangSmith could evaluate the external Lyzr agent.

Conceptually:

```python
def run_lyzr(inputs):
    question = extract_question(inputs)

    response = requests.post(
        LYZR_API_URL,
        headers=HEADERS,
        json={
            "user_id": USER_ID,
            "agent_id": AGENT_ID,
            "session_id": SESSION_ID,
            "message": question,
        },
    )

    response.raise_for_status()

    return {
        "answer": parse_lyzr_response(response.json())
    }
```

The input parser was made resilient to multiple dataset input formats, including:

```text
question
input.question
JSON-string encoded input
```

This resolved an early nested-input schema issue during experimentation.

---

## LangSmith Evaluation

LangSmith was used for:

- Golden dataset storage
- Experiment execution
- Trace inspection
- Input/output verification
- Failure evidence
- Reproducibility

A small one-row experiment was successfully validated before the larger baseline run.

---

## Baseline Results

The 40-case baseline dataset produced:

- **40 total golden test cases**
- **30 successfully executed Lyzr responses**
- **10 Not Evaluated**
- The remaining cases were blocked after the Lyzr API returned **HTTP 402 — Payment Required / Credits Exhausted**

The HTTP 402 cases were **not counted as model failures** because no model prediction was produced.

This distinction is important:

```text
Model failure != infrastructure / credit failure
```

Only executed model responses should be included in model-quality PASS/FAIL calculations.

---

## Human Evaluation

Human scoring was used to compare the model prediction against the expected behavior.

The review focused on questions such as:

- Did the answer satisfy the user request?
- Did the agent avoid unsupported claims?
- Did it handle missing information correctly?
- Did it distinguish facts from assumptions?
- Did it remain objective?
- Did it provide useful analysis instead of only asking follow-up questions?
- Did it properly reconcile conflicting evidence?

---

## Failure Analysis

The most important observed failure categories were:

### 1. Incomplete Task Fulfillment / Excessive Clarification

In some cases, the agent correctly recognized missing context but responded mostly with clarification questions instead of first providing the useful framework, risk factors, or decision variables that could already be explained.

Representative cases included:

- ROI analysis
- Acquisition-risk analysis

### 2. Overconfident Evidence Reconciliation / Insufficient Uncertainty Handling

In a conflicting-evidence case, the agent converted partially supported information into a more definitive conclusion than the evidence justified.

Representative case:

- SSO support vs. Enterprise licensing condition

### Most Important Improvement Area

The highest-value improvement was to make the agent:

1. provide useful analysis before requesting clarification,
2. distinguish verified, inferred, and unknown information,
3. qualify conclusions when evidence conflicts,
4. and avoid overstating certainty.

---

## Prompt Improvement

A targeted update was added to the Lyzr Orchestrator instructions.

### Added Rules

```text
EVIDENCE, UNCERTAINTY, AND CLARIFICATION RULES

1. Do not fabricate or assume unavailable facts.

2. Distinguish:
   - Verified
   - Inferred
   - Unknown / Not Verified

3. When sources conflict:
   - consider source authority,
   - recency,
   - product version,
   - licensing tier,
   - and applicability.

4. Do not respond only with clarification questions when useful
   analysis can still be provided.

5. If evidence is insufficient for a definitive recommendation,
   state that clearly and identify what additional information
   is required.

6. Maintain objective and neutral analysis.
```

The prompt change specifically targets:

- **Incomplete task fulfillment / excessive clarification**
- **Overconfident evidence reconciliation / insufficient uncertainty handling**

---

## Regression Test Plan

The planned regression set includes:

### Previously Failed Cases

- **MC021** — ROI
- **MC029** — Conflicting SSO evidence
- **MC038** — Acquisition risk

### Previously Passing Control

- **MC034** — Exact enterprise pricing unavailable

The passing control was selected to verify that the new prompt does not degrade the agent's existing anti-hallucination behavior.

---

## Regression Testing Limitation

The first post-fix regression case, **MC029**, was attempted after updating the Lyzr Orchestrator prompt.

However, Lyzr returned:

```text
HTTP 402
Credits exhausted
```

Therefore:

- no post-fix model prediction was produced,
- the remaining regression cases were not executed,
- no synthetic or manually invented outputs were substituted,
- and no false post-fix PASS/FAIL claims were made.

This limitation is documented as part of the project evidence.

---

## Evaluation Philosophy

This project follows several evaluation principles:

### 1. Separate Model Quality from Infrastructure Failures

API-credit failures are recorded as:

```text
Not Evaluated
```

not:

```text
FAIL
```

### 2. Evaluate Behavior, Not Exact Wording

A response can pass even if it does not exactly match a reference answer, provided it demonstrates the intended behavior.

### 3. Treat Uncertainty as a Feature

The agent should say:

```text
Not verified
```

or:

```text
Insufficient evidence
```

when the available information does not support a confident answer.

### 4. Regression Tests Should Include a Passing Control

This helps verify that a prompt improvement fixes failures without breaking behavior that already worked.

### 5. Do Not Fabricate Evaluation Evidence

If an experiment cannot run, document the constraint rather than generating artificial results.

---

## Project Files

Suggested repository structure:

```text
.
├── README.md
├── evaluation/
│   ├── golden_dataset.csv
│   ├── baseline_scoring.xlsx
│   └── failure_analysis.md
│
├── src/
│   └── run_40_dataset_eval.py
│
├── screenshots/
│   ├── langsmith_baseline.png
│   ├── failed_case_mc021.png
│   ├── failed_case_mc029.png
│   ├── failed_case_mc038.png
│   ├── prompt_before.png
│   ├── prompt_after.png
│   └── regression_credit_error.png
│
├── docs/
│   └── prompt_fix_regression_evidence.docx
│
└── requirements.txt
```

---

## Example Setup

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd <your-repository-folder>
```

### 2. Create a virtual environment

Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
```

macOS / Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

Typical packages:

```text
langchain
langsmith
requests
python-dotenv
pandas
openpyxl
```

### 4. Configure environment variables

Create a `.env` file:

```env
LANGSMITH_API_KEY=your_langsmith_key
LANGSMITH_PROJECT=your_project_name

LYZR_API_KEY=your_lyzr_key
LYZR_AGENT_ID=your_agent_id
LYZR_USER_ID=your_user_id
```

Do **not** commit `.env` to GitHub.

---

## Running the Evaluation

Example:

```bash
python src/run_40_dataset_eval.py
```

The runner:

1. loads the LangSmith dataset,
2. extracts the evaluation prompt,
3. calls the Lyzr agent,
4. returns the model prediction,
5. records the trace in LangSmith.

---

## Evidence Included in the Submission

The project submission includes:

- Golden dataset
- Completed human evaluation spreadsheet
- PASS / FAIL results
- Participant-assigned failure categories
- Failure analysis
- Before / after Lyzr prompt screenshots
- Prompt improvement summary
- LangSmith experiment evidence
- Representative failed-case traces
- Regression test plan
- Post-fix regression attempt
- Credit-exhaustion evidence
- Documented project limitations

---

## Optional LLM-as-a-Judge

The optional LLM-as-a-Judge extension was not included in the final project scope.

Human evaluation was used as the primary scoring method.

This kept the project focused on:

- reliable baseline evaluation,
- failure analysis,
- targeted prompt improvement,
- and transparent reporting.

---

## Key Learnings

This project reinforced several lessons about evaluating agentic AI systems:

1. A good evaluation dataset needs more than happy-path prompts.
2. Ambiguous and adversarial prompts reveal important agent behavior.
3. Safe refusal is not automatically a failure.
4. Excessive clarification can still be a quality issue.
5. Conflicting evidence requires calibrated uncertainty.
6. Prompt improvements should target observed failures, not hypothetical ones.
7. Regression testing should include both failed cases and previously passing controls.
8. Infrastructure failures must be separated from model-quality failures.
9. Evaluation evidence should remain truthful and reproducible.

---

## Future Improvements

Potential next steps include:

- Add LLM-as-a-Judge scoring
- Add automated faithfulness / groundedness checks
- Add more real-vendor evaluation cases
- Compare multiple LLMs
- Add cost and latency metrics
- Add automatic regression testing
- Add confidence scoring
- Add RAG-based evidence retrieval
- Expand the dataset with harder multi-vendor cases
- Re-run the post-fix regression suite when Lyzr inference credits are available

---

## Author

**Week 4 — Mastering Agentic AI Project**

Market Analysis Agent evaluation using:

**Lyzr + LangChain + LangSmith + Human Evaluation**

---

## Disclaimer

This project is an educational evaluation project. Vendor names such as **Vendor A**, **Vendor B**, and **Vendor C** are used in several test cases as placeholders to evaluate agent behavior under incomplete, ambiguous, or unsupported inputs.
