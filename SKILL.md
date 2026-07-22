---
name: nlm-srs-query
description: Query and maintain Vietnamese SRS/spec sources in Google NotebookLM via the installed `nlm` CLI. Use when the user asks what an SRS, spec, requirement, document, NotebookLM notebook, or source says; needs verbatim evidence; or wants to list, add, inspect, or sync SRS sources. Always discover current notebook/source IDs instead of hardcoding them.
---

# nlm-srs-query

## Goal

Manage the minimal SRS lifecycle through `nlm`: authenticate, discover the correct notebook and sources, add or sync sources when requested, then answer requirements questions with auditable verbatim citations.

## Workflow

### 1. Preflight authentication

```bash
nlm --version
nlm login --check
```

If `nlm` is missing, stop and report it; do not install software automatically. If authentication is absent or expired, run `nlm login` and tell the user that a browser will open for Google sign-in.

### 2. Discover the notebook

```bash
nlm notebook list --json
```

Never reuse a remembered UUID. Match a user-supplied title case-insensitively. If no title was supplied and several notebooks are plausible, show their titles and ask the user to choose.

### 3. Discover or maintain SRS sources

```bash
nlm source list <NOTEBOOK_ID> --json
nlm source describe <SOURCE_ID> --json
nlm source content <SOURCE_ID> --json
```

Use `describe` to identify an ambiguous source. Use `content` only when raw source text is needed; prefer query citations for normal answers.

Add a source only when requested. Wait until ingestion completes before querying:

```bash
nlm source add <NOTEBOOK_ID> --url "<URL>" --wait
nlm source add <NOTEBOOK_ID> --file "<PATH>" --wait
nlm source add <NOTEBOOK_ID> --text "<TEXT>" --title "<TITLE>" --wait
```

Do not invent source titles or metadata. For Drive sources, inspect freshness first:

```bash
nlm source stale <NOTEBOOK_ID> --json
```

Sync changes NotebookLM data. Show the stale sources and get explicit user confirmation before running:

```bash
nlm source sync <NOTEBOOK_ID> --source-ids <SOURCE_IDS> --confirm
```

### 4. Query with structured output

```bash
nlm notebook query <NOTEBOOK_ID> "<QUESTION_IN_VIETNAMESE>" --json
```

Options:

- `--conversation-id <ID>`: preserve context for follow-up questions.
- `--source-ids <ID1,ID2>`: restrict evidence to selected SRS sources.
- `--timeout <SECONDS>`: raise the default 120 seconds for large documents.

### 5. Completion gate

Before answering the user:

1. Compare the result with the original question.
2. Check whether conditions, exceptions, roles, statuses, or evidence are missing.
3. If material information is missing, ask a focused follow-up using the returned `conversation_id` and the same source scope.
4. Stop when the requirement is supported, unsupported, or genuinely ambiguous. Do not loop merely to obtain a preferred answer.
5. Synthesize all relevant turns; do not return only the first response.

### 6. Return auditable evidence

Return:

1. A concise answer.
2. Each relevant `references[].cited_text` verbatim, labeled with its citation/source information when available.
3. A clear limitation when no citation supports the claim or sources conflict.

Never paraphrase inside a quoted citation. Keep interpretation separate. Do not present implementation inference as an SRS requirement.

## Error recovery

| Failure | Action |
|---|---|
| CLI missing | Report it; do not auto-install |
| Cookies/session expired | Run `nlm login` |
| Notebook not found | Run `nlm notebook list --json` again |
| Source not found | Run `nlm source list <NOTEBOOK_ID> --json` again |
| Source still processing | Wait, then retry; use `--wait` when adding future sources |
| Query timeout | Retry once with a larger `--timeout` |
| Rate limit or server error remains after CLI retries | Stop and report the exact error; do not busy-loop |
| Sources conflict or lack evidence | State the conflict/gap and request the missing source or clarification |

## Examples

### Verify an SRS requirement

```bash
nlm login --check
nlm notebook list --json
nlm source list <NOTEBOOK_ID> --json
nlm notebook query <NOTEBOOK_ID> \
  "Yêu cầu clone Nhóm/Loài từ hệ thống là gì? Trích nguyên văn căn cứ và nêu ngoại lệ." \
  --source-ids <SRS_SOURCE_IDS> --json
nlm notebook query <NOTEBOOK_ID> \
  "Bổ sung điều kiện, vai trò hoặc trạng thái còn thiếu; chỉ dùng các nguồn đã chọn." \
  --source-ids <SRS_SOURCE_IDS> --conversation-id <CONVERSATION_ID> --json
```

### Add or refresh an SRS source

```bash
nlm login --check
nlm notebook list --json
nlm source add <NOTEBOOK_ID> --file "/path/to/srs.pdf" --wait
nlm source list <NOTEBOOK_ID> --json

# Drive source: inspect first; sync only after explicit confirmation
nlm source stale <NOTEBOOK_ID> --json
nlm source sync <NOTEBOOK_ID> --source-ids <SOURCE_IDS> --confirm
```

## Constraints

- Use only the installed `nlm` CLI; do not use NotebookLM MCP tools, Patchright, or Python/browser automation scripts.
- Do not run `nlm chat start`; its interactive REPL is unsuitable for agent execution.
- Do not hardcode notebook, source, conversation, or artifact IDs.
- Never delete a notebook/source or change sharing without explicit confirmation after showing the exact target and permanent impact.
- Treat source sync as a write: require confirmation before adding `--confirm`.
- Keep this skill scoped to SRS lifecycle. Do not use research, studio artifacts, podcasts, quizzes, slides, downloads, exports, batch, or cross-notebook workflows.
- Use local source/code tools first for implementation questions; use NotebookLM when the user needs the documented requirement or citation.
