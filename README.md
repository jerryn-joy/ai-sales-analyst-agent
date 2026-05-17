# Nova - AI Sales Analyst Agent (n8n + Groq)

A privacy-friendly AI chatbot that answers natural-language sales questions directly from your Google Sheet — no cloud data uploads, no third-party storage. Built with n8n workflows, Groq LLMs, and a clean branded chat UI.

> The core idea: show how free tools and open models can analyze your own data locally. The demo uses sales data (a common use case), but the architecture adapts to any dataset or domain.

<p align="center">
  <img src="./assets/demo.gif" alt="Chatbot Demo">
</p>

---

## What it does

- Natural-language Q&A over **Products**, **Customers**, and **Orders**
- Defensive data layer: header checks, missing/empty tab detection, day-first date parsing, numeric coercion
- Transparent analytics: lists **order IDs**, shows subtotals/totals, formats currency and percentages to two decimals
- Programmatic math only — uses a calculator tool, no hand-computed sums
- Composable: agent can invoke a **chart sub-workflow** that returns a rendered image URL
- Short-term memory across the last few turns
- Clean, branded chat UI with custom CSS
- Local-first: runs on your machine; your data never leaves it

### Workflows

- **Main Workflow — `AI Sales Analyst Agent`:** handles the chat interface, Sheet ingestion, data validation, and agent reasoning
- **Sub-workflow — `Generate Chart`:** invoked by the agent when the user requests a chart or when a visual clearly finalizes the answer

---

## Architecture

<img src="./assets/architecture.png" width="700"/>

---

## Usage

Ask questions like:

- "Total revenue for September?"
- "Top 5 customers by spend"
- "Products low on stock (< 10)"
- "Revenue by category last 3 months"
- "Monthly revenue for Jan–Jun and the trend (chart)?"
- "Which category brings in the most money?"
- "Who are our top 3 customers and which cities buy the most?"

The agent will validate your sheet structure, compute metrics programmatically, list order IDs per segment, and generate a chart when asked or when it clearly finalizes the answer.

A ready-made template is included at `data/sample-sales-data-template.xlsx`

> For detailed setup, internals, and troubleshooting, see **[docs/DETAILED_README.md](docs/DETAILED_README.md)**.

---

## Repo Structure

```text
├─ workflows/
│  ├─ ai-data-analyst-chatbot.json    # Main n8n workflow (Chat, Sheets, Validate, Agent, Tools)
│  └─ generate-chart.json             # Sub-workflow (Chart.js v2 config → QuickChart)
├─ data/
│  └─ sample-sales-data-template.xlsx # Sheet template with required tabs/headers
├─ assets/
│  ├─ demo.mp4                        # Short demo video
│  ├─ demo.gif                        # Demo preview (for README)
│  ├─ architecture.png                # System architecture diagram
│  └─ chat-ui-screenshot.png          # Screenshot of the chat UI
├─ docs/
│  ├─ SETUP.md                        # Installation and setup guide
│  ├─ DETAILED_README.md              # In-depth developer documentation
│  └─ TROUBLESHOOTING.md              # Common issues and solutions
├─ .gitignore
├─ LICENSE
└─ README.md
```

---

## Roadmap

- Ingest once; store in a database (SQL or vector DB) and update on change — better for large datasets
- Extend memory beyond short-term conversation context
- Additional data source connectors

---

## What this demonstrates

- Agentic workflow design with clear separation of concerns (main workflow + composable sub-workflow)
- Robust data handling: pre-checks, exact header enforcement, helpful error messages
- Prompt engineering for programmatic math, transparent output, and chart discipline
- Local-first architecture as a practical privacy alternative to cloud AI tools
- Clean developer experience: clear setup, sample data, and branding hooks
