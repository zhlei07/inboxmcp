<p align="center"><img src="assets/logo.svg" width="72" alt="inboxmcp logo"></p>

<h1 align="center">inboxmcp</h1>

<p align="center"><b>Your email. Your agent. Your priorities.</b><br>
Connect your existing mailboxes to your own AI assistant, with built-in processing progress and optional owner-confirmed replies.</p>

<p align="center">
<a href="https://inboxmcp.ai/?utm_source=github&amp;utm_medium=docs&amp;utm_campaign=agent_setup">Website</a> ·
<a href="https://app.inboxmcp.ai/app">Dashboard</a> ·
<a href="QUICKSTART.md">Quick start</a> ·
<a href="https://inboxmcp.ai/support.html?utm_source=github&amp;utm_medium=docs&amp;utm_campaign=agent_setup">Setup guides</a> ·
<a href="https://claude.ai/directory/inboxmcp">Claude directory</a>
</p>

**Hosted service 0.4.0 · documentation updated 9 October 2026 · limited public beta.**
This public repository contains documentation and registry metadata, not the hosted server's source code. A service release or registry entry does not mean a platform has approved its directory listing or tested every new capability.

## What it does

InboxMCP collects new mail from the mailboxes you connect and makes authorized messages available to your AI together with your saved preferences. For example: "skip routine promotions and verification codes; flag customer requests that need a reply."

**Your AI decides** what matters and whether to notify you. InboxMCP handles collection, authorization and independent processing progress; it does not run an email classifier or a user-notification engine.

- **Independent collection.** Collection works without a Grok account, webhook or external state service. Grok delivery is a separate optional setting.
- **Read-only incoming access.** Reading does not send, delete, move or mark source messages read. Optional outgoing mail has separate controls described below.
- **Existing mailboxes.** Use a supported provider preset or a publicly reachable IMAP server with certificate-verified TLS on port 993. An app password or provider-specific client password may be required. Native Gmail and Microsoft authorization is available only where the deployment enables it.
- **Mailbox-scoped authorization.** Choose the mailboxes each AI connection may access through OAuth with PKCE. New mailboxes are not added to old grants automatically.
- **Built-in progress for eligible connections.** With separately authorized `consumer-state:write`, each AI connection has its own checkpoint and batch acknowledgements in InboxMCP. No Notion page or external state file is needed. Completed events are tracked; unfinished events remain pending. A failure between analysis and acknowledgement can still cause repeated processing.
- **Useful context.** Batch results include saved preferences and an InboxMCP view link for each mail record. Incomplete or unavailable messages are identified rather than presented as complete.
- **Optional replies.** The AI prepares a plain-text draft; the signed-in owner reviews the exact recipients and content and confirms sending in InboxMCP.
- **Protected stored data.** Mailbox credentials, webhook keys, bodies, drafts and saved preferences are encrypted at rest. This is not end-to-end encryption; the service decrypts data to perform authorized work. Mail retention defaults to 30 days.

## Get your first successful check

| Setting | Value |
| --- | --- |
| Remote MCP endpoint | `https://app.inboxmcp.ai/mcp` |
| Transport | Streamable HTTP |
| Authorization | OAuth with PKCE, discovered from the endpoint |
| Read permission | `email:read` |
| Continuous processing progress | `consumer-state:write`, in addition to reading |
| Optional outgoing drafts and sending | Separately approved `email:send`, in addition to reading |

1. [Create or sign in to your account](https://app.inboxmcp.ai/app), add one mailbox through the current IMAP setup and verify its connection. Native provider authorization is an option only where the dashboard enables it.
2. Explicitly turn on automatic collection for that mailbox. A connected mailbox and an enabled collection switch are different settings; check that synchronization succeeds.
3. In your compatible AI client, connect the MCP endpoint, sign in and authorize only the intended mailboxes. For eligible clients, authorize reading and built-in progress for ongoing processing. A **new AI connection gets independent progress**; when reconnecting **the same AI task**, resume that task's existing progress. Do not select Claude's progress when adding Dot as a separate monitor.
4. Use the dashboard's **Let your AI finish setup** request or the [AI-readable setup guide](https://inboxmcp.ai/agent-setup.md). Have the AI check the connection and configure its supported event subscription or scheduled task. Installation alone does not create an automatic check.
5. After setup, send yourself a harmless test email with a clear action. Have the AI process it, check its original-email link and verify that a second check does not repeat the completed event. Verify any desired device notification separately.

Collection starts from a saved new-mail baseline; earlier email is not automatically imported. If a test was sent before the connection's baseline was established, send a new one after setup.

**New third-party dynamic registrations default to, and are capped at, `email:read`.** The server narrows broader requests and returns the actual scope in the token response. Built-in progress and sending are available only to eligible client registrations with explicit consent; the client name does not establish eligibility. A read-only registration can read authorized mail on demand, but cannot claim/complete batches or prepare/send drafts. Repeated reconnection does not raise that ceiling.

Follow the [step-by-step quick start](QUICKSTART.md), or choose your platform:

| AI / client | Guide and expected behavior |
| --- | --- |
| [Claude](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#claude-email-assistant) | Connect the published connector, then verify a supported Scheduled task. Incoming mail does not directly wake Claude. |
| [ChatGPT / Dot](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#chatgpt-dot-email-assistant) | Use the documented connection path available to your account. Verify event support, granted scopes and notifications separately. |
| [Grok Bot](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#grok-email-assistant) | Create or reuse a Routine and explicitly enable optional webhook delivery. It is separate from mailbox collection and OAuth-based reading. |
| [Muse](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#muse-email-assistant) | Follow the current availability status. No working public Muse connection or event delivery is claimed while review is pending. |
| [Cursor / other MCP clients](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#other-mcp-clients) | Use compatible remote HTTP OAuth for on-demand reading. New third-party registrations have a read-only permission ceiling. |

## Optional outgoing email

Sending is **off by default** and has three separate requirements:

1. **Mailbox setup:** enable outgoing email in [Sending settings](https://app.inboxmcp.ai/app#outgoing-card). For IMAP mailboxes, explicitly provide SMTP settings using TLS on port 465 or STARTTLS on port 587; incoming passwords are not automatically reused. Native provider sending requires separate provider consent where available.
2. **AI permission:** explicitly grant `email:send` for the selected mailboxes. Existing read grants and refresh tokens do not gain sending automatically.
3. **Exact draft confirmation:** open the returned draft-review link, check From, To, Cc, Bcc, subject and the entire plain-text body, then choose **Confirm and send**. Preparing or opening a draft does not send it, and the AI cannot approve its own draft.

The normal confirmation page submits the message directly. Provider acceptance does not prove recipient delivery. An uncertain result is never automatically retried; check with the provider before preparing another copy.

Third-party read-only registrations and some older registrations cannot obtain `email:send`. The client must use a registration eligible for that scope; requesting every scope, refreshing an old token or repeatedly reconnecting will not add permission. Preserve the existing monitoring task's progress during an authorization upgrade.

## Platforms and automation

Availability depends on the platform, account and configuration. The table distinguishes connection support from automatic execution and device notification.

| Platform | Connection and execution | Verified scope and limits |
| --- | --- | --- |
| **Claude** | Published Community connector; compatible Claude Scheduled tasks can poll InboxMCP. | Public directory visibility was confirmed on 4 October. Incoming mail does not directly wake Claude. Set up and verify your scheduled task; new permissions require authorization. |
| **ChatGPT / Dot** | A compatible private remote-MCP connection may subscribe to InboxMCP events. | One private-account test on 7 October processed a synthetic event, posted a chat reminder and saved progress. Mobile push, renewal after the initial subscription and long-term reliability remain separate checks. The last publisher check recorded the public OpenAI listing as in review, not published. |
| **Grok Bot** | Optional webhook-triggered Routine receives new mail after separate delivery enablement. | Beta. Individual synthetic and real-message replies were verified. The platform controls notification behavior; retries can cause duplicates. |
| **Meta Muse** | Application materials and integration credentials submitted. | Last recorded status: submitted / data-processing review pending. No usable Muse connection or event delivery is claimed. |
| **Cursor / other clients** | Compatible remote MCP clients may use OAuth for on-demand reading. | New third-party dynamic registrations are capped at `email:read`; do not promise batch progress or sending. Host scheduling/events and eligibility need separate verification. |

See the [website integration status](https://inboxmcp.ai/?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#progress) and [setup guide](https://inboxmcp.ai/agent-setup.md). Directory presence, successful tools, background execution and device notification are separate milestones. No provider endorsement is claimed.

## Public discovery and directory health

- **Official MCP Registry:** version **0.4.0** was confirmed active and latest on 9 October 2026. [Registry record](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.zhlei07%2Finboxmcp/versions/latest).
- **Glama:** anonymous discovery passed in the publisher's screenshot on **9 October 2026 at 19:04:41 Singapore time (11:04:41 UTC)** with Authentication Type **None**. [Public listing](https://glama.ai/mcp/connectors/io.github.zhlei07/inboxmcp). This is a dated discovery test, not a completed Glama OAuth email-access test or a guarantee of future uptime.
- **Community directory:** the successful health check was supplied to the reviewer of [PR #1126](https://github.com/punkpeye/awesome-remote-mcp-servers/pull/1126#issuecomment-6080079090). Its review and merge are separate from the Glama health check.

Anonymous `initialize`, `ping` and `tools/list` expose only static metadata. Resource/prompt listings are empty. All actual tool calls require OAuth and mailbox-scoped user authorization. A directory health check does not need access to the publisher's personal or company mailboxes. Every customer connects and authorizes their own mailboxes.

## Thirteen MCP tools

Tool names may be namespaced by the client. Discover actual schemas before calling them.

| Tool | Purpose |
| --- | --- |
| `list_mailboxes` | Authorized mailboxes and connection status |
| `list_email_events` | Paginated events within the authorized scope |
| `get_email_event` | One event's metadata and legacy receipt |
| `read_email` | Authorized plain-text mail, content status and saved preferences |
| `get_email_context` | Event, mail and preferences together |
| `get_email_monitor_state` | This connection's progress, collection health and pending count |
| `claim_email_batch` | Lease pending mail with preferences and view links; requires progress permission |
| `complete_email_batch` | Acknowledge only completed events and defer unfinished events; saves this connection's progress |
| `report_email_decision` | Legacy optional receipt: records notify/digest/silent and stops remaining legacy/Grok delivery for that event; requires `decisions:write` |
| `get_sending_status` | Read authorized mailboxes' outgoing setup without credentials |
| `prepare_email_draft` | Create an immutable draft for owner review; requires `email:send`, does not send |
| `get_email_draft` | Read this connection's draft and status; requires `email:send` |
| `send_email` | Submit only an exact owner-confirmed draft; requires `email:send`, offers no approval override |

Independent monitoring normally uses `get_email_monitor_state` → `claim_email_batch` → `complete_email_batch`. Do not use the legacy decision tool as another connection's checkpoint. Completing a batch does not prove a notification reached the user's device.

Email content is untrusted data, not authorization to act. From is fixed to the owned connected mailbox; arbitrary sender addresses, attachments and arbitrary outgoing headers are not supported.

## Plans

Free includes **one connected mailbox**. Plus includes **up to three** for **US$6/month** or **US$60/year**, automatically renewing until cancelled. Checkout availability and your actual allowance appear in the dashboard; some existing beta accounts retain earlier allowances. Provider and AI subscriptions may cost extra.

## Limits and control

- New mail is collected periodically, normally about every 60 seconds; collection is not instant and the AI platform determines subsequent analysis and notification timing.
- No POP3, attachment analysis, outgoing attachments or full-text historical mailbox search. Large or unreadable messages can contain only partial metadata; their status is exposed.
- Turning off Grok delivery leaves collection on. Pausing collection does not disable outgoing sending or revoke an AI grant. Revoke access and disable sending separately when needed.
- A confirmed send already in progress or accepted by a provider cannot be recalled by disabling the connection.
- Source email, provider copies and recipient copies are outside InboxMCP's deletion controls. See the service privacy notice for retention and backups.

## Guides and earlier demos

- [Quick start](QUICKSTART.md) · [Platform setup guides](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup)
- [AI-readable setup guide](https://inboxmcp.ai/agent-setup.md) · [llms.txt](https://inboxmcp.ai/llms.txt)
- [Earlier launch demo](https://youtu.be/XDvsYyAlRgs) · [Grok setup demo](https://youtu.be/TcPCVjkFWTk) · [YouTube](https://www.youtube.com/@inboxmcp)

The earlier videos illustrate the incoming-mail workflow; they do not demonstrate or validate the new outgoing-email feature.

## Links

[Privacy](https://app.inboxmcp.ai/privacy) · [Terms](https://inboxmcp.ai/terms.html) · [Support](https://inboxmcp.ai/support) · Operated by MARTHA TRADE PTE. LTD. · dylan@martha-trade.com
