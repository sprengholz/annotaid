# NOTA: A browser-based tool for LLM-assisted qualitative coding of open text

**[Your Name]**
University of Bamberg, [Department], Bamberg, Germany
E-mail: [your.email@uni-bamberg.de]

---

## Metadata

| Field | Value |
|---|---|
| Current code version | 1.0 |
| Permanent link to code/repository | https://github.com/[your-username]/rate-lm |
| Legal Software License | MIT |
| Computing platform / Operating system | Platform-independent (modern web browser) |
| Programming language | HTML5, CSS3, JavaScript (ES6+) |
| Software code languages, tools, and services used | Vanilla JavaScript; LM Studio; OpenAI-compatible REST API |
| Link to developer documentation/manual | https://github.com/[your-username]/rate-lm |
| Support email for questions | [your.email@uni-bamberg.de] |

---

## Abstract

NOTA (Natural-language Open Text Annotator) is a lightweight, browser-based tool that
enables social science researchers to classify large collections of open text using
locally-running large language models (LLMs). Operating entirely within the user's
browser and communicating with a local LM Studio server, NOTA processes no data
externally, making it suitable for research involving sensitive or confidential text data
such as interview transcripts or survey responses. Researchers define a coding scheme
through a natural-language system prompt, optionally test it on individual items, and then
apply it to an entire dataset uploaded as a CSV file. Results are returned as an enriched
CSV with the model's output appended as a new column. The tool requires no programming
knowledge and no cloud subscription, lowering the barrier to LLM-assisted qualitative
coding for researchers without computational backgrounds.

**Keywords:** qualitative coding; content analysis; large language models; local inference;
social science; text annotation; LM Studio

---

## 1. Motivation and Significance

Coding open text — assigning categories or labels to free-text responses such as
interview answers, social media comments, or open survey items — is a central task in
qualitative and mixed-methods social science research [CITE]. Traditionally, this work is
performed manually by trained coders following an established coding scheme, a process
that is time-consuming, resource-intensive, and subject to intercoder variability [CITE].

The emergence of large language models has opened new possibilities for automating or
semi-automating this process. Recent studies demonstrate that LLMs can perform
zero-shot and few-shot text classification with accuracy comparable to trained human
coders on a range of social science tasks [CITE, CITE]. However, the practical adoption
of these methods in empirical research is constrained by several barriers: (1) API-based
cloud services raise data privacy concerns when handling sensitive research data; (2)
existing Python-based pipelines require programming proficiency that many social science
researchers do not possess; and (3) the overhead of setting up and managing computational
environments discourages non-technical users.

NOTA addresses these barriers by combining the accessibility of a no-code web interface
with the privacy guarantees of fully local inference. The tool targets social scientists,
survey researchers, and content analysts who need to code moderate to large volumes of
open text — typically hundreds to thousands of items — without sending data to
third-party servers and without writing code. By integrating with LM Studio [CITE], a
widely used desktop application for running open-weight LLMs locally, NOTA makes
LLM-assisted coding practically accessible to a broad academic audience.

---

## 2. Software Description

### 2.1 Architecture

NOTA is implemented as a single self-contained HTML file that runs in any modern web
browser without installation. It communicates with a locally-running LM Studio instance
via its OpenAI-compatible REST API (`/v1/chat/completions`). No data leaves the user's
machine at any point: the browser sends requests directly to `localhost`, and all
processing occurs on the user's hardware. The tool has no server-side component, no
database, and no external dependencies beyond the fonts loaded at startup.

The interface is organised into three tabs — **Test**, **Batch**, and **Settings** —
each corresponding to a distinct phase of the coding workflow.

### 2.2 Coding Workflow

**Defining the coding scheme.** The researcher formulates a coding instruction as a
natural-language system prompt in the context panel. This prompt describes the categories
to be assigned and the decision rules to apply, equivalent to a codebook in traditional
content analysis. The panel includes a rough token counter to help researchers monitor
prompt length relative to the model's context window.

**Single-item testing.** Before batch processing, the **Test** tab allows the researcher
to submit individual text items and inspect the model's response in real time, with
streaming output. This allows iterative refinement of the system prompt without
committing to a full run.

**Batch processing.** In the **Batch** tab, the researcher uploads a CSV file (comma-,
semicolon-, or tab-separated formats are auto-detected), selects the column containing
the text to be coded, and specifies a name for the output column. Before the full run,
a preview mode applies the prompt to five randomly sampled items so the researcher can
verify output quality. The full batch then processes all rows and the results are
appended as a new column. Parallelism is configurable (1–16 concurrent requests),
enabling faster processing on capable hardware. Interrupted runs can be resumed from
the last completed row, avoiding duplicate processing.

**Output.** The enriched dataset is downloaded as a new CSV file retaining all original
columns plus the coded output column, ready for import into standard analysis software
(e.g., SPSS, Stata, R, or Excel).

### 2.3 Configuration

The **Settings** tab exposes the LM Studio server address (default:
`http://127.0.0.1:1234`), a model selector populated by querying the server's
`/v1/models` endpoint, and the concurrency setting. All user inputs — including the
system prompt, server address, and last-used column names — are persisted in the
browser's local storage, so settings are preserved across sessions.

---

## 3. Illustrative Example

To illustrate a typical use case, consider a researcher studying public attitudes toward
climate policy using responses to an open survey question ("What do you think is the
most important step governments should take on climate change?", *n* = 500). A
deductive coding scheme with five categories (e.g., *renewable energy*, *carbon pricing*,
*individual behaviour*, *international cooperation*, *other/unclear*) is translated into
a system prompt such as:

> *"You are a coding assistant. For each text, assign exactly one of the following
> categories: 'renewable energy', 'carbon pricing', 'individual behaviour',
> 'international cooperation', 'other'. Reply with the category label only."*

After testing the prompt on a handful of items and refining the instructions, the
researcher uploads the CSV, selects the response column, and initiates the batch run.
With a mid-range open-weight model (e.g., Llama 3.1 8B) running locally, processing
500 items takes approximately 5–15 minutes depending on hardware. The downloaded
output CSV is ready for statistical analysis, with the coded column alongside all
original survey variables.

---

## 4. Impact

NOTA lowers the practical threshold for LLM-assisted text coding in the social sciences
by removing the two most common barriers to adoption: programming requirements and data
privacy concerns. Compared to API-based solutions (e.g., direct use of the OpenAI or
Anthropic APIs), local inference ensures that confidential research data — including
data subject to institutional ethics approvals or GDPR obligations — never leaves the
researcher's environment. Compared to Python-based tools, the zero-installation,
browser-native design means the tool is immediately usable by any researcher who can
run LM Studio on their machine.

The tool is particularly well-suited to research contexts where (a) datasets are too
large for manual coding alone, (b) fully automated coding by a cloud service is ethically
or contractually prohibited, and (c) the researcher requires a reproducible, documented
coding procedure. The system prompt serves as a transparent record of the coding logic
that can be reported in a methods section or supplementary materials, supporting
methodological transparency and replication.

As open-weight models continue to improve, the quality of zero-shot coding achievable
locally is expected to approach that of larger proprietary models, making tools like NOTA
an increasingly viable alternative to cloud-based annotation pipelines.

---

## 5. Conclusions

NOTA provides social science researchers with a practical, privacy-preserving tool for
applying LLMs to qualitative text coding tasks. Its browser-based, no-code design
makes it accessible to a wide research audience, and its integration with local inference
via LM Studio ensures that sensitive data is never transmitted externally. The tool is
freely available under the MIT license at [GitHub URL].

---

## Conflict of Interest

The author declares no conflict of interest.

## Acknowledgements

[Optional: funding, colleagues, etc.]

## References

[CITE] Krippendorff, K. (2018). *Content Analysis: An Introduction to Its Methodology*
(4th ed.). Sage.

[CITE] Neuendorf, K. A. (2017). *The Content Analysis Guidebook* (2nd ed.). Sage.

[CITE] Gilardi, F., Alizadeh, M., & Kubli, M. (2023). ChatGPT outperforms crowd workers
for text-annotation tasks. *PNAS*, 120(30), e2305016120.

[CITE] Törnberg, P. (2023). ChatGPT-4 outperforms experts and crowd workers in
annotating political Twitter messages with zero-shot learning. *PLOS ONE*, 18(7),
e0285639.

[CITE] LM Studio. (2024). *LM Studio: Discover, download, and run local LLMs*.
https://lmstudio.ai
