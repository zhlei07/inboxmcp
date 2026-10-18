# What's new and feature status

InboxMCP is a hosted service, so updates reach every account when they are deployed; there is nothing to install or upgrade. The MCP server still reports version **0.4.0**: the October 10–13 changes below were deployed without a new version number. This repository holds documentation only, not the server's source code.

Status checked **19 October 2026**. Platform and directory review states keep the date of their last recorded check. Nothing here implies approval or endorsement by a platform.

## Feature status

### Live now

| Feature | Since | Notes |
| --- | --- | --- |
| Independent mailbox collection over IMAP (TLS, port 993) | Beta launch | Runs without Grok, webhooks or an external state service. Collection is periodic, normally about every 60 seconds. |
| Built-in processing progress for each AI connection | 4 Oct | `get_email_monitor_state` → `claim_email_batch` → `complete_email_batch`. No Notion page, Drive file or client cursor. |
| Owner-confirmed outgoing email | 9 Oct (0.4.0) | Needs mailbox sending (on by default for new IMAP mailboxes since 13 Oct), a separate `email:send` grant that starts unticked, and the owner's confirmation of every exact draft. |
| Per-permission consent | 10 Oct | The owner ticks each requested permission. Details below. |
| Connect-time sign-in for Cursor, Grok Bot and Claude | 10 Oct | Anonymous `initialize` now receives the OAuth challenge, which is what starts these clients' sign-in. |
| Grok Bot as a custom MCP server | 10 Oct | Connection verified on 10 October through Cursor's hosted client. Scheduled MCP processing in Grok Bot is not yet verified. |
| Message-ID deduplication and `list_consumer_receipts` | 10 Oct | 14 MCP tools in total. |
| Text of large IMAP messages | 10 Oct | Read selectively instead of headers only. |
| Deferral reporting that never blocks new mail | 10 Oct | `retry_can_help`, `needs_owner_review` and `expired_deferred_events`, also shown on the dashboard. |
| Pro plan | 10 Oct | US$10/month, unlimited mailboxes subject to fair use, with an automatic safeguard at 100 active mailboxes. |
| Discord community | 11 Oct | [discord.gg/juts7yc9f](https://discord.gg/juts7yc9f), also linked from a dismissible dashboard card. |
| Sending offered at connect | 13 Oct | The connect-time challenge now requests `email:send`, so clients such as Claude can offer it. It starts unticked, needs its own confirmation, and every draft still needs the owner's approval. |
| Directory health checks answered again | 13 Oct | Recognised directory health checks get a data-free `initialize`. Glama reports InboxMCP as healthy with 14 tools (tested 13 October 00:13 UTC). Every other client keeps the connect-time OAuth challenge. |
| MCP prompts | 13 Oct | `set_up_email_monitoring` and `check_new_email`, for clients that list connector prompts. |
| Outlook and Microsoft 365 with Microsoft sign-in | 17 Oct | **Connect Outlook / Microsoft 365** in the dashboard; no app password. Work tenants may need their administrator's approval. New mail is collected about every minute. |
| Change an AI connection's mailboxes in place | 17 Oct | Tick or untick mailboxes under AI connections; the agent sees the change on its next request, without reconnecting. |
| Collection and sending on by default | 13 Oct | A newly added IMAP mailbox starts collecting and, after a verified SMTP sign-in with the same login, can send. Either can be unticked before saving. Existing mailboxes keep their settings and get one-click buttons. |

### In progress or waiting on a platform

| Item | Current state | Still needed |
| --- | --- | --- |
| ChatGPT / OpenAI directory listing | Plugin 0.1.2 was still *In review*, not published, at the portal check on 19 October. A private plugin works without it. | OpenAI's review decision. |
| ChatGPT Dot event-triggered runs | Live for anyone whose ChatGPT plan has dots, as a private plugin. One Pro account ran from 7 to 13 October with daily subscription renewal and 537 emails completed. | Mobile push and long-term reliability. Each account still needs its own check. |
| Native Gmail sign-in | Built and tested with synthetic provider responses. **Not enabled in production.** (Microsoft sign-in is live since 17 October.) | Google app registration and review, followed by real authorization, collection and refresh tests. Until then, Gmail uses IMAP with an app password. |
| Cursor Marketplace | Cursor web and Cursor Desktop (localhost callback) both completed sign-in and read mail on 18 October. The Marketplace application from 2 October had no decision at the 19 October check. | The marketplace decision. |
| Grok Bot scheduled processing | The MCP connection works (10 October). The optional webhook Routine has passed single synthetic and real replies. | A Routine or schedule that calls InboxMCP tools unattended, and batch delivery. |
| Meta Muse | Application materials submitted; questionnaire under review. | Muse's review and an official event contract. No Muse connection is available yet. |
| Claude wake-on-arrival | Not available on ordinary Claude plans. Scheduled tasks are the supported route. | A platform feature for arrival-triggered runs. |
| Claude directory listing | The live listing still describes the original six tools. An updated listing with all 14 tools and optional owner-confirmed sending was submitted for Anthropic's review on 13 October and was still in review on 19 October. | Anthropic's review; the live listing is unchanged until then. |
| Community directory | Glama is healthy (tested 18 October, TDQS A). InboxMCP is listed in [awesome-remote-mcp-servers](https://github.com/punkpeye/awesome-remote-mcp-servers) since 15 October (PR #1126 merged). | Nothing pending. |

### Not currently supported

POP3, attachment analysis, outgoing attachments, full-text historical mailbox search, arbitrary sender addresses and autonomous sending without owner confirmation.

## Release notes

### 13 October 2026 (server version 0.4.0)

- **Deploys no longer interrupt the service.** A standby server from the new version takes requests while the main one is replaced. Each deploy used to cause about 30 seconds of HTTP 502, which could fail an AI's call or a directory health check. A 10-minute check across the next deploy got 396 of 396 successful answers.
- **Text from HTML when the plain part is empty or a placeholder.** Some emails carry an empty or placeholder plain-text part, such as "View online", beside the real message in HTML. Your AI now gets the HTML text. An email with no text at all, such as a lone ZIP, now lists its attachments (names, types and sizes only) instead of arriving blank. Copies stored earlier with short text are re-read once.
- **Open mail no longer crowds out new mail.** An email that an AI leaves without an outcome when its 15-minute lease ends now counts as a deferral. It comes back after newer mail, and after three lapses with growing delays (up to one hour). Repeated lapses flag it for owner review. If retention removes it, it is reported once instead of stopping the connection's claims. Found on a live Dot that kept about 17 unreadable emails open as reminders.
- **ChatGPT Dot for everyone with dots.** The [Dot guide](https://inboxmcp.ai/support.html?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#chatgpt-dot-email-assistant) now covers who can set it up (ChatGPT Pro outside the EEA, Switzerland and the UK, or Business Premium) and the current steps: Plugins → Add → Add custom MCP server, OAuth, Create as a plugin. It also reports a week of results from one Pro account. Dot should give every claimed email an outcome in the same run instead of keeping it open as a reminder.
- **Sending can be offered at connect.** Claude and SDK clients request exactly the scope named in the connect-time challenge, which now includes `email:send`. Requesting is not granting: on the consent page sending and its confirmation start unticked, and each draft still needs the owner's approval in InboxMCP.
- **Consent page guard.** Ticking only one of the sending permission and its confirmation now shows a hint instead of an error page. The page also says that InboxMCP does not create schedules, and where to set one up, when progress is requested.
- **Directory health checks.** An `initialize` from a recognised directory health check (Glama, mcpbeat, or a client name containing "probe" or "uptime") gets the static handshake again. Cursor, Claude, Codex, ChatGPT and any unrecognised client keep the OAuth challenge they need to start sign-in. Glama now reports InboxMCP as healthy with 14 tools.
- **MCP prompts.** `set_up_email_monitoring` (optional frequency, time zone and when to notify) asks the AI to create a recurring check with the built-in progress workflow, and `check_new_email` (optional focus) processes new mail once. They carry no account data and grant nothing.
- **Collection and sending on by default for new mailboxes.** Adding a mailbox now starts collection and turns on sending in one step. InboxMCP derives the SMTP server from the IMAP server within the same domain (for example `imap.qq.com` → `smtp.qq.com`, port 465; iCloud uses STARTTLS on 587), verifies a sign-in with the receiving login and stores an encrypted copy. If the provider refuses it or the server is unknown, nothing is stored and receiving is unaffected; Microsoft mailboxes are excluded because they no longer accept passwords. Existing mailboxes are not changed: the Send email card offers **Use this mailbox’s receiving password** and **Enable sending for all my mailboxes**. Your AI still needs `email:send`, which starts unticked, and each draft still needs your confirmation. The service privacy notice and terms describe the change.
- **Mailbox providers.** The [support page](https://inboxmcp.ai/support?utm_source=github&utm_medium=docs&utm_campaign=agent_setup#mailbox-providers) lists every IMAP preset, including Chinese personal and business providers, and the providers that cannot connect.

### 10–11 October 2026 (server version 0.4.0)

**Connecting your AI**

- **Per-permission consent.** Any dynamically registered client may request `email:read`, `consumer-state:write`, `decisions:write` and `email:send`. The consent page shows a checkbox for each requested permission, and the grant is exactly what the owner leaves ticked. `email:read` is required. Progress and decisions start ticked. Sending starts unticked and needs an extra confirmation, and each actual send still needs the owner's approval of the exact draft. The previous read-only ceiling for new third-party registrations is retired.
- **Existing grants never widen.** Grants issued under the earlier read-only ceiling stay read-only, and a refresh can only keep or narrow a grant. To add a permission, reconnect deliberately and approve it.
- **Connect-time sign-in.** Anonymous `initialize` now returns the OAuth challenge. Cursor and Claude document that interactive sign-in starts from this challenge; Grok Bot's connection was verified with it on 10 October. The challenge again requests progress permission, so new Claude connections can use built-in progress.
- **Native-app callbacks.** RFC 8252 loopback callbacks on any port, and custom schemes such as `cursor://`, are accepted, with stricter callback validation.
- **Consent is submitted once.** A double click sends one approval. A request that was already used now shows a page saying the connection has probably succeeded, instead of a bare error.
- Unknown well-formed scopes such as `offline_access` are ignored instead of failing the request.

**Reading mail**

- **Message-ID deduplication.** Within one mailbox, copies of the same message in other folders or Gmail labels, moves and resends no longer create new events, Grok deliveries or AI queue items. Messages with a missing or malformed Message-ID are never merged, and different mailboxes stay separate.
- **New tool `list_consumer_receipts`** (`email:read`). Lists this connection's own completed events, newest first, without email bodies.
- **Large IMAP messages.** Messages over 1 MiB, such as requests for quotation or invoices with PDF or Excel attachments, previously reached the AI as headers only. Their text parts are now read selectively, without setting the read flag or downloading attachments. A short list of attachment names, types and approximate sizes follows the text. Text cut by a size limit is marked `truncated`.
- Partly read Gmail API text is reported as `truncated`, including messages already stored.

**Processing progress**

- Every claimed email reports `retry_can_help`. Agents defer an email only while a retry can help. Otherwise they decide from the sender, subject and any available text, and tell the owner when that cannot rule out something important.
- `needs_owner_review` marks email that was deferred repeatedly or is close to retention removal.
- If retention removes an email that this connection deferred, the event is reported once in `expired_deferred_events` and kept in a recent summary. It no longer blocks new mail.
- The dashboard's connection card shows repeatedly deferred and expired-deferral counts, in English, Chinese and Spanish.

**Outgoing email safety**

- Drafts with RFC 2047 encoded words in recipient local parts or subjects are rejected, so a recipient or subject cannot look different on the review page from what is delivered. The built message must parse back to exactly the reviewed recipients before any connection opens.

**Plans and billing**

- **Pro** at US$10/month: unlimited mailboxes subject to fair use, with an automatic safeguard that stops new connections at 100 active mailboxes. When upgrading from Plus, Plus renewal stops automatically once the first Pro payment is verified. The remaining Plus period is not refunded.
- Checkouts that were never paid are closed and hidden from the subscription list. An abandoned Pro checkout no longer blocks buying Plus.

**Guides and community**

- Grok Bot MCP setup was added to the setup guides. The privacy notice now covers Message-ID hashes, OAuth logs and the body-free expired-deferral notices.
- Discord community launched, with a dismissible invitation card on the dashboard.

### 9 October 2026: 0.4.0

- Owner-confirmed outgoing email: SMTP setup, a separate `email:send` permission, immutable drafts, and sending only after the owner confirms the exact draft.
- A dashboard checklist that guides the first successful email test.
- Version 0.4.0 published to the official MCP Registry. Glama's anonymous discovery test passed; this predates the 10 October connect-time sign-in change.

### Earlier in October 2026

- **8 Oct:** retained IMAP placeholders are recovered in the background, and failed items are deferred without blocking an AI's queue.
- **7 Oct:** a private ChatGPT Dot account processed a synthetic email event automatically, posted a chat reminder and saved progress.
- **4 Oct:** built-in progress for each AI connection, with leases, partial acknowledgement and idempotent retries. Also: the server-side MCP Events extension and an optional Grok batch format. Claude Community connector visibility confirmed.

Questions or feature requests: [Discord](https://discord.gg/juts7yc9f) or [support](https://inboxmcp.ai/support?utm_source=github&utm_medium=docs&utm_campaign=agent_setup).
