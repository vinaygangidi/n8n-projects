# n8n-projects

n8n Cloud workflows for cash reconciliation and loan underwriting analysis; Mistral OCR, OpenAI, OneDrive.

![Type](https://img.shields.io/badge/type-n8n%20workflows-orange?style=flat-square)
![Last Commit](https://img.shields.io/github/last-commit/vinaygangidi/n8n-projects?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

## What This Does

Two exported n8n Cloud workflow definitions, each a document-processing pipeline that pulls
files from OneDrive, extracts text with Mistral OCR, and routes the result through an
OpenAI-backed agent.

These are JSON exports, not running services. Import them into an n8n instance and
reconnect the credentials to use them.

## How It Works

### `project-cahreconciliation/CashReconciliation.json`

Reconciles a bank statement against invoice data.

1. **Manual trigger** — run on demand
2. **Get Bank Statement** (OneDrive) — fetch the statement file
3. **Extract text** (Mistral AI) — OCR the statement
4. **Extract the Data from Unstructured File** (OneDrive) — pull the second source document
5. **Get Invoice Data** (Microsoft Excel) — read invoice rows
6. **Code in JavaScript** — normalize the extracted text
7. **Process the Invoice Vs Bank Statement Data** (AI Agent + OpenAI Chat Model) — match
   transactions to invoices
8. **Get Transaction, Matches, Summary** (Code) — shape the final output

### `project-2-losunderwriteranalysis/UnderwritingLoanAnalysis.json`

Classifies and analyzes a folder of loan documents.

1. **Manual trigger**
2. **Search a folder** → **Get items in a folder** → **Download a file** (OneDrive)
3. **Loop Over Items** (`splitInBatches`) — process each document
4. **Extract text** (Mistral AI) — OCR each file
5. **Classify the File** (Code) — determine document type
6. **Information Extractor** (+ OpenAI Chat Model) — pull structured fields
7. **Combine the Data** (Code) — merge per-document results
8. **AI Agent** (+ OpenAI Chat Model) — produce the underwriting analysis

Both workflows carry sticky notes in the canvas with inline documentation.

## Quickstart

1. Open your n8n instance (Cloud or self-hosted).
2. Choose **Workflows → Import from File** and select one of the JSON files.
3. Reconnect credentials on each node that needs them:
   - Microsoft OneDrive (OAuth2)
   - Microsoft Excel (OAuth2) — `CashReconciliation` only
   - Mistral AI (API key)
   - OpenAI (API key)
4. Update the OneDrive folder and file references to point at your own documents.
5. Click **Execute workflow**.

## Configuration

Credentials are held in n8n's credential store, not in these files. No environment variables
are involved.

| Credential | Used by | Purpose |
|---|---|---|
| Microsoft OneDrive OAuth2 | Both workflows | Locate and download source documents |
| Microsoft Excel OAuth2 | `CashReconciliation` | Read invoice rows |
| Mistral AI API key | Both workflows | OCR / text extraction |
| OpenAI API key | Both workflows | Agent reasoning and information extraction |

File and folder paths are hardcoded inside the OneDrive nodes and must be edited after
import.

## Limitations

- **Not runnable as-is.** These are workflow exports. Without an n8n instance and
  reconnected credentials they do nothing.
- **Credentials are stripped from the exports.** Node credential references point at IDs
  from the original n8n workspace and will not resolve in yours; each must be reselected by
  hand.
- **Paths are hardcoded.** OneDrive folder and file references point at the original
  author's storage. They will fail until edited.
- **Manual trigger only.** Neither workflow has a schedule, webhook, or queue trigger, so
  neither runs unattended.
- **No error handling.** No error branches, retries, or failure notifications. A failed OCR
  or API call halts the run without alerting.
- **Document contents go to third parties.** Every processed file is sent to Mistral for OCR
  and to OpenAI for analysis. Do not run these against confidential financial documents
  without checking your agreements.
- **No n8n version pinned.** These were exported from a particular n8n version; node
  parameters can shift between releases, and an import into a much newer instance may need
  adjusting.
- **Reconciliation output is unverified model output.** The matching in
  `CashReconciliation` is produced by an LLM with no deterministic arithmetic check.
- **Directory name has a typo.** `project-cahreconciliation` should read
  `project-cashreconciliation`. Left as-is to avoid breaking existing links.

## License

MIT — see [LICENSE](LICENSE).
