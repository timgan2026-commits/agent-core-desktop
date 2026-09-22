# Screenshots

Real captures of the actual Control Center UI, running locally against a
real backend with the default MOCK provider (no API key, no local model
required — fully reproducible by anyone). Nothing here is a mockup.

| File | Shows |
|---|---|
| [01-main-ui.png](01-main-ui.png) | Main Chat view — new conversation, autonomy mode selector, model badge |
| [02-hardware-model-selection.png](02-hardware-model-selection.png) | Settings → Hardware & recommended model — real detected CPU/RAM/GPU/VRAM, installed models, and a live recommendation with its reasoning |
| [03-run-activity-feed.png](03-run-activity-feed.png) | A completed autonomous run: goal, plan, security activity log, artifacts, completion proof, and live events feed |
| [04-evidence-verification.png](04-evidence-verification.png) | Close-up of the same run's success criteria — each with a PASS/FAIL badge and the actual evidence string, not just a checkmark |
| [05-ask-approval.png](05-ask-approval.png) | Approval Center — a real pending `ASK`-classified action (creating a file under `manual` autonomy mode), with Approve/Deny |
| [06-blocked-resume.png](06-blocked-resume.png) | A `BLOCKED` run — the offline planner honestly reporting it has no rule for an ambiguous request, with the real "requires clarification" reason shown |
| [07-failure-recovery.png](07-failure-recovery.png) | A real `FAILED` run (deleting a file that doesn't exist) — the success criterion stays `PENDING` (never falsely marked satisfied) and the actual error is shown verbatim |

## How these were produced

1. `python main.py serve` with a clean, empty workspace directory (no
   personal paths or data — the workspace root visible in screenshot 7's
   error text is a throwaway demo directory, not a real user path).
2. The React Control Center (`npm run dev`) driven through a handful of
   real API calls (`POST /autonomous-runs`, `POST /approvals/:id/approve`)
   to reach each state shown.
3. Captured with `html2canvas` against the live DOM and saved as real PNG
   files — not a screenshot tool overlay, not a mockup.

## Not included

A short screen-recording/GIF walking through a full run end-to-end (goal
→ plan → execute → verify) would be a good addition but was not produced
in this pass — a static walkthrough of the same flow is in
[demo-1-simple-task.md](../workflows/demo-1-simple-task.md).
