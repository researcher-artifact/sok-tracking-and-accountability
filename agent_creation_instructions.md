# Deploying the Scientific Article Data-Extraction Prompt as a Microsoft Copilot Studio Agent: Implementation Guide

## 1. Overview and Purpose

The file [data\_extraction\_agent\_instructions.md](https://github.com/researcher-artifact/sok-tracking-and-accountability/blob/main/data_extraction_agent_instructions.md) contains a carefully engineered system prompt designed for automated extraction of structured data from scientific articles. 

## 2. Temperature Configuration for Maximum Accuracy

### 2.1 What Is Temperature?

The **temperature** parameter controls the degree of randomness (stochasticity) in the language model's token-selection process. A lower temperature makes the model more **deterministic**, favouring the highest-probability tokens; a higher temperature introduces more **variability**, which is useful for creative tasks but detrimental to factual extraction.

### 2.2 Current Limitation in Copilot Studio

As of the current platform version, **Microsoft Copilot Studio does not expose a direct temperature slider** in its agent configuration interface. The temperature value is managed by the underlying Microsoft Copilot platform and is not configurable as a standalone parameter. 

### 2.3 Recommended Workarounds for Maximum Accuracy

Since this agent performs **scientific data extraction** — a task that demands the highest possible precision and zero hallucination — the following strategies should be employed to approximate a **low-temperature behaviour (0.1–0.3)**: 

1.  **Explicit instruction in the system prompt.** Include a directive such as:
    > *"You must respond with maximum factual accuracy. Do not generate creative, speculative, or probabilistic content. Every claim must be directly traceable to the article text or the metadata file. Behave as if your temperature is set to 0.1."*

2.  **Leverage the guardrails as present in the prompt.** Section 5 of the prompt ("Guardrails against hallucination") contains strong anti-hallucination directives such as "No fabrication", "No external knowledge", and "No silent inference". These effectively constrain the model's output space, functionally reducing its effective temperature.

3.  **Use structured output requirements.** The prompt mandates a strict 26-column Markdown table with specific missing-data tokens (`NOT REPORTED`, `NOT APPLICABLE`, etc.). This rigid output schema further reduces the model's degrees of freedom. 

## 3. Step-by-Step: What to Fill in Each Copilot Studio Field

When creating the agent, you will use the **"Configure"** (manual) method in Copilot Studio. Below is a field-by-field guide:

### 3.1 Agent Name

| Field    | Recommended Value                   |
| -------- | ----------------------------------- |
| **Name** | `Scientific Article Data Extractor` |

This name will be visible to all users who interact with the agent.

### 3.2 Agent Icon

| Field    | Recommended Value                                                                                      |
| -------- | ------------------------------------------------------------------------------------------------------ |
| **Icon** | Select a scholarly or scientific icon (e.g., a magnifying glass over a document, or a data table icon) |

This is optional but improves discoverability. Click **"Change Icon"** on the configuration page to select or upload one. 

### 3.3 Description

| Field           | Recommended Value                                                                                                                                                                                                                                                                                                                        |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Description** | `An AI-powered assistant that extracts structured data from scientific article PDFs, cross-references them against a pre-loaded metadata file, and returns a standardised 26-column Markdown table conforming to the DEF schema. Designed for systematic literature reviews in information security, IoMT, and digital object tracking.` |

The description is **critically important** because it helps Copilot Studio's generative orchestration understand when and how to use this agent.

### 3.4 Instructions (System Prompt)

| Field            | Recommended Value                                                                                                                                    |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Instructions** | Paste the **entire prompt** avaiable in [data\_extraction\_agent\_instructions.md](https://github.com/researcher-artifact/sok-tracking-and-accountability/blob/main/data_extraction_agent_instructions.md)|

This is the most important field. The "Instructions" box in Copilot Studio accepts natural-language directives that shape the agent's behaviour

### 3.5 Primary Language

| Field                | Recommended Value |
| -------------------- | ----------------- |
| **Primary Language** | `English`         |

Since the prompt mandates that all extracted content must be written in **formal academic English** regardless of the original article's language, set the agent's primary language to English.

### 3.6 Knowledge Sources

| Field         | Recommended Value                                                                                                                                                              |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Knowledge** | Upload the **metadata file** (e.g., your XLSX or CSV file containing Article IDs, Rayyan IDs, DataBase IDs, Titles, Years, and DOIs) AND the **DEF.pdf** schema reference file |

1.  **Knowledge** → **+ Add Knowledge** → **Upload File**
2.  Upload the metadata file (e.g., `articles_metadata.xlsx`)
3.  **Name:** `Article Metadata — SLR Tracking and Accountability`
4.  **Description:** `Contains one row per scientific article with columns: Article ID, Rayyan ID, DataBase ID (Seed), Article Title, Year, DOI, Journal, Volume, Issue, Authors, Scopus URL, Language, Publisher. This file serves as the authoritative source for bibliographic metadata during data extraction.`
5.  Click **Add to Agent**

> ⚠️ **Important:** Supported file formats include XLSX, CSV, PDF, DOCX, TXT, and others. The maximum file size is **512 MB per file**, with up to **500 files per agent**. Encrypted or password-protected files are not supported.

### 3.7 Topics (Optional)

| Field      | Recommended Value                          |
| ---------- | ------------------------------------------ |
| **Topics** | Leave as default (generative answers mode) |

For this use case, you do **not** need to create custom topics. The agent should operate in **generative answers mode**, where it uses the instructions and knowledge sources to generate responses dynamically. The prompt itself is sufficiently structured to guide the agent's behaviour without topic branching.

### 3.8 Actions (Optional)

| Field       | Recommended Value                        |
| ----------- | ---------------------------------------- |
| **Actions** | None required for the initial deployment |

### 3.9 Generative AI Settings

| Field                        | Recommended Value |
| ---------------------------- | ----------------- |
| **Generative Answers**       | ✅ Enabled         |
| **Generative Orchestration** | ✅ Enabled         |

Navigate to **Settings** → enable **Generative Answers** and **Generative Orchestration**. This ensures the agent uses AI reasoning to process the uploaded PDFs against the knowledge base.

### 3.10 Authentication & Security

| Field              | Recommended Value                                                      |
| ------------------ | ---------------------------------------------------------------------- |
| **Authentication** | Configured per organisational policy (recommended: Microsoft Entra ID) |


### 3.11 Channels (Publishing)

| Field        | Recommended Value                                     |
| ------------ | ----------------------------------------------------- |
| **Channels** | Microsoft 365 Copilot Chat |

## 4. End-User Workflow After Deployment

Once the agent is published, the end user's workflow is:

1.  **Open the agent** in Copilot Chat
2.  **Attach one or more article PDFs** (the filename of each PDF, without the `.pdf` extension, must match the `Article ID` in the pre-loaded metadata file)
3.  **Wait for the agent** to process the articles, cross-reference the metadata knowledge base, and return the 26-column Markdown table
4.  **Copy the table** into the systematic literature mapping spreadsheet.

## 5. Maintenance Considerations

*   **Updating the metadata file:** Whenever new articles are added to the systematic literature mapping, the agent administrator must update the metadata knowledge source in Copilot Studio (Knowledge → select the file → replace/re-upload).
*   **Testing:** Use the built-in **Test Pane** in Copilot Studio to validate extraction quality before publishing updates.
*   **Monitoring:** After publication, monitor the agent's analytics to identify failed conversations or extraction errors.
