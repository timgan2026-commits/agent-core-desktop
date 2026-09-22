# Capabilities

Legend: **✅ Verified** (independently exercised with real evidence, not
just unit-tested in isolation) · **⚠️ Partial** (implemented, but at
least one meaningful part is unverified or has a known limitation) ·
**🧭 Planned** (not yet built).

This list is deliberately conservative. A capability that hasn't actually
been exercised end-to-end is never marked verified here, even if the code
for it exists and passes unit tests.

## Planning & models

| Capability | Status | Notes |
|---|---|---|
| Offline, deterministic planning | ✅ | No model, no API key, no network — the default. Fully reproducible. |
| Local model planning (Ollama) | ✅ | Verified against a real, running local model across multiple task types (simple, tool-using, multi-step). Open-ended analytical tasks remain harder for smaller local models — a known, disclosed limitation, not hidden. |
| Cloud model planning (Anthropic Claude) | ⚠️ | Implemented and covered by tests using a mocked provider; not independently re-verified against the live Anthropic API in the most recent internal verification pass (no API key was configured during that pass). |
| Hardware-aware local model selection | ✅ | Detects CPU/RAM/GPU/VRAM, recommends a model with a documented safety margin (never right at the memory edge), and correctly declines to recommend an installed model that's too close to that edge. Verified live against real hardware. |
| Model provider failure handling | ✅ | A configured real model that becomes unreachable produces an explicit, typed error state — never a silent fallback to a different mode, unless that fallback is explicitly opted into. |

## Execution

| Capability | Status | Notes |
|---|---|---|
| Filesystem tools (read/write/delete/search) | ✅ | Sandboxed to a configured workspace root; path traversal outside it is refused. |
| Terminal / shell execution | ✅ | Bounded by a per-command timeout; destructive command patterns are classified and gated (see [Security](security.md)). |
| Git operations | ✅ | Status/diff/add/commit/branch/log unrestricted; push/reset are approval-gated. |
| GitHub integration | ✅ | Issues/PRs via a real REST client or an offline mock provider; outbound content is scanned for secret-shaped values before publishing. |
| Browser automation | ✅ | Real end-to-end verification: navigate, click, type, extract, screenshot, multi-tab, file download, persistent login profiles. |
| Computer Use (real mouse/keyboard/screen control) | ⚠️ | Off by default. The real desktop-control backend builds and is unit-tested against real automation libraries; a live, interactive execution smoke test has not been re-verified in the most recent internal pass (a permission prompt for exclusive control of test hardware was declined in that session — noted honestly rather than skipped over). |
| MCP (Model Context Protocol) tools | ⚠️ | A working stdio MCP client exists; remote/HTTP MCP servers are not yet supported. |
| Docker execution isolation | ⚠️ | Implemented; not verified against a live Docker daemon in the most recent internal pass. |

## Autonomy & safety

| Capability | Status | Notes |
|---|---|---|
| ALLOW / ASK / DENY permission gate | ✅ | Every tool call, in every autonomy mode, passes through one policy decision. `DENY` cannot be executed under any mode; `ASK` cannot be silently skipped. |
| Autonomy modes (manual → autopilot) | ✅ | From "confirm everything" to "auto-run safe actions, still ask before risky ones" — no mode grants DENY execution or skips ASK. |
| Human-in-the-loop approvals | ✅ | An ASK-classified action blocks until a human explicitly approves or denies (or a timeout elapses — timeout is treated as denied, never as approved). |
| Real cancellation | ✅ | Cancelling a running task interrupts it at the next safe checkpoint — not just a status flag that gets ignored. |
| Human takeover | ✅ | Autonomous execution can be paused and handed to a human mid-run, then resumed. |
| Bounded execution | ✅ | Iteration count, wall-clock time, per-command timeout, and tool-call budgets all cap how far a run can go before it stops and reports rather than looping indefinitely. |

## Verification

| Capability | Status | Notes |
|---|---|---|
| Evidence-based completion | ✅ | A step is only "completed" if independent evidence (file read back, exit code inspected, test/lint/build result) backs it — not the model's own claim. |
| Adversarial verification testing | ✅ | Deliberately tested against a tool that lies about success, a file modified outside the tracked tool path, and a tool that crashes mid-run — verification correctly reports each as not-completed rather than false-positive. |
| Checkpoint / resume | ✅ | A run interrupted mid-execution (process restart, crash) resumes from its last checkpoint with prior evidence intact — verified across a simulated process restart. |

## Interface

| Capability | Status | Notes |
|---|---|---|
| Web control center (React UI) | ✅ | Chat, run history, live activity stream, approvals, settings — all backed by the real HTTP API, no mocked/demo-only UI state. |
| HTTP API | ✅ | The UI is one client of this API; the same API is usable directly (see [examples/](../examples/)). |
| Live activity stream | ✅ | Server-Sent Events with automatic reconnection and gap detection — the UI never silently shows stale state as current. |
| Desktop packaging (Windows) | ⚠️ | A packaged installer builds, installs, and runs correctly — verified through the packaged app's own live API, real files on disk, and a full crash/restart test suite. Interactive mouse/keyboard testing of that *exact packaged build's UI* is still pending (see [Roadmap](roadmap.md)). |
| macOS / Linux packaging | 🧭 | Not yet built. |
| Chat history durability | ⚠️ | Currently stored client-side; a durable, server-backed history store is designed but not yet implemented. |

## What we don't claim

- No tokens-per-second or latency benchmark numbers are published anywhere
  in this showcase — model performance estimates that do exist internally
  are explicitly labeled as estimates, not measurements.
- No claim that Computer Use has been exercised against real, arbitrary
  desktop applications beyond what's noted above.
- No claim of multi-platform support beyond Windows today.
