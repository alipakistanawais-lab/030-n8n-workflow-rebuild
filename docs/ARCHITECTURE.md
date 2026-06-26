# Architecture — 030 Office Workflow AI Assistant

## Overview

A Telegram-based Microsoft 365 assistant built entirely in n8n. A single main workflow
hosts the Telegram entry point, access control, the AI agent, reply formatting and error
handling. The AI agent routes work to 23 modular sub-workflows (the "Microsoft 365 Tool
Layer"). Every Microsoft action goes through Microsoft Graph using one generic OAuth2
credential.

## Main workflow data flow

```
Telegram Trigger
  -> Telegram Input Mapper            (Code: normalize update, compute access)
  -> Access Control Check             (IF: isAllowed?)
       false -> Access Denied Reply   (Telegram: "not available")
       true  -> AI Agent Command Center
                   |  uses OpenAI Chat Model   (ai_languageModel)
                   |  uses Window Buffer Memory (ai_memory, key = telegramChatId)
                   |  uses 23 Tool - * (ai_tool -> sub-workflows)
                   main(0) -> Clean Final Reply Formatter
                                -> Telegram Long Reply Splitter
                                   -> Telegram Reply
                                      -> Execution Logger
                   error(1) -> Error Handler
                                -> Telegram Error Reply
                                   -> Execution Logger
```

### Node responsibilities

- **Telegram Trigger** — receives Telegram `message` updates.
- **Telegram Input Mapper** — produces a normalized item with: `platform`, `telegramChatId`,
  `telegramUserId`, `telegramUserName`, `telegramMessageId`, `chatInput`, `messageType`,
  `languageHint`, `originalPayload`, plus `isAllowed` (access decision).
- **Access Control Check** — IF on `isAllowed`. Unauthorized users get exactly `not available`.
- **AI Agent Command Center** — receives exactly `={{ $json.chatInput }}`. Routes intents to tools.
- **OpenAI Chat Model** — `gpt-4.1-mini` (change in the node if desired).
- **Window Buffer Memory** — per-chat conversation memory keyed on `telegramChatId`.
- **Clean Final Reply Formatter** — strips leading `=`, `undefined`/`null`, debug JSON and any
  n8n branding/attribution; returns `finalReply`.
- **Telegram Long Reply Splitter** — splits long replies into ≤3900-char chunks, never cutting a
  numbered-list line in the middle.
- **Telegram Reply** — sends each chunk with attribution disabled (`appendAttribution: false`).
- **Error Handler / Telegram Error Reply** — clean, branded-free error message on agent failure.
- **Execution Logger** — compact per-reply log line.

## Tool layer (sub-workflows)

Each sub-workflow:
1. starts with an **Execute Workflow Trigger** with named string inputs,
2. normalizes inputs in a `00 Prepare` Code node,
3. calls Microsoft Graph via an HTTP Request node (generic OAuth2),
4. ends in a Code node that returns a **`response`** string plus a `verificationStatus`.

| # | Tool node (main) | Sub-workflow | Graph endpoint(s) |
|---|---|---|---|
| 1 | Tool - Email List And Memory | WF_EMAIL_LIST_CONTEXT | `GET /me/mailFolders/Inbox/messages` + static-data memory |
| 2 | Tool - Get Email Content | WF_GET_EMAIL_CONTENT_BY_MESSAGE_ID | `GET /me/messages/{id}` |
| 3 | Tool - List Email Attachments | WF_LIST_EMAIL_ATTACHMENTS | `GET /me/messages/{id}/attachments` |
| 4 | Tool - Send Attachment To Telegram | WF_ATTACHMENT_SEND_TO_TELEGRAM | `GET .../attachments/{id}/$value` + Telegram sendDocument |
| 5 | Tool - Save Email Attachments To OneDrive | WF_SAVE_EMAIL_ATTACHMENTS_TO_ONEDRIVE | `GET /me/messages/{id}?$expand=attachments` + `PUT /me/drive/root:/...:/content` |
| 6 | Tool - Save Today Unread Attachments | WF_SAVE_TODAY_UNREAD_ATTACHMENTS_TO_ONEDRIVE_ROOT | `GET /me/mailFolders/Inbox/messages?$filter=...&$expand=attachments` + `PUT .../content` |
| 7 | Tool - Create Outlook Draft | WF_CREATE_OUTLOOK_DRAFT_TEXT_ONLY | `POST /me/messages` |
| 8 | Tool - Create Reply Draft | WF_CREATE_OUTLOOK_REPLY_DRAFT | `POST /me/messages/{id}/createReply` |
| 9 | Tool - Create Reply All Draft | WF_CREATE_REPLY_ALL_DRAFT | `POST /me/messages/{id}/createReplyAll` |
| 10 | Tool - Create Draft With OneDrive Attachment | WF_CREATE_OUTLOOK_DRAFT_WITH_ONEDRIVE_ATTACHMENT | `GET drive file` + `POST /me/messages` + `POST .../attachments` |
| 11 | Tool - Create Calendar Event | WF_CREATE_CALENDAR_EVENT | `POST /me/events` |
| 12 | Tool - Search Calendar Events | WF_SEARCH_CALENDAR_EVENTS | `GET /me/calendarView` |
| 13 | Tool - Check Calendar Conflicts | WF_CHECK_CALENDAR_CONFLICTS | `GET /me/calendarView` |
| 14 | Tool - Search OneDrive Files | WF_SEARCH_ONEDRIVE_FILES | `GET /me/drive/root/search(q=...)` |
| 15 | Tool - Search OneDrive Folders | WF_SEARCH_ONEDRIVE_FOLDERS | `GET /me/drive/root/search(q=...)` |
| 16 | Tool - List OneDrive Root Files | WF_LIST_ONEDRIVE_ROOT_FILES | `GET /me/drive/root/children` |
| 17 | Tool - List Files In Exact Folder | WF_LIST_FILES_IN_EXACT_ONEDRIVE_FOLDER | `GET /me/drive/root:/path:/children` |
| 18 | Tool - Save Report To OneDrive | WF_SAVE_REPORT_TO_ONEDRIVE | `PUT /me/drive/root:/file:/content` (conflictBehavior=fail) |
| 19 | Tool - Save Report To Selected Folder | WF_SAVE_REPORT_TO_SELECTED_ONEDRIVE_FOLDER | `PUT /me/drive/root:/folder/file:/content` |
| 20 | Tool - List To Do Tasks | WF_LIST_TODO_TASKS | `GET /me/todo/lists` + `/tasks` |
| 21 | Tool - Create To Do Task | WF_CREATE_TODO_TASK | `GET /me/todo/lists` + `POST .../tasks` |
| 22 | Tool - Resolve Contact Email | WF_RESOLVE_CONTACT_EMAIL | `GET /me/people?$search=...` |
| 23 | Tool - Video Transcription | WF_VIDEO_TRANSCRIPTION | `GET drive item` / `GET attachment` metadata (size guard) |

## Numbered email memory

`WF_EMAIL_LIST_CONTEXT` is the heart of numbered follow-ups:

- `action=list` fetches the latest Inbox emails, stores a numbered list **per `telegramChatId`**
  in n8n workflow static data (`$getWorkflowStaticData('global')`), and returns the numbered list.
  For each email it stores: `listNumber`, `message_id`, `conversationId`, `subject`, `senderName`,
  `senderEmail`, `receivedDateTime`, `isRead`, `hasAttachments`, `webLink`, `bodyPreview`,
  `createdAt`, `telegramChatId`.
- `action=get` with an `emailNumber` looks up the saved list for the current chat and returns the
  **exact saved `message_id`** (it never re-searches latest emails). If the list is missing it tells
  the user to list emails again.

Follow-up tools (content, attachments, reply, save) all consume the `message_id` resolved from this
memory, so "Email 2" always refers to the same email the user saw.

## Timezone & language
- All date/time handling defaults to **Europe/Berlin** (calendar create/search, "today" ranges, To Do due dates).
- The agent replies in the language of the latest user message (English/German), following mid-conversation switches.
