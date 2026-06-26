# Known Limitations

- Telegram credential was not present in the legacy exports; select it manually after import.
- Contact emails are intentionally blank placeholders in `WF_RESOLVE_CONTACT_EMAIL`; add verified company emails before named-contact actions.
- Attachment binary send/upload loops are scaffolded with real metadata endpoints and safe responses; live binary transfer should be tested and completed inside n8n after credentials are selected.
- `WF_SAVE_TODAY_UNREAD_ATTACHMENTS_TO_ONEDRIVE_ROOT` identifies today unread attachment emails; full per-attachment binary save loop needs live n8n testing.
- Video transcription is not configured because no transcription API credential was provided. The workflow fails cleanly and handles files over 25MB.
- Email pagination is implemented up to the requested limit of 25 in the main assistant flow. True 1000-email pagination is not implemented.
- Microsoft To Do task creation uses `tasks` as a default list identifier unless a concrete list ID is supplied; configure tenant-specific default list if needed.
