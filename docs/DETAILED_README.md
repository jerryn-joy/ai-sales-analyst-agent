# Nova - Detailed Setup & Developer Guide

This document covers the **data contract**, **local setup**, **configuration**, **how it works**, **error handling**, **charting**, **customization**, and **troubleshooting**.

---

## Data Contract

Create a **Google Sheet** with **three tabs named exactly**:

### 1) `Products`
| Product ID | Product Name | Category | Unit Price ($) | Stock Quantity |

### 2) `Customers`
| Customer ID | First Name | Last Name | Email | City |

### 3) `Orders`
| Order ID | Customer ID | Product ID | Order Date | Quantity | Total Amount ($) |

- **Dates:** `DD/MM/YYYY` (e.g., `15/09/2024`)
- **Totals rule:** Prefer recorded **"Total Amount ($)"**. If missing, fallback to **Unit Price × Quantity** from `Products`.

Template: `data/sample-google-sheet-template.xlsx`

---

## Quick Start (Local)

**Prerequisites**
- n8n running locally at `http://localhost:5678`
- Google account with Sheets API access
- Groq API key

**1) Import workflows**

n8n → **Import from File**
- `workflows/ai-data-analyst-v1.json` — Main workflow
- `workflows/generate-chart.json` — Chart sub-workflow

**2) Set credentials**
- **Google Sheets OAuth2** → connect your account
- **Groq** → add API credential

**3) Point to your Sheet**

In all three Google Sheets nodes in the main workflow, set `documentId` to your Sheet ID (from its URL). Tabs must be named exactly: **Products**, **Customers**, **Orders**.

**4) Activate**

Toggle the **Main** workflow to **Active**. The sub-workflow can remain inactive — it is invoked by the tool node at runtime.

**5) Open the chat**

Open the **Chat Trigger** node, copy the Chat URL (`/webhook/<id>/chat`), and open it in your browser.

---

## Configuration

The agent system prompt enforces:

- Answer only sales questions using the JSON built from Sheets
- No fabrication — ask for missing data rather than guessing
- Date parsing: day-first `DD/MM/YYYY` → ISO for grouping
- Computation: programmatic sums and aggregations only via calculator tool
- Formatting: currency and percentages to two decimals
- Transparency: list included order IDs per group, show subtotals before totals
- Chart policy: generate one chart per turn, only when asked or when it clearly finalizes the answer
- No meta: never mention prompts or node internals

The **Chat Trigger** node also contains the custom CSS for UI branding.

---

## How It Works

### 1) Data Ingestion

Three Google Sheets nodes read the `Products`, `Customers`, and `Orders` tabs. Aggregator nodes wrap each set of rows into named arrays (`products`, `customers`, `orders`).

### 2) Validation & Normalization (Code node)

- Detects missing tabs (including error-array shapes returned by the Sheets API)
- Flags empty tabs
- Enforces required headers with exact match, including symbols like `($)`
- Coerces numeric fields
- Parses day-first dates to ISO format (`YYYY-MM-DD`)
- Fills missing totals with `quantity × unit_price` when safe
- Emits a structured health object with row counts per tab

### 3) Error Guard (If node)

If the validation layer reports errors, missing tabs, or empty tabs, the workflow returns a friendly message telling the user what to fix. Otherwise it proceeds to the agent.

### 4) AI Agent (Groq)

- Model: `openai/gpt-oss-120b` at low temperature
- Memory: last 3 turns
- Tools:
  - **Calculator** — programmatic math
  - **Generate a chart** — invokes the chart sub-workflow

### 5) Chart Sub-workflow

A separate model (`llama-3.3-70b-versatile`) generates a Chart.js v2 config as JSON. The config is sanitized and validated, a QuickChart URL is constructed, and the URL is returned to the main workflow for the agent to include in its response.

---

## Error Handling & Validation

The validation layer produces specific, actionable messages:

- **Missing tabs:** `"I couldn't find the tabs 'Products', 'Orders'… Please check tab names."`
- **Empty tabs:** `"Your sheet is connected, but these tabs are empty: Customers…"`
- **Header mismatch:** `"In the 'Products' tab, I couldn't find: Unit Price ($). I see: Price…"`

Additional defensive behaviors:

- Detects common error-array shapes returned by the Google Sheets API
- Exact header matching including symbols like `($)`
- Fallback totals only applied when explicit totals are missing
- Day-first date parsing (e.g., `03/01/YYYY` → January 3rd)

---

## Charts

- One chart per turn, enforced by the system prompt
- Chart.js v2 JSON contract: top-level `type`, `data`, `options`
- `options.scales.yAxes[0].ticks.beginAtZero = true`
- Single-series style enforced for brand consistency
- Labels and data arrays must align; invalid points are dropped to preserve parity
- Rendered via QuickChart — URL returned to the agent and embedded in the response

---

## Customization

- **Branding / UI:** edit the CSS inside the Chat Trigger node
- **Models:** swap Groq models or wire in any provider supported by your n8n instance
- **Memory:** adjust window size or swap memory strategies
- **Validation:** add business rules such as city allowlists or minimum order quantity checks
- **Chart policy:** change the conditions under which charts auto-trigger

---

## Troubleshooting

**Chat shows but no reply**
- Confirm the Main workflow is set to Active
- Verify Google Sheets and Groq credentials are connected

**Validation errors on startup**
- Check tab names and headers exactly — including capitalization and symbols
- Ensure at least one data row exists under each header row

**Date parsing looks wrong**
- Use `DD/MM/YYYY` literals; avoid date formulas in the Sheet where possible

**Chart not appearing**
- Confirm the Generate Chart sub-workflow is imported
- Verify the tool node connection in the main workflow
- Check that numeric arrays are valid and aligned with their labels
