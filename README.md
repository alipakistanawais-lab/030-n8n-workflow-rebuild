# 030 Office Workflow AI Assistant

A clean, production-ready Telegram-based Microsoft 365 AI assistant for n8n. Approved company
users control Outlook, Calendar, OneDrive and Microsoft To Do through a Telegram bot, driven by an
AI agent that routes intents to modular Microsoft Graph sub-workflows.

> Built from scratch. The old exported workflows were used only as reference for credential
> names/IDs, Graph endpoints and node patterns — they are **not** imported or modified.

## What it does
- Access control (only approved Telegram chat IDs; everyone else gets `not available`)
- Outlook: read/unread/latest/search, numbered email memory and follow-ups, content retrieval
- Attachments: detect, list, send to Telegram, save to OneDrive
- Outlook **drafts only** (plain, reply, reply-all, with OneDrive attachment) — never sends
- Calendar: create, search/summarize, conflict & duplicate detection (Europe/Berlin)
- OneDrive: file/folder search, root listing, exact-folder listing, save reports
- Microsoft To Do: list and create tasks
- Contact name → email resolution (strict)
- German/English switching, vague-input handling, graceful video handling
- Clean Telegram replies (no n8n branding), long-reply splitting, full error handling

## Layout
```
workflows/   1 main workflow + 23 sub-workflows (importable n8n JSON)
docs/        SETUP_GUIDE, ARCHITECTURE, SAFETY_RULES, CREDENTIALS_TO_SELECT
tests/       COMPANY_TEST_CHECKLIST, EXPECTED_RESULTS, KNOWN_LIMITATIONS
```

## Start here
1. `docs/SETUP_GUIDE.md` — import order and configuration
2. `docs/CREDENTIALS_TO_SELECT.md` — which credentials to pick after import
3. `tests/COMPANY_TEST_CHECKLIST.md` — acceptance tests

Main workflow: `workflows/PROD_030_TELEGRAM_OFFICE_AI_ASSISTANT_CLEAN.json`

## Safety
Never sends email (drafts only), never deletes/moves email, never overwrites OneDrive files,
never invents IDs/emails/dates, and reports a verification status on every action. See
`docs/SAFETY_RULES.md`.
