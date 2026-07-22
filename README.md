# nlm-srs-query

An unofficial Claude Code skill for querying and maintaining Vietnamese software requirements specification (SRS) sources in Google NotebookLM through the `nlm` CLI. It discovers notebooks and sources at runtime, answers with verbatim citations, preserves follow-up context, and keeps write operations explicit.

## Prerequisites

- Claude Code with skill support.
- An installed `nlm` CLI compatible with version 0.7.7.
- A Google account with access to NotebookLM and the target notebooks.

Install `nlm` using the [upstream installation instructions](https://github.com/jacob-bd/notebooklm-mcp-cli#installation). This project intentionally does not duplicate an installation command that may change upstream.

Verify the CLI after installation:

```bash
nlm --version
```

## Install the skill

### User scope

Install for all Claude Code projects:

```bash
mkdir -p ~/.claude/skills/nlm-srs-query
cp SKILL.md ~/.claude/skills/nlm-srs-query/SKILL.md
```

### Project scope

From the target project root, install only for that project:

```bash
mkdir -p .claude/skills/nlm-srs-query
cp /path/to/nlm-srs-skill/SKILL.md .claude/skills/nlm-srs-query/SKILL.md
```

Restart Claude Code or begin a new session so the skill is rediscovered.

## Authenticate

Check the current session first:

```bash
nlm login --check
```

If authentication is missing or expired, run:

```bash
nlm login
```

Complete Google sign-in in the browser opened by `nlm`. Do not paste cookies, tokens, or browser profile data into prompts, repositories, logs, or issue reports.

## Discover notebooks and sources

Always discover current IDs instead of saving or hardcoding them:

```bash
nlm notebook list --json
nlm source list <NOTEBOOK_ID> --json
nlm source describe <SOURCE_ID> --json
```

Match the notebook title requested by the user. If multiple notebooks or sources remain plausible, ask the user to choose by title before querying or changing data.

## Query with citations

Query the selected notebook in structured form:

```bash
nlm notebook query <NOTEBOOK_ID> "<QUESTION_IN_VIETNAMESE>" --json
```

Return a concise interpretation plus each relevant `references[].cited_text` verbatim. Keep quoted evidence separate from interpretation. If no citation supports a claim, say so rather than presenting an implementation inference as an SRS requirement.

### Follow-up questions

Reuse the returned conversation ID to preserve context:

```bash
nlm notebook query <NOTEBOOK_ID> "<FOLLOW_UP_QUESTION>" \
  --conversation-id <CONVERSATION_ID> --json
```

Use follow-ups only to fill material gaps such as missing conditions, exceptions, roles, or statuses. Stop when the requirement is supported, unsupported, or genuinely ambiguous.

### Restrict source scope

Limit evidence to selected SRS sources:

```bash
nlm notebook query <NOTEBOOK_ID> "<QUESTION_IN_VIETNAMESE>" \
  --source-ids <SOURCE_ID_1,SOURCE_ID_2> --json
```

Keep the same `--source-ids` scope on follow-up queries.

## Add sources

Add a source only when the user requests it. Wait for ingestion before querying:

```bash
nlm source add <NOTEBOOK_ID> --url "<URL>" --wait
nlm source add <NOTEBOOK_ID> --file "<PATH>" --wait
nlm source add <NOTEBOOK_ID> --text "<TEXT>" --title "<TITLE>" --wait
```

Do not invent source titles or metadata.

## Sync Drive sources

Inspect freshness first:

```bash
nlm source stale <NOTEBOOK_ID> --json
```

Show the stale sources and obtain explicit user confirmation before syncing:

```bash
nlm source sync <NOTEBOOK_ID> --source-ids <SOURCE_IDS> --confirm
```

Sync changes NotebookLM data. Never add `--confirm` before the user approves the exact sources.

## Safety

- Discover notebook, source, and conversation IDs for every task; never hardcode or publish them.
- Never store authentication cookies, tokens, `.env` files, browser profiles, or personal notebook data in this repository.
- Do not use browser automation, NotebookLM MCP tools, or custom scripts; the skill is intentionally limited to the installed `nlm` CLI.
- Require explicit confirmation before sync, deletion, sharing changes, or any other persistent/destructive operation.
- Do not run interactive `nlm chat start` during agent execution.
- Use local source-code tools for implementation questions; use NotebookLM for documented requirements and citations.

## Troubleshooting

| Problem | Action |
|---|---|
| `nlm` is not found | Follow the upstream installation link, then rerun `nlm --version`. The skill does not install software automatically. |
| Authentication is absent or expired | Run `nlm login`, complete sign-in, then rerun `nlm login --check`. |
| Notebook is not found | Rerun `nlm notebook list --json`; select by current title. |
| Source is not found | Rerun `nlm source list <NOTEBOOK_ID> --json`; do not reuse an old ID. |
| Source is still processing | Wait and retry; use `--wait` for future additions. |
| Query times out | Retry once with a larger `--timeout <SECONDS>`. |
| Rate limit or server error persists | Stop and report the exact error; do not busy-loop. |
| Sources conflict or lack evidence | State the conflict or gap and request the missing source or clarification. |
| Skill does not appear | Confirm the path ends in `nlm-srs-query/SKILL.md`, then restart Claude Code or begin a new session. |

## Update

Replace the installed copy with the latest repository version.

User scope:

```bash
cp SKILL.md ~/.claude/skills/nlm-srs-query/SKILL.md
```

Project scope:

```bash
cp /path/to/nlm-srs-skill/SKILL.md .claude/skills/nlm-srs-query/SKILL.md
```

Review upstream changes before replacement if the installed copy has local edits.

## Uninstall

User scope:

```bash
rm -rf ~/.claude/skills/nlm-srs-query
```

Project scope:

```bash
rm -rf .claude/skills/nlm-srs-query
```

These commands remove only the skill files. They do not uninstall `nlm` or modify NotebookLM data.

## Compatibility

Tested against `nlm` 0.7.7. Later CLI versions may change commands, flags, or JSON fields; compare the installed CLI help and the [upstream project](https://github.com/jacob-bd/notebooklm-mcp-cli) before use.

## Disclaimer

This project is unofficial and is not affiliated with, endorsed by, or sponsored by Google, NotebookLM, the `nlm` upstream project, Anthropic, or Claude Code. Google NotebookLM is a Google product; Claude Code is an Anthropic product. Use is subject to the applicable service terms and organizational policies.

## License

MIT. See [LICENSE](LICENSE).
