---
name: synalinks-memory-cli
description: >
  Use the Synalinks Memory CLI to interact with the Synalinks Memory knowledge layer
  for AI agents. This skill covers uploading data, querying predicates (tables,
  concepts, rules), asking natural language questions, searching, and exporting data.
  Trigger this skill whenever the user mentions Synalinks Memory, wants to add data to
  their knowledge base, query tables/concepts/rules, ask questions over their data
  using the Synalinks agent, export predicate data, or manage their Synalinks Memory
  instance from the terminal. Also trigger when you see references to the `synalinks`
  CLI command, Synalinks predicates, or the SYNALINKS_API_KEY environment variable.
---

# Synalinks Memory CLI

Synalinks Memory is the knowledge and context layer for AI agents. It uses **logical
rules** to derive knowledge from raw data — every claim traces back to evidence,
eliminating hallucinations. The CLI (`synalinks-memory-cli`) lets you interact with
Synalinks Memory directly from the terminal.

The key abstraction in Synalinks Memory is the **predicate**. There are three kinds:

- **Tables** — raw data uploaded from files or databases
- **Concepts** — derived knowledge the system learns from your questions
- **Rules** — logical rules learned from your questions based on concepts and tables

When users ask natural language questions, the system learns new concepts and rules
automatically, building a growing knowledge base over time.

## Setup

### Installation

```bash
pip install synalinks-memory-cli
```

Or with uv:

```bash
uv add synalinks-memory-cli
```

Or run without installing via uvx:

```bash
uvx synalinks-memory-cli list
```

### Authentication

The CLI requires a Synalinks API key. Set it as an environment variable:

```bash
export SYNALINKS_API_KEY="synalinks_..."
```

Or pass it per-command:

```bash
synalinks-memory-cli --api-key "synalinks_..." list
```

You can also override the API base URL with `--base-url` if connecting to a
custom instance.

To get your API key, you need a `starter` or `pro` plan at [https://app.synalinks.com](https://app.synalinks.com)

## Commands

### Ask a natural language question

The default behavior — no subcommand needed. Just type your question:

```bash
synalinks-memory-cli "What were the top 5 products by revenue last month?"
synalinks-memory-cli How are sales doing this quarter
```

This streams the question to Synalinks knowledge engineer agent, which reasons 
over your data using logical rules and returns an answer rendered as Markdown. 
The system learns new concepts and rules as it answers, so each question makes
the knowledge base smarter.

Quotes are optional — multi-word questions work with or without them.

### Add data

Upload a CSV or Parquet file as a new table:

```bash
synalinks-memory-cli add data/sales.csv
synalinks-memory-cli add data/events.parquet --name Events --description "Event log" --overwrite
```

Options:
- `--name` — CamelCase predicate name (derived from filename if omitted)
- `--description` — human-readable description for the table
- `--overwrite` — replace an existing table with the same name

The command prints the predicate name, column names, and row count on success.

### List predicates

See all tables, concepts, and rules in your knowledge base:

```bash
synalinks-memory-cli list
```

Displays a rich table with the type, name, and description of each predicate.

### Execute (query) a predicate

Fetch rows from any table, concept, or rule:

```bash
synalinks-memory-cli execute Users
synalinks-memory-cli execute Users --limit 50 --offset 10
```

Options:
- `--limit`, `-n` — max rows to return (default 20)
- `--offset` — row offset for pagination (default 0)

#### Export to file

Add `--format` to save data instead of printing it:

```bash
synalinks-memory-cli execute Users --format csv
synalinks-memory-cli execute Users --format parquet -o users.parquet
synalinks-memory-cli execute Users -f json --limit 500
```

Options:
- `--format`, `-f` — one of `json`, `csv`, `parquet`
- `--output`, `-o` — output file path (defaults to `<predicate>.<format>`)

### Search

Fuzzy keyword search within a predicate:

```bash
synalinks-memory-cli search Users "alice"
synalinks-memory-cli search Products "wireless headphones" --limit 10
```

Options:
- `--limit`, `-n` — max rows (default 20)
- `--offset` — pagination offset (default 0)

## Common Workflows

### Workflow 1: Upload data and explore it

```bash
# Upload a dataset
synalinks-memory-cli add data/customers.csv --name Customers --description "Customer records"

# See what predicates exist
synalinks-memory-cli list

# Browse the data
synalinks-memory-cli execute Customers --limit 10

# Search for specific records
synalinks-memory-cli search Customers "acme corp"
```

### Workflow 2: Ask questions and build knowledge

```bash
# Ask questions — the system learns concepts and rules automatically
synalinks-memory-cli "Which customers have the highest lifetime value?"
synalinks-memory-cli "What's the average order size by region?"

# List predicates to see newly learned concepts and rules
synalinks-memory-cli list

# Query a derived concept directly
synalinks-memory-cli execute HighValueCustomers
```

### Workflow 3: Export data for analysis

```bash
# Export to CSV for spreadsheet use
synalinks-memory-cli execute Orders --format csv -o orders_export.csv

# Export to Parquet for data pipelines
synalinks-memory-cli execute SalesMetrics --format parquet -o metrics.parquet

# Export as JSON
synalinks-memory-cli execute Products -f json --limit 1000 -o products.json
```

## Error Handling

All errors print to stderr with a `[red]Error:[/red]` prefix. Common issues:

- **Missing API key** — set `SYNALINKS_API_KEY` or pass `--api-key`
- **Unknown predicate** — check the name with `synalinks list`
- **File not found** — verify the path passed to `synalinks add`
- **`--output` without `--format`** — you must specify a format when using `-o`

## Technical Details

For deeper reference on the CLI internals, Python SDK types, and the streaming
ask protocol, see [synalinks-memory-cli](https://github.com/SynaLinks/synalinks-memory-cli).