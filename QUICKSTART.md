# Connect your own mailbox to your AI

InboxMCP is a hosted, multi-user email connector. Each person creates their own account, connects their own mailboxes and chooses which mailboxes an AI may access. Installing a connector does not expose the publisher's mailbox or another user's mail.

**Remote endpoint:** `https://app.inboxmcp.ai/mcp`

**Transport:** Streamable HTTP

**Authentication for email tools:** OAuth with PKCE, discovered from the endpoint

No local server deployment is required. Keep the endpoint exactly as shown; do not add tracking parameters, passwords or tokens to it.

## 1. Connect and collect

1. [Open InboxMCP](https://inboxmcp.ai/?utm_source=github&utm_medium=docs&utm_campaign=agent_setup) and create or sign in to your account.
2. Add a mailbox in the [dashboard](https://app.inboxmcp.ai/app). The current public setup uses supported IMAP providers with TLS on port 993; an app password or provider-specific client password may be required. Native Gmail/Microsoft authorization is usable only if the dashboard actually offers it. Enter credentials only on the provider or InboxMCP website, never in an AI chat.
3. Explicitly enable automatic collection and check that synchronization succeeds. Mailbox collection works independently of Grok delivery.
4. Save the preferences you want your AI to consider when reviewing mail.

Mailbox collection begins at its new-mail baseline; it is not a full historical mailbox import.

## 2. Choose your AI and authorize it

Use the platform-specific guide below. A compatible remote MCP client connects to the endpoint above, then opens InboxMCP's sign-in and consent page. Select only your intended mailboxes.

| AI / client | Guide and expected behavior |
| --- | --- |
| [Claude](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#claude-email-assistant) | Connect the published connector, then verify a supported Scheduled task. Incoming mail does not directly wake Claude. |
| [ChatGPT / Dot](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#chatgpt-dot-email-assistant) | Use the documented connection path available to your account. Verify event support, granted scopes and notifications separately. |
| [Grok Bot](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#grok-email-assistant) | Create or reuse a Routine and explicitly enable optional webhook delivery. It is separate from mailbox collection and OAuth-based reading. |
| [Muse](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#muse-email-assistant) | Follow the current availability status. No working public Muse connection or event delivery is claimed while review is pending. |
| [Cursor / other MCP clients](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#other-mcp-clients) | Use compatible remote HTTP OAuth for on-demand reading. New third-party registrations have a read-only permission ceiling. |

**If the processing-progress selector appears**, choose the appropriate option:

- **New task or a different AI:** choose an independent connection starting from new mail. For example, adding Dot alongside Claude should not replace Claude's progress.
- **Reconnect the same AI task:** resume that task's existing progress. Do not create a replacement task or reset its baseline just to reconnect.
- **Change the authorized mailboxes:** confirm the change. Retained mailboxes continue their progress; newly added mailboxes begin at the new authorization boundary.

The selector appears only when built-in progress is authorized and previous progress is available to choose from. The first eligible connection creates its progress automatically; read-only connections have no selector. If no selector appears, continue normal authorization. Its absence is not an error, and reconnecting cannot remove a read-only permission ceiling. Do not select another AI's progress merely because it is the only existing entry.

### Permissions depend on the client registration

- `email:read` allows reading the selected mailboxes.
- `consumer-state:write`, when available and separately authorized with reading, allows InboxMCP to save that connection's processing progress.
- `email:send`, when available and separately authorized with reading, allows draft preparation and sending only after the owner confirms the exact draft on the website.

**New third-party dynamic registrations default to, and are capped at, `email:read`.** Requesting every advertised scope does not grant every scope: the server narrows the request, and the issued token states the actual scope. Additional permissions are available only to eligible client registrations; a familiar client name alone does not grant them.

If your connection receives only `email:read`, use it for on-demand reading. Do not promise built-in batch acknowledgements or outgoing mail, create an external progress database as a workaround, or repeatedly reconnect expecting the permission ceiling to change. Check the guide for the supported platform path or contact support.

## 3. Ask your AI to verify setup

Give your AI this request:

> Read https://inboxmcp.ai/agent-setup.md and use my existing InboxMCP connection. First call list_mailboxes to verify the authorized mailbox scope. If built-in progress is authorized, call get_email_monitor_state and use claim_email_batch / complete_email_batch for this connection. For eligible monitoring, preserve an existing task's progress; a new task or different AI uses independent progress. Only choose a progress option if the selector appears; the first eligible connection creates progress automatically, and read-only connections have no selector. Do not create a Notion page, Drive file or local cursor. Configure automatic checks only if this platform and account actually support a schedule or event subscription. State what was tested and what still needs verification. Do not enable sending or change mailbox permissions unless I request it.

The [AI-readable guide](https://inboxmcp.ai/agent-setup.md) is the detailed reference; [llms.txt](https://inboxmcp.ai/llms.txt) is its discovery index. Both are instructions to inspect, not proof that a particular client supports background execution.

For an eligible monitoring connection:

1. Check collection health and pending work using `get_email_monitor_state`.
2. Claim a batch, analyze each readable item and use only the returned `view_url` in a mail reminder.
3. Complete actually processed event IDs, including intentional no-reminder decisions. Defer unreadable or unfinished IDs and continue other readable items.
4. Stop when idle; respect busy/retry times. InboxMCP saves the progress without an external state service.

Your AI platform supplies its schedule, background execution and notifications. A successful MCP connection alone does not create any of those.

## 4. Verify one new message, then check for duplicates

After authorization and setup, send yourself a harmless email with a clear requested action. Ask the AI to check it, open the returned mail-record link and check again with no new mail. For batch monitoring, the completed event should not be processed again by that same connection. Another independently authorized AI has its own progress.

Verify a real future scheduled/event-triggered run and device notification separately. Do not use a manually successful run as proof that background monitoring or mobile push works.

If a message is unreadable, defer it rather than recording it as successfully processed or silent. If collection is paused, permission is missing or retention removed pending mail, report the problem instead of claiming the monitor is working.

## Directory health checks are not mailbox access

Glama's anonymous connection test was confirmed successful in the publisher's screenshot on **9 October 2026 at 19:04:41 Singapore time (11:04:41 UTC)**, with Authentication Type set to **None**. This is a dated test result, not a promise of future uptime or a completed Glama OAuth email-access test.

Anonymous discovery exposes only static handshake/tool metadata (`initialize`, `ping`, `tools/list` and empty resource/prompt catalogs). All actual tool calls still require OAuth and the user's selected-mailbox authorization. A directory does not need the publisher's real mailbox authorization to check discovery health.

## Optional outgoing mail

Check `get_sending_status`, enable outgoing setup on the website if desired, and authorize `email:send` only through an eligible client. The AI can prepare an immutable draft and share its review link. The signed-in owner reviews its exact recipients and content and confirms sending in InboxMCP. The AI has no approval override. Provider acceptance is not proof of recipient delivery; an uncertain result must not be automatically retried.

[Full AI-readable reference](https://inboxmcp.ai/agent-setup.md) · [Support](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup)
