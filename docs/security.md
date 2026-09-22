# Security model

This describes Agent Core's security model at the policy level — what
guarantees exist and why. It intentionally does not publish the exact
classification rules, pattern lists, or thresholds the private core uses
internally; publishing that level of detail would mainly help someone
craft a bypass, not help a legitimate integrator or evaluator.

## One gate, every action

```
REQUEST  →  CLASSIFY  →  POLICY  →  ALLOW / ASK / DENY  →  EXECUTE
```

Every tool invocation — filesystem, terminal, git, browser, Computer Use,
MCP — passes through a single permission decision before it runs. There
is no second, lighter-weight path for API-initiated or automated actions;
the same gate applies whether the request came from an interactive chat
session, an autonomous run, or a direct API call.

- **ALLOW** — runs automatically. Reserved for actions with no meaningful
  blast radius if wrong: reading files, running tests, checking status.
- **ASK** — requires explicit human approval before running. Reserved for
  actions that are hard to reverse or reach outside the immediate
  workspace: deleting data, force-pushing, publishing something publicly,
  touching production-shaped infrastructure.
- **DENY** — never executes, under any configuration, in any autonomy
  mode. Reserved for actions that are almost never legitimate in this
  context: attempts to disable security controls, obviously destructive
  system-level commands, credential theft.

No autonomy mode — including the most permissive one — can turn a `DENY`
into something that runs, or an `ASK` into something that skips approval.
The only thing autonomy mode changes is how much of the normally-ALLOW
surface still pauses for a human before starting at all.

## Human-in-the-loop, for real

An `ASK`-classified action blocks execution until a human explicitly
approves or denies it. If nothing is watching (a non-interactive or
unattended run), the safe default is **deny**, never silent approval — a
request that times out waiting for a human is treated exactly like an
explicit denial.

Cancellation is real: requesting cancellation of a running task interrupts
it at the next safe checkpoint, not just at the very end. A human can also
take over a running autonomous task mid-execution and hand control back
later.

## Secrets

- Agent Core never asks for or stores account passwords, payment details,
  or similar credentials as part of its own configuration beyond the
  provider API keys you explicitly set (e.g. for a cloud model) — those
  are read from environment configuration only, never hardcoded, never
  included in a prompt sent to a model, and never written to persisted
  memory.
- Content that *looks like* a credential (API-key-shaped, token-shaped,
  private-key-shaped) is refused before it can be written to long-term
  memory, and is scanned for before anything is published somewhere a
  third party could read it (an issue, a pull request, a comment).
- Logs and the live activity stream carry sizes, durations, and outcomes
  for model calls — never prompt or response text, and never an
  environment variable's value.

## Local vs. cloud model boundary

Agent Core runs fully offline by default. A local model (via Ollama) never
sends your task, your code, or your files anywhere off your machine.
Switching to a cloud model (Anthropic Claude) is an explicit, visible
configuration choice — never a silent, automatic fallback. If cloud usage
is ever offered as an automatic suggestion (only under an explicit
opt-in policy, and only when local hardware genuinely can't run anything
reasonable), applying it still requires an explicit action — it is never
selected on your behalf.

## Filesystem & process sandboxing

Filesystem and terminal (and git, by working directory) actions are
confined to a configured workspace root. A request to read, write, or
execute outside that root — including via path traversal or a symlink
escape — is refused. An optional container-isolation execution backend
adds a further layer (no host environment variables forwarded by default,
network disabled by default) for deployments that want it.

## Desktop app hardening

The packaged desktop application uses standard Electron hardening
(context isolation, no direct Node integration exposed to the UI, a
minimal preload surface). Remote debugging is available only in
development builds, behind an explicit opt-in environment variable — the
packaged, distributed build never exposes it. Process management is
scoped to the application's own child process; nothing else on the
machine is touched.

## Bounded, everywhere

Every autonomous or long-running execution path is bounded: how many
iterations it can take, how long it can run in wall-clock time, how long
any single command can run before timing out, and how many tool calls a
multi-agent fan-out can spawn. A run that would otherwise loop or run
away instead stops and reports what happened.

## Verification before "success"

A claimed success is checked, not trusted. See
[Capabilities → Verification](capabilities.md#verification) for what that
means in practice, and [demo/workflows/demo-6-failure-recovery.md](../demo/workflows/demo-6-failure-recovery.md)
for a concrete example of a verification failure being reported honestly
instead of hidden.

---

Have a specific security question for an evaluation or integration? See
[Commercial / Partnership](../README.md#commercial--partnership).
