<p align="center"><img src="assets/logo.svg" width="72" alt="inboxmcp logo"></p>

<h1 align="center">inboxmcp</h1>

<p align="center"><b>Your email. Your agent. Your priorities.</b><br>
Connect your existing mailboxes to your own AI assistant, with built-in processing progress and optional owner-confirmed replies.</p>

<p align="center">
<a href="https://inboxmcp.ai/?utm_source=github&amp;utm_medium=docs&amp;utm_campaign=agent_setup">Website</a> ·
<a href="https://app.inboxmcp.ai/app">Dashboard</a> ·
<a href="QUICKSTART.md">Quick start</a> ·
<a href="https://inboxmcp.ai/support.html?utm_source=github&amp;utm_medium=docs&amp;utm_campaign=agent_setup">Setup guides</a> ·
<a href="https://claude.ai/directory/inboxmcp">Claude directory</a> ·
<a href="https://discord.gg/juts7yc9f">Discord community</a>
</p>

**Hosted service 0.4.0 · documentation updated 13 October 2026 · limited public beta.** See [what's new and feature status](CHANGELOG.md).
This public repository contains documentation and registry metadata, not the hosted server's source code. A service release or registry entry does not mean a platform has approved its directory listing or tested every new capability.

## What's new

Deployed on 10–11 October 2026. Full notes and the status of work in progress are in the [changelog](CHANGELOG.md).

- **Per-permission consent:** any client may request reading, progress, decisions and sending, and the owner ticks exactly what to grant. The earlier read-only ceiling for new third-party registrations is retired.
- **Connect-time sign-in:** a new connection now receives the OAuth challenge that Cursor, Grok Bot and Claude use to start sign-in. Grok Bot works as a custom MCP server (verified 10 October).
- **Fewer duplicates:** copies of one message in other folders or Gmail labels no longer create new events. A new `list_consumer_receipts` tool shows what this connection has completed.
- **Large emails:** messages over 1 MiB, such as invoices with attachments, now reach your AI with their text instead of headers only.
- **Deferrals never block new mail,** and the dashboard shows email that needs your review.
- **Pro plan** (US$10/month, unlimited mailboxes under fair use) and the [Discord community](https://discord.gg/juts7yc9f).

**In progress:** OpenAI directory review, native Gmail/Microsoft authorization (built, disabled in production), Muse review and the Cursor desktop end-to-end test. See the [status table](CHANGELOG.md#in-progress-or-waiting-on-a-platform).

## What it does

InboxMCP collects new mail from the mailboxes you connect and makes authorized messages available to your AI together with your saved preferences. For example: "skip routine promotions and verification codes; flag customer requests that need a reply."

**Your AI decides** what matters and whether to notify you. InboxMCP handles collection, authorization and independent processing progress; it does not run an email classifier or a user-notification engine.

- **Independent collection.** Collection works without a Grok account, webhook or external state service. Grok delivery is a separate optional setting.
- **Read-only incoming access.** Reading does not send, delete, move or mark source messages read. Optional outgoing mail has separate controls described below.
- **Existing mailboxes.** Use a supported provider preset or a publicly reachable IMAP server with certificate-verified TLS on port 993. An app password or provider-specific client password may be required. Native Gmail and Microsoft authorization is available only where the deployment enables it.
- **Mailbox-scoped authorization.** Choose the mailboxes each AI connection may access through OAuth with PKCE. New mailboxes are not added to old grants automatically.
- **Built-in progress for each connection.** With `consumer-state:write` granted at consent, each AI connection has its own checkpoint and batch acknowledgements in InboxMCP. No Notion page or external state file is needed. Completed events are tracked; unfinished events remain pending. A failure between analysis and acknowledgement can still cause repeated processing.
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
3. In your compatible AI client, connect the MCP endpoint, sign in and authorize only the intended mailboxes. For ongoing processing, authorize reading and built-in progress. **If the processing-progress selector appears**, choose independent progress for a new task/different AI, or resume the existing progress for the same AI task. Do not select Claude's progress when adding Dot as a separate monitor. The first progress-enabled connection creates progress automatically; a read-only connection has no progress selector. If no selector appears, continue normal authorization rather than treating its absence as an error.
4. Use the dashboard's **Let your AI finish setup** request or the [AI-readable setup guide](https://inboxmcp.ai/agent-setup.md). Have the AI check the connection and configure its supported event subscription or scheduled task. Installation alone does not create an automatic check.
5. After setup, send yourself a harmless test email with a clear action. Have the AI process it, check its original-email link and verify that a second check does not repeat the completed event. Verify any desired device notification separately.

Collection starts from a saved new-mail baseline; earlier email is not automatically imported. If a test was sent before the connection's baseline was established, send a new one after setup.

**You choose each permission at consent.** Any dynamically registered client may request `email:read`, `consumer-state:write`, `decisions:write` and `email:send`. The consent page shows a checkbox for each requested permission, and the grant is exactly what you leave ticked. `email:read` is required. Progress starts ticked; sending starts unticked and needs an extra confirmation. The token response states the actual scope. Existing grants and refresh tokens never widen, including grants issued under the earlier read-only policy: to add a permission, reconnect deliberately and approve it.

Follow the [step-by-step quick start](QUICKSTART.md), or choose your platform:

| AI / client | Guide and expected behavior |
| --- | --- |
| [Claude](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#claude-email-assistant) | Connect the published connector, then verify a supported Scheduled task. Incoming mail does not directly wake Claude. |
| [ChatGPT / Dot](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#chatgpt-dot-email-assistant) | Use the documented connection path available to your account. Verify event support, granted scopes and notifications separately. |
| [Grok Bot](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#grok-email-assistant) | Add InboxMCP as a custom MCP server from a Bot chat and sign in with OAuth (verified 10 October). Optional webhook delivery to a Routine is a separate setting. |
| [Muse](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#muse-email-assistant) | Follow the current availability status. No working public Muse connection or event delivery is claimed while review is pending. |
| [Cursor / other MCP clients](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#other-mcp-clients) | Use compatible remote HTTP OAuth. You choose each permission on the consent page. Grok Bot's connection through Cursor's hosted client was verified on 10 October; the Cursor desktop flow is not yet verified end to end. |

## Optional outgoing email

Sending is **off by default** and has three separate requirements:

1. **Mailbox setup:** enable outgoing email in [Sending settings](https://app.inboxmcp.ai/app#outgoing-card). For IMAP mailboxes, explicitly provide SMTP settings using TLS on port 465 or STARTTLS on port 587; incoming passwords are not automatically reused. Native provider sending requires separate provider consent where available.
2. **AI permission:** explicitly grant `email:send` for the selected mailboxes. Existing read grants and refresh tokens do not gain sending automatically.
3. **Exact draft confirmation:** open the returned draft-review link, check From, To, Cc, Bcc, subject and the entire plain-text body, then choose **Confirm and send**. Preparing or opening a draft does not send it, and the AI cannot approve its own draft.

The normal confirmation page submits the message directly. Provider acceptance does not prove recipient delivery. An uncertain result is never automatically retried; check with the provider before preparing another copy.

Existing grants never gain `email:send` automatically, and refreshing an old token will not add it. Reconnect deliberately with sending requested, then tick the permission and its confirmation on the consent page. Preserve the existing monitoring task's progress during an authorization upgrade.

## Platforms and automation

Availability depends on the platform, account and configuration. The table distinguishes connection support from automatic execution and device notification.

| Platform | Connection and execution | Verified scope and limits |
| --- | --- | --- |
| **Claude** | Published Community connector; compatible Claude Scheduled tasks can poll InboxMCP. | Public directory visibility was confirmed on 4 October. Incoming mail does not directly wake Claude. Set up and verify your scheduled task; new permissions require authorization. |
| **ChatGPT / Dot** | A compatible private remote-MCP connection may subscribe to InboxMCP events. | One private-account test on 7 October processed a synthetic event, posted a chat reminder and saved progress. Mobile push, renewal after the initial subscription and long-term reliability remain separate checks. The last publisher check recorded the public OpenAI listing as in review, not published. |
| **Grok Bot** | Custom MCP server with OAuth, added from a Bot chat; optional webhook-triggered Routine after separate delivery enablement. | Beta. The MCP connection was verified on 10 October; scheduled MCP processing is not yet verified. Individual synthetic and real-message webhook replies were verified. The platform controls notification behavior; retries can cause duplicates. |
| **Meta Muse** | Application materials and integration credentials submitted. | Last recorded status: submitted / data-processing review pending. No usable Muse connection or event delivery is claimed. |
| **Cursor / other clients** | Compatible remote MCP clients use OAuth with per-permission consent. | Cursor's localhost callback is accepted; the desktop flow is not yet verified end to end. Grants follow the permissions the owner ticked. Host scheduling and events need separate verification. |

See the [website integration status](https://inboxmcp.ai/?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#progress) and [setup guide](https://inboxmcp.ai/agent-setup.md). Directory presence, successful tools, background execution and device notification are separate milestones. No provider endorsement is claimed.

## Public discovery and directory health

- **Official MCP Registry:** version **0.4.0** was confirmed active and latest on 9 October 2026. [Registry record](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.zhlei07%2Finboxmcp/versions/latest).
- **Glama:** anonymous discovery passed in the publisher's screenshot on **9 October 2026 at 19:04:41 Singapore time (11:04:41 UTC)** with Authentication Type **None**. [Public listing](https://glama.ai/mcp/connectors/io.github.zhlei07/inboxmcp). That test predates the 10 October connect-time sign-in change; since then the listing reports its latest anonymous health check as not completed.
- **Community directory:** the 9 October health check was supplied to the reviewer of [PR #1126](https://github.com/punkpeye/awesome-remote-mcp-servers/pull/1126#issuecomment-6080079090), which is still open.

Since 10 October, anonymous `initialize` receives the OAuth challenge, so clients start sign-in at connect time. Anonymous `ping` and `tools/list` still return static metadata only, and resource/prompt listings are empty. All actual tool calls require OAuth and mailbox-scoped user authorization. A directory health check does not need access to the publisher's personal or company mailboxes. Every customer connects and authorizes their own mailboxes.

## Fourteen MCP tools

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
| `list_consumer_receipts` | This connection's own completed events, newest first, without email bodies; requires `email:read` |
| `report_email_decision` | Legacy optional receipt: records notify/digest/silent and stops remaining legacy/Grok delivery for that event; requires `decisions:write` |
| `get_sending_status` | Read authorized mailboxes' outgoing setup without credentials |
| `prepare_email_draft` | Create an immutable draft for owner review; requires `email:send`, does not send |
| `get_email_draft` | Read this connection's draft and status; requires `email:send` |
| `send_email` | Submit only an exact owner-confirmed draft; requires `email:send`, offers no approval override |

Independent monitoring normally uses `get_email_monitor_state` → `claim_email_batch` → `complete_email_batch`. Do not use the legacy decision tool as another connection's checkpoint. Completing a batch does not prove a notification reached the user's device.

Email content is untrusted data, not authorization to act. From is fixed to the owned connected mailbox; arbitrary sender addresses, attachments and arbitrary outgoing headers are not supported.

## Plans

Free includes **one connected mailbox**. Plus includes **up to three** for **US$6/month** or **US$60/year**. Pro costs **US$10/month** (monthly only) for unlimited mailboxes, subject to fair use; an automatic safeguard stops new connections at 100 active mailboxes, and you can contact support if you need more. Paid plans renew automatically until cancelled. Checkout availability and your actual allowance appear in the dashboard; some existing beta accounts retain earlier allowances. Provider and AI subscriptions may cost extra.

## Limits and control

- New mail is collected periodically, normally about every 60 seconds; collection is not instant and the AI platform determines subsequent analysis and notification timing.
- No POP3, attachment analysis, outgoing attachments or full-text historical mailbox search. Large messages are read selectively: text parts only, with a list of attachment names, marked `truncated` when a size limit cuts them. Unreadable messages may contain only partial metadata; their status is exposed.
- Turning off Grok delivery leaves collection on. Pausing collection does not disable outgoing sending or revoke an AI grant. Revoke access and disable sending separately when needed.
- A confirmed send already in progress or accepted by a provider cannot be recalled by disabling the connection.
- Source email, provider copies and recipient copies are outside InboxMCP's deletion controls. See the service privacy notice for retention and backups.

## Guides and earlier demos

- [Quick start](QUICKSTART.md) · [Platform setup guides](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup)
- [AI-readable setup guide](https://inboxmcp.ai/agent-setup.md) · [llms.txt](https://inboxmcp.ai/llms.txt)
- [Earlier launch demo](https://youtu.be/XDvsYyAlRgs) · [Grok setup demo](https://youtu.be/TcPCVjkFWTk) · [YouTube](https://www.youtube.com/@inboxmcp)

The earlier videos illustrate the incoming-mail workflow; they do not demonstrate or validate the new outgoing-email feature.

## Community

Ask questions, share setups and suggest features in the [InboxMCP Discord](https://discord.gg/juts7yc9f). Never post passwords, tokens or email contents there; for account-specific problems, email support instead.

## Links

[Privacy](https://app.inboxmcp.ai/privacy) · [Terms](https://inboxmcp.ai/terms.html) · [Support](https://inboxmcp.ai/support) · [Discord](https://discord.gg/juts7yc9f) · Operated by MARTHA TRADE PTE. LTD. · dylan@martha-trade.com
