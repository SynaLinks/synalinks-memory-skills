---
name: synalinks-memory-cli
description: >
  Use the Synalinks Memory CLI to interact with the Synalinks Memory knowledge layer
  for AI agents. This skill covers uploading data, querying predicates (tables,
  concepts, rules), chatting with the agent, searching, and exporting data.
  Trigger this skill whenever the user mentions Synalinks Memory, wants to add data to
  their knowledge base, query tables/concepts/rules, chat with the agent over their data,
  export predicate data, or manage their Synalinks Memory instance from the terminal.
  Also trigger when you see references to the `synalinks` CLI command, Synalinks
  predicates, or the SYNALINKS_API_KEY environment variable.
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

A **Synalinks API key** is required to authenticate with your knowledge base.

When you create a knowledge base on [app.synalinks.com](https://app.synalinks.com), a **default API key** is generated automatically with read/write access and no predicate restrictions — you can use it right away.

To create a key with granular access, go to **Profile icon** (in the header) > **API Keys** > **Create API Key**.

Set it as an environment variable:

```bash
export SYNALINKS_API_KEY="synalinks_..."
```

Or pass it per-command:

```bash
synalinks-memory-cli --api-key "synalinks_..." list
```

You can also override the API base URL with `--base-url` if connecting to a
custom instance.

## Commands

### Chat with the agent

**IMPORTANT: Only use the chat command when no existing concepts or rules can answer
the user's need.** Always check first with `list` and `execute` — if a relevant
concept or rule already exists, query it directly instead. The chat command triggers
the knowledge engineer agent, which is expensive and slow. Use it only to teach the
system something new (creating new concepts/rules) or when the existing predicates
genuinely cannot answer the question.

The default behavior — no subcommand needed. Just type your question:

```bash
synalinks-memory-cli "What were the top 5 products by revenue last month?"
synalinks-memory-cli How are sales doing this quarter
```

This streams the question to the Synalinks knowledge engineer agent, which reasons
over your data using logical rules and returns an answer rendered as Markdown.
The system learns new concepts and rules as it answers, so each question makes
the knowledge base smarter.

Quotes are optional — multi-word questions work with or without them.

Chat is **multi-turn** — conversation history is persisted to disk
(`~/.synalinks/chat_history.json`) so follow-up questions have full context
across CLI invocations. The MCP server also preserves history in-memory across
calls. To reset the conversation and start fresh, use `/clear`:

```bash
synalinks-memory-cli /clear
```

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

### Insert a row

Insert a single row into a table. The argument is a JSON object mapping column
names to values:

```bash
synalinks-memory-cli insert Users '{"name": "Alice", "email": "alice@example.com"}'
```

### Update rows

Update rows in a table that match a filter. Takes two JSON arguments: a filter
(column→value conditions, ANDed) and the values to set:

```bash
synalinks-memory-cli update Users '{"name": "Alice"}' '{"email": "alice@new.com"}'
```

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

#### Export data

Add `--format` to output formatted data. Without `-o` it prints to stdout (pipeable);
with `-o` it saves to a file:

```bash
synalinks-memory-cli execute Users --format csv
synalinks-memory-cli execute Users --format parquet -o users.parquet
synalinks-memory-cli execute Users -f json --limit 500
```

Options:
- `--format`, `-f` — one of `json`, `csv`, `parquet` (prints to stdout if no `-o`)
- `--output`, `-o` — save to file instead of stdout

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

### Workflow 2: Chat and build knowledge

**Always check existing predicates first** — only use chat when no concept or rule
already answers the question:

```bash
# 1. Check what already exists
synalinks-memory-cli list

# 2. If a relevant concept/rule exists, query it directly (fast, free)
synalinks-memory-cli execute HighValueCustomers

# 3. Only chat when the knowledge doesn't exist yet (creates new concepts/rules)
synalinks-memory-cli "Which customers have the highest lifetime value?"
synalinks-memory-cli "What's the average order size by region?"

# 4. Newly learned concepts and rules are now available for direct queries
synalinks-memory-cli list
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
- **`--output` without `--format`** — `-o` requires `--format` to be set

## Technical Details

For deeper reference on the CLI internals, Python SDK types, and the streaming
chat protocol, see [synalinks-memory-cli](https://github.com/SynaLinks/synalinks-memory-cli).