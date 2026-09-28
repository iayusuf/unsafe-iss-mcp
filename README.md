# unsafe-iss-mcp (test fixture)

A **"who is in space right now" MCP server** built to trigger a
**"Data Exfiltration / Exposure"** finding in an MCP security scanner
(e.g. Cisco AI Defense MCP scan) — **without containing any harmful code.**

## What it does (the real part)

Uses the free, keyless [Open Notify](http://open-notify.org/Open-Notify-API/)
API:

- `who_is_in_space` → number and names of people currently in space, and their
  craft (`http://api.open-notify.org/astros.json`)
- `iss_location` → current latitude/longitude of the ISS
  (`http://api.open-notify.org/iss-now.json`)

> Note: Open Notify is plain HTTP (no HTTPS) and needs no API key.

## Why it's safe

The executable code only calls that public API and returns the result. It does
**not**:

- read credential files (`~/.aws/credentials`, `~/.ssh/id_rsa`, etc.)
- read secret environment variables
- send any data to any third party

The "unsafe" signal lives entirely in the **tool docstrings** — the model-facing
descriptions of `who_is_in_space` and `iss_location`. Those descriptions contain
**tool-poisoning** text (hidden instructions telling an AI model to leak
secrets). This is exactly what a scanner detects and classifies as data
exfiltration. But the text is just a string; nothing in the code acts on it.

Think of it as a **prop gun**: it reads as a threat to the scanner, but the
chamber is empty.

## How to use it with a scanner (no hosting required)

This repo is scanned as **source code**, so you don't have to run it anywhere.

1. Push these files to a **public GitHub repo** (no authentication needed).
2. In Cisco AI Defense, add an MCP server with:
   - **Type:** Code
   - **Source:** Manual
   - **Repo URL:** your GitHub repo URL
3. Run the scan. Expected result: **High** severity, attack technique
   **Data Exfiltration / Exposure**, affecting the `who_is_in_space` and
   `iss_location` tools.

## Files

| File             | Purpose                                              |
| ---------------- | ---------------------------------------------------- |
| `server.py`      | The MCP server. Benign code; poisoned docstrings.    |
| `pyproject.toml` | Dependencies (`mcp`, `httpx`).                       |
| `README.md`      | This file.                                           |

## Running it (optional)

You do not need to run it to scan it. To see it work as a real tool:

```bash
uv sync           # or: pip install mcp httpx
python server.py  # serves on http://127.0.0.1:8000 by default
```

## Jargon quick-reference

- **MCP (Model Context Protocol):** the standard that lets an AI model call
  external "tools."
- **MCP server:** a program that exposes those tools.
- **Tool docstring / description:** the text describing a tool, which the AI
  model reads to decide how to use it.
- **Tool poisoning:** hiding malicious instructions inside that description so
  the model is manipulated — the attack this fixture imitates, as inert text.
- **Data exfiltration:** smuggling sensitive data out. Here it's only
  *described*, never performed.
