# 0.1.3 fork verification — 2026-10-13

Target: installed macOS DeepSeek Harness **0.2.0-rc.2**. Desktop Node: 24.18.1;
isolated acceptance runner: bundled primary runtime Node 24.21.0. Original source:
`dd2673/dsh-cross-session-agent` at `f0f7c3d6ab66c66f19472ae33220bad4613724d4`.

## Results

| Check | Result | Evidence boundary |
| --- | --- | --- |
| Regression suite and pack allowlist | **60 passed, 0 failed, 0 skipped** | Installed desktop Electron Node; real rc.2 SessionQuery integration included |
| Fresh native profile acceptance | **8 assertions passed** | Packed final fork installed through the official CLI into a temporary DSH_HOME; real Agent, tool dispatch, SessionQuery and JSONL services |
| Cold A → B → A → B | Passed | Native Agent loops with a deterministic local model adapter; B ends with FIXTURE_APPROVED |
| Native receipts and fork ownership | Passed | Three receipts claimed through the plugin; a real fork excludes its parent's Inbox events |
| Authorization and lifecycle | Passed | Explicit pairs, same cwd, archive and abort gates; cross-cwd and unpaired targets rejected |
| Durable message budget | Passed | Fourth relay denied; restart still denies without waking cold B or calling a model |
| Deferred cancellation/archive/resume races | Passed | Controlled asynchronous regression fixtures, including fresh-handle disposal and publication commit validation |
| Main desktop installation | **Installed and running** | Official CLI, scripts disabled; runtime files byte-identical to the fork; profile's existing plugins preserved |
| Main desktop online-model loop | **Passed** | Two newly created synthetic Sessions using deepseek-account / deepseek-flash; five successful native tool calls; A ends with MAIN_A_FIXED and B with MAIN_APPROVED |
| Main desktop transport evidence | **3 of 3 claimed** | Public SessionQuery over exact UI-copied test IDs and native Inbox events; no extra model turn is inferred |
| rc.2 relay rendering and sender navigation | Passed | Native desktop notice shows sender title and literal message; keyboard activation of its sender link opens the correct Session through uiWorkspace.openSession |
| Header Session ID copying | Passed on main desktop | The actual clipboard was pasted into the unsent test composer and matched each new test Session ID |
| Sidebar Session ID menu | Menu visible; copying passed behavioral regression | Public workspace menu slot is present on main desktop. Regression copies the menu row's full ID independently of the active chat and closes the menu. Final main-desktop clipboard read-back remains pending because computer-use input stopped responding |
| Dependency audit | **0 known advisories** | pnpm audit over 59 lockfile dependencies, including development dependencies |
| Runtime static review | No hidden network, telemetry, credential-file reads, shell/eval or independent mailbox found | Only the three lib runtime files are packed; acceptance scripts and local profiles are excluded |

The real desktop loop exercised the Host implementation retained unchanged in
0.1.3; later changes corrected the rc.2 renderer, sender navigation and sidebar
copy action. Package metadata explicitly loads the public UI Workspace owner.

The main profile installer reports missing npm peers for Cordis, Schemastery,
LLM and Tools: the desktop supplies those core packages through its runtime
resolver. The real model's successful tool dispatch establishes that the Host
resolved and executed them. No duplicate core packages were added to the profile.

The installed app.asar SHA-256 remained
`18d5036bbb58eaff41d351ede99314b8d06505aec06fb2e164c49874ada30f9e`.
No main application restart or core patch was needed. Existing personal Session
content was not used for messaging; the online test had a dedicated workspace
and two new synthetic Sessions. Exact native logs and personal profile backups
are kept locally and are not included in this repository or npm package.

## Reproduce the isolated acceptance

```sh
pnpm install --frozen-lockfile --ignore-scripts
pnpm test
node scripts/live-profile-e2e.mjs /path/to/dsh
```

The runner creates a fresh temporary home, packs and installs the plugin with
scripts disabled, disables HMR only in that test profile, and uses uncompressed
native JSONL to make the evidence inspectable. It boots the profile, gracefully
stops its own process and restarts over the same logs. Its deterministic model
adapter has no online credentials or requests. It never connects to an existing
server or submits to a personal Session. Native transport, lifecycle and
persistence services come from the target CLI.

For desktop reproduction, create two same-workspace root Sessions, copy their
full IDs, configure one pair with maxMessages=3, and give A and B explicit
review/fix/re-review duties. A sends REVIEW_1, B returns FIX_1, A sends REVIEW_2,
and B reports approval without another relay. Verify native tool results,
message IDs, sources and Inbox claims. Transport claims do not establish
business approval; B's explicit final result does.

## Use boundary

This revision is usable for explicitly configured same-workspace root/fork
pairs on the tested host version. It defaults to no authority. Give both
Sessions finite orchestration duties and a finite maxMessages budget. Remove
the pair when the task ends. The receiver's own Harness permissions still apply.

The installed synthetic test pair has already spent its three-message budget.
For actual work, configure the intended Session IDs and a new coordination id;
the model cannot change that configuration or reset its budget.

Readable user/assistant/compact text is not scanned for secrets. Cold resume can
continue pre-existing Inbox work. Already accepted messages are not retracted
by a later sender stop. Arbitrary model behavior, other DSH releases, future
source updates and undisclosed dependency vulnerabilities are outside this
acceptance.
