# Demo 5 — Computer Use (real desktop control)

> **Honesty note.** Computer Use is off by default and is Agent Core's
> highest-blast-radius capability (it can reach the whole desktop, not a
> sandboxed workspace). This scenario describes the *designed* behavior.
> The real automation backend builds correctly against real
> hardware-automation libraries and is covered by unit tests for its risk
> classification; a live, interactive mouse/keyboard/screen execution
> smoke test has not been independently re-verified in the most recent
> internal pass — see [Capabilities](../../docs/capabilities.md). We are
> not claiming this scenario has been demonstrated end-to-end on real
> hardware in that pass, and we're saying so directly rather than
> blurring the distinction.

## Goal

> "Open the system's text editor, type a short note, and save it to the
> desktop."

## Designed steps

1. Computer Use must be explicitly enabled (it is not registered as an
   available tool at all otherwise) — this alone is a config-level,
   visible opt-in.
2. Every action — moving the mouse, clicking, typing, taking a screenshot,
   focusing a window — is classified before it runs, exactly like every
   other tool.
3. An action with no clear target (an ambiguous click or drag) is `ASK`,
   not assumed-safe — the absence of information is treated as a reason
   to pause, not to guess.
4. Typing content that looks like a secret (password/token-shaped) is
   `ASK` even with no other risk signal present.
5. System-level shortcuts that could escape the intended sandbox (e.g. a
   "run" dialog shortcut) are `DENY` outright, not `ASK` — there's no
   approval path that allows them.
6. Pause / take-control / **stop** are available at any point — a human
   can freeze all agent-initiated desktop actions immediately, take
   manual control, and hand it back later.

## Expected result (when live-verified)

- The editor opens, the note is typed, and the file is saved to the
  reported location.
- The activity stream shows each individual action (not just "task done")
  with its own risk classification and outcome.
- A screenshot-based observation step confirms the end state, rather than
  assuming success from the last action alone.

## Evidence

- Risk classification for Computer Use actions is covered by dedicated
  automated tests (ambiguous-target handling, secret-shaped text,
  sandbox-escape shortcuts) — these pass today.
- Browser automation (a related but distinct, lower-blast-radius
  capability) **has** been verified end-to-end on real infrastructure —
  see [Demo 3](demo-3-browser-workflow.md) — which is why that one is
  marked ✅ in Capabilities while Computer Use is marked ⚠️.
