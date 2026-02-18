# Claude Code Skills for Synalinks Memory

---

## What is Synalinks Memory?

[Synalinks Memory](https://app.synalinks.com) is the knowledge and context layer for AI agents. It uses **logical rules** to derive knowledge from raw data, every claim traces back to evidence, eliminating hallucinations. The key abstraction is the **predicate**:

- **Tables** — raw data uploaded from files or databases
- **Concepts** — derived knowledge the system learns from your questions
- **Rules** — logical rules learned automatically based on concepts and tables

When you ask natural language questions, the system learns new concepts and rules, building a growing knowledge base over time.

## Included Skills

### `synalinks-memory-cli`

Teaches Claude Code how to use the [Synalinks Memory CLI](https://github.com/SynaLinks/synalinks-memory-cli) to interact with your knowledge base from the terminal. Covers:

- **Uploading data** — add CSV/Parquet files as tables
- **Querying predicates** — fetch rows from tables, concepts, and rules
- **Asking questions** — stream natural language questions to the knowledge engineer agent
- **Searching** — fuzzy keyword search within any predicate
- **Exporting** — save predicate data as CSV, Parquet, or JSON

The skill activates automatically when you mention Synalinks Memory, the `synalinks-memory-cli` command, predicates, or the `SYNALINKS_API_KEY` environment variable.

## Install

### Claude Code

Copy the skill into your Claude Code skills directory:

```shell
git clone https://github.com/SynaLinks/synalinks-memory-skills.git
cd synalinks-memory-skills
mkdir -p ~/.claude/skills/
cp -r synalinks-memory-cli ~/.claude/skills/
```

Verify the skill is in place:

```shell
head ~/.claude/skills/synalinks-memory-cli/SKILL.md
```

Start Claude Code — the skill loads automatically and activates when relevant:

```shell
claude
```

## License

Licensed under Apache 2.0. See the [LICENSE](LICENSE) file for full details.