# dsh-cross-session-agent — DSH 0.2 compatibility fork

English | [中文](README.md)

An unofficial MIT fork of [dd2673/dsh-cross-session-agent](https://github.com/dd2673/dsh-cross-session-agent), targeting **DSH 0.2.0-rc.2 and Node 24**. Other DSH versions are not claimed as compatible.

Supports an explicitly authorized implement → review → fix → re-review workflow between two existing root/fork Sessions. All four host tools require a configured pair of exact Session IDs and identical, nonempty `cwd` values. **No configured pair means no cross-session authority.** Self, real subagents, archived Sessions and cross-workspace targets are denied.

## Install and configure

Start with a disposable `DSH_HOME` before installing into a personal profile:

```sh
dsh plugin --profile desktop add 'github:lilongsky/dsh-cross-session-agent#fork-v0.1.3' --ignore-scripts
```

Use `desktop` for the desktop app or `web` for the Web UI. The installed desktop profile was live-tested.

The bundle patch loads the plugin without granting any pair. Add a row to that profile's `cordis.patch.yml`, replacing the placeholders with the full stable Session IDs:

```yaml
- id: cross-session-agent
  config:
    allowedPairs:
      - id: review-task-001
        sessions: [session-FULL-A-ID, session-FULL-B-ID]
        maxMessages: 20
```

The coordination `id` accepts 1–64 ASCII letters, digits, underscores or hyphens. Each pair and each id must be unique. Titles are display data, never routing identities. `maxMessages` is a **bidirectional message count**, not a round count: default 20, range 1–100. It is counted from both Sessions' own native Inbox events and survives plugin/process restarts. Per-pair delivery serialization prevents concurrent budget overruns. The existing process-local rate limit allows 10 messages per pair per 60 seconds.

For a separate task, the user configures a new coordination id. Agents cannot reset the budget through tools. Remove the pair configuration to revoke access after completion. Harness continues to own each Session's permissions and approval policies.

## Copy a Session ID

Use the sidebar Session row’s **… → Copy session ID** to copy that row’s complete stable identity without opening it. The conversation header also retains its copy button. Titles are not identities, and copying an ID grants no messaging authority.

## Workflow and tools

Give A and B their orchestration duties explicitly. For example, A implements and sends its changes and validation to B, fixes B's concrete findings, and stops after three fix rounds or a blocker. B reviews, returns actionable findings, re-reviews, and finishes with an explicit pass without creating acknowledgement loops.

Pair configuration grants access; it does not substitute for user intent. An `@Session` reference or UI candidate grants no authority; the host rechecks every operation.

| Tool | Behavior |
| --- | --- |
| `list_peer_agents` | Only authorized, same-workspace, unarchived peers |
| `send_agent_message` | `followup` queues the next turn and resumes cold targets through the public factory; `steer` / `inject` require a running target |
| `check_delivery` | Only the calling sender's deliveries; an explicit messageId can be queried after restart |
| `get_peer_context` | Bounded, read-only overview/conversation/activity; never resumes or wakes a target |

Accepted, pending, claimed and discarded are transport facts. They are not proof of reading, replying or completed work. Both live and cold receipts use public SessionQuery snapshots, excluding a fork's inherited parent Inbox prefix.

## Privacy and lifecycle

The plugin runtime performs no networking, telemetry, credential-file access or shell execution, and creates no separate mailbox/database. Relays retain the real sender ID and plugin source instead of impersonating user consent. The React/textContent client uses the DSH 0.2 icon export and retains read-only rendering of historical dsh-agent-message cards.

The context whitelist includes direct user text, canonical compact checkpoints, completed assistant text and bounded structural tool/turn activity. It excludes reasoning, tool arguments/results, error text, system prompts, plugin/relay messages, approvals and attachments. Error codes are limited to safe symbols of at most 64 characters; others become TOOL_ERROR. **Allowed conversation text is not scanned for secrets.** Secrets placed in readable user/assistant text can be visible to an authorized peer. See [PRIVACY.md](PRIVACY.md).

Cancellation is checked around asynchronous reads, resume and native delivery. Archive and identity checks also run in resume setup and its synchronous publication commit. A message already accepted by Harness is not retracted by a later sender stop. Cold resume can continue existing native Inbox work and therefore still requires authorized orchestration. Handles are owned until plugin unload; a failed pre-delivery check releases a newly acquired handle.

## Verify

```sh
pnpm install --frozen-lockfile --ignore-scripts
pnpm test
node scripts/live-profile-e2e.mjs /path/to/dsh
```

The acceptance runner creates a new disposable DSH_HOME and boots then restarts the real profile, Agent loops, tool dispatcher, SessionQuery and JSONL persistence. Its deterministic local provider makes no online model requests. It accepts no existing server URL and sends nothing to personal Sessions. See [TDD_CONTRACT.md](TDD_CONTRACT.md). This does not prove that arbitrary online models obey the workflow or that future revisions are safe.

MIT attribution remains in [NOTICE.md](NOTICE.md). This fork does not publish to the upstream npm name; its npm publish workflow is disabled.

[0.1.3 验证结果 / verification results](docs/validation-0.2.0-rc.2.md)
