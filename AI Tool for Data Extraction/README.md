# AI Tool for Data Extraction

## A Prompt-Engineering Artifact for AI-Assisted Structured Data Extraction from Scientific Articles


### Abstract

This folder contains the prompt-engineering artifacts that support the AI-assisted data-extraction pipeline described in the companion Systematization of Knowledge (SoK) study on tracking and accountability. The artifacts enable researchers to deploy a large-language-model (LLM) agent — specifically within **Microsoft Copilot Studio** — that reads scientific article PDFs, cross-references them against a bibliographic metadata file, and produces a standardised 26-column Markdown table conforming to a predefined Data Extraction Form (DEF) schema. The pipeline is designed to operate alongside a human researcher in a dual-extraction workflow, thereby enhancing reproducibility, auditability, and efficiency in systematic literature mappings (SLMs).


### 1. Purpose and Research Context

Systematic literature mapping require the extraction of structured data from large corpus of scientific articles. Manual extraction is labour-intensive and susceptible to human error and inconsistency.

This artifact addresses these challenges by providing a fully specified, reproducible prompt that instructs an LLM to perform structured extraction with strict guardrails against hallucination. The resulting output is a machine-readable table that can be directly integrated into a literature-mapping spreadsheet, where it is subsequently validated by a human researcher ("Researcher 1").

The dual-column design of the DEF schema (one column for the AI tool, one for the human researcher, for each extraction dimension) enables transparent comparison between AI-generated and human-generated extractions, facilitating inter-rater reliability analysis.

### 2. Repository Contents

This folder contains three Markdown files, each serving a distinct role in the deployment pipeline:

| # | File | Role | Description |
|---|------|------|-------------|
| 1 | `data_extraction_prompt.md` | **Standalone Prompt** | A self-contained data-extraction prompt designed for use with any LLM interface that accepts system-level instructions (e.g., ChatGPT, Claude, Gemini, or a local model). In this variant, the user attaches **both** the metadata file and the article PDF(s) at conversation time. |
| 2 | `data_extraction_agent_instructions.md` | **Agent System Prompt** | An adapted version of the extraction prompt optimised for deployment as a **Microsoft Copilot Studio agent**. In this variant, the bibliographic metadata file and the DEF schema reference document are pre-loaded into the agent's knowledge base; the user need only attach the article PDF(s). An accuracy directive is prepended to enforce low-temperature deterministic behaviour. |
| 3 | `data_extract_agent_creation_instructions.md` | **Deployment Guide** | A step-by-step implementation guide that describes how to configure and publish the extraction agent within Microsoft Copilot Studio, including field-by-field configuration instructions, temperature-approximation strategies, knowledge-source setup, and post-deployment maintenance considerations. |

#### 2.1 Relationship Between Files

```text
┌─────────────────────────────────────────────────────────────────┐
│              data_extract_agent_creation_instructions.md         │
│              (Deployment Guide — how to build the agent)        │
│                              │                                  │
│                              ▼                                  │
│              data_extraction_agent_instructions.md               │
│              (System prompt pasted into Copilot Studio)         │
│                              │                                  │
│                              ▼                                  │
│              Metadata file + DEF.pdf → Agent Knowledge Base     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              data_extraction_prompt.md                           │
│              (Standalone prompt — platform-agnostic)            │
│                              │                                  │
│                              ▼                                  │
│              User attaches metadata file + article PDFs         │
└─────────────────────────────────────────────────────────────────┘
```

### 3. The DEF Schema (26-Column Output)

The Data Extraction Form (DEF) defines a standardised 26-column Markdown table. The columns are listed below in their mandatory order. Intentional typographical variants in column headers (e.g., *Ryyan*, *Papper*, *Titlle*) are preserved to maintain schema compatibility with the original mapping spreadsheet.

| # | Column Header | Source |
|---|---------------|--------|
| 1 | Article ID | Metadata file |
| 2 | Ryyan ID | Metadata file |
| 3 | Dataset | Metadata file |
| 4 | Papper | User-supplied |
| 5 | DOI | Metadata file (fallback: article) |
| 6 | Titlle | Metadata file (fallback: article) |
| 7 | Year | Metadata file (fallback: article) |
| 8 | Objective of the Study (Researcher 1) | Human — left empty by AI |
| 9 | Objective of the Study (AI Tool) | **AI-extracted** |
| 10 | Proposed Solution and Tracking Details (Researcher 1) | Human — left empty by AI |
| 11 | Proposed Solution and Tracking Details (AI Tool) | **AI-extracted** |
| 12 | What is being tracked? (Researcher 1) | Human — left empty by AI |
| 13 | What is being tracked? (AI Tool) | **AI-extracted** |
| 14 | Architecture Type | **AI-extracted** |
| 15 | Technology Stack (Researcher 1) | Human — left empty by AI |
| 16 | Technology Stack (AI Tool) | **AI-extracted** |
| 17 | Purpose of Tracking (Researcher 1) | Human — left empty by AI |
| 18 | Purpose of Tracking (AI Tool) | **AI-extracted** |
| 19 | Responsibility Attribution (Researcher 1) | Human — left empty by AI |
| 20 | Responsibility Attribution (AI Tool) | **AI-extracted** |
| 21 | Main Findings and Conclusions (Researcher 1) | Human — left empty by AI |
| 22 | Main Findings and Conclusions (AI Tool) | **AI-extracted** |
| 23 | Study Limitations (Researcher 1) | Human — left empty by AI |
| 24 | Study Limitations (AI Tool) | **AI-extracted** |
| 25 | Notes (Researcher 1) | Human — left empty by AI |
| 26 | Notes (Researcher AI Tool) | **AI-extracted** |

The AI agent populates columns 9, 11, 13, 14, 16, 18, 20, 22, 24, and 26. Columns designated for "Researcher 1" are intentionally left empty to be completed by the human reviewer during the validation phase.

### 4. Hallucination Guardrails

A distinguishing feature of this prompt-engineering artifact is its comprehensive set of anti-hallucination guardrails (Section 5 of the extraction prompts). These constraints are summarised below:

| Guardrail | Description |
|-----------|-------------|
| **No fabrication** | Every entity in a cell must be literally present in the article or metadata file. |
| **Explicit missing-data tokens** | When data is unavailable, the agent must use one of: `NOT REPORTED`, `NOT APPLICABLE`, `NOT PROVIDED`, or `NOT FOUND IN METADATA FILE`. |
| **No external knowledge** | The agent must not supplement extractions with information from its training data. |
| **No silent inference** | Any inferred content must be explicitly tagged with `(inferred)` or prefixed with `Inferred:`. |
| **No translation drift** | Faithful translation is required; hedged language must be preserved. |
| **No editorial voice** | The agent must remain descriptive and neutral at all times. |
| **DOI / Title / Year integrity** | Strict precedence rules: metadata file → article → `NOT REPORTED`. |
| **Self-check before output** | The agent must internally verify column order, completeness, and traceability before emitting the table. |

### 5. Prerequisites

To deploy the **Copilot Studio agent** variant, the following are required:

- **Microsoft Copilot Studio** access (via a Microsoft 365 licence with Copilot entitlements).
- A **metadata file** (XLSX, CSV, PDF, or DOCX) containing bibliographic information for all articles in the review corpus, with at minimum the columns: `Article ID`, `Rayyan ID`, `DataBase ID`, `Article Title`, `Year`, and `DOI`.
- The **DEF schema reference document** (`DEF.pdf`) to be uploaded to the agent's knowledge base.
- **Article PDFs** named by their Article ID (e.g., `2-s2.0-85199216187.pdf`).

For the **standalone prompt** variant (`data_extraction_prompt.md`), any LLM interface that supports file attachments and system-level prompting may be used (e.g., OpenAI ChatGPT, Anthropic Claude, Google Gemini, or locally hosted models).

### 6. Quick-Start Guide

#### 6.1 Using the Standalone Prompt

1. Open your preferred LLM interface (e.g., ChatGPT, Claude, Gemini).
2. Paste the entire content of `data_extraction_prompt.md` as the system prompt or initial instruction.
3. Attach the metadata file and one or more article PDFs.
4. The model will return a single Markdown table with one row per article.

#### 6.2 Deploying the Copilot Studio Agent

1. Follow the step-by-step instructions in `data_extract_agent_creation_instructions.md`.
2. Paste the content of `data_extraction_agent_instructions.md` into the **Instructions** field of the agent configuration.
3. Upload the metadata file and `DEF.pdf` to the agent's **Knowledge** sources.
4. Publish the agent to **Microsoft 365 Copilot Chat**.
5. End users attach article PDFs in the chat interface; the agent returns the populated DEF table.


### 7. Intended Use and Limitations

- **Intended use.** This artifact is designed for academic research purposes, specifically to support systematic literature reviews. It is not intended for clinical, diagnostic, or decision-making applications.
- **Limitations.** The quality of AI-extracted data depends on the underlying LLM's capabilities, the clarity of the source articles, and the completeness of the metadata file. All AI-generated extractions should be validated by a human researcher before inclusion in published analyses.
- **Reproducibility.** The prompt is deterministic by design (low-temperature directives, structured output, anti-hallucination guardrails), but minor variations may occur across LLM providers and model versions.