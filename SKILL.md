---
name: autocontext
description: Manage Autocontext contexts and tasks. Delegate knowledge retrieval, data ingestion, and refinement workflows to the Autocontext platform. Recognizes requests like "create a context", "ingest files", "run a retrieval task", or "list my contexts".
license: SSPL-1.0
compatibility: Requires the autocontext binary in PATH (Linux x86_64) and network access to an AUTOCONTEXT API instance.
metadata:
  author: pinecone-io
  version: "0.1.0"
allowed-tools: Bash(autocontext:*)
---

# AUTOCONTEXT CLI Skill

**AUTOCONTEXT is a context compute platform** that orchestrates Docker-backed workflows for knowledge retrieval, data ingestion, and entity extraction. It connects contexts to Pinecone vector databases and uses Gemini for inference.

This skill should be activated whenever the user needs to:
- Manage autocontexts (create, select, list)
- Ingest documents or files into a context
- Run retrieval tasks against a context
- Run refinement tasks
- Manage tasks

## Key Concepts

- **Autocontext**: A container for your data and configurations. Tasks cannot exist without an autocontext.
- **Workflow**: A specific operation type (`ingest`, `retrieve`, `refine`, etc.). Use `autocontext workflow list` to see available ones.
- **Task**: An execution instance of a workflow within a specific autocontext. Created via `autocontext task create`.

## Authentication

Before running commands, authenticate with a Pinecone API key:

```bash
autocontext login
# Or set PINECONE_API_KEY environment variable.
```

The CLI config is stored in `~/.autocontext/autocontext.json`.

## Autocontext Management

All tasks belong to an autocontext. You can select an active autocontext to avoid passing the autocontext ID to every command.

```bash
# Create a new autocontext
autocontext autocontext create --name "my-context"

# Select an active autocontext for the session
autocontext autocontext select <CONTEXT_ID>

# List your autocontexts
autocontext autocontext list
```

## Task Management

Tasks execute workflows against an autocontext.

### Workflows

- `ingest`: Reads files, chunks them, and upserts vectors to Pinecone.
- `retrieve`: Expands queries, searches Pinecone, and synthesizes answers.
- `refine`: An autonomous coding loop that iterates on ingest/retrieve scripts.

List all available workflows:

```bash
autocontext workflow list
```

### Creating Tasks

Create a task using the `autocontext task create` command. Always use `--json` when parsing output.

```bash
# Create a retrieval task
TASK_ID=$(autocontext task create --workflow retrieve --instruction "Find all documents about pricing" --json | jq -r '.id')

# Create an ingestion task
TASK_ID=$(autocontext task create --workflow ingest --instruction "Ingest the newly uploaded files" --json | jq -r '.id')
```

### Monitoring and Managing Tasks

```bash
# Get task details and output
autocontext task get --id <TASK_ID> --json

# List tasks
autocontext task list --json

# Cancel a running task
autocontext task cancel --id <TASK_ID>
```

## Common Patterns

### Run a Task and Wait for Output

```bash
# 1. Create the task
TASK_ID=$(autocontext task create --workflow retrieve --instruction "What is the auth token format?" --json | jq -r '.id')

# 2. Poll until completed
RESULT=$(autocontext task get --id "$TASK_ID" --json)
STATE=$(echo "$RESULT" | jq -r '.state')

if [ "$STATE" != "completed" ] && [ "$STATE" != "failed" ] && [ "$STATE" != "cancelled" ]; then
  sleep 5
  RESULT=$(autocontext task get --id "$TASK_ID" --json)
fi

# 3. Read output
echo "$RESULT" | jq -r '.output'
```

## Output Format

Always use `--json` for structured, parseable JSON output from the CLI. Without `--json`, the CLI formats the output into human-readable tables.
