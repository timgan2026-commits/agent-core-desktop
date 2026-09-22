# Demo 4 — Approval workflow (human-in-the-loop)

Shows an action classified as requiring human approval actually stopping
and waiting — not just logging a warning and proceeding.

![A real pending ASK approval in the Approval Center](../screenshots/05-ask-approval.png)

## Goal

> "Delete every file in the `scratch/` folder that hasn't been modified
> in the last 90 days."

## Steps

1. Agent Core plans the deletion. Bulk-deleting files is classified `ASK`
   — a destructive, non-trivially-reversible action.
2. The run pauses. Its status becomes "waiting for approval," not
   "running" and not silently skipped — a client polling or streaming the
   run's state sees this distinction explicitly.
3. A human reviews the pending request — which files, how many, why the
   agent believes they qualify — through the approvals view or
   `GET /approvals`.
4. The human approves (or denies) the specific request via
   `POST /approvals/:id/approve` (or `/deny`).
5. If approved, execution resumes from exactly where it paused — no
   re-planning, no re-running of already-completed steps.
6. If denied, or if no one responds before the approval window elapses,
   the action does **not** run — a timeout is treated as a denial, never
   as an approval.

## Expected result

- Approved path: the qualifying files are deleted; the run completes with
  evidence of exactly which files were removed.
- Denied / timed-out path: no files are deleted; the run ends in a state
  that clearly distinguishes "the agent chose not to" from "a human said
  no" from "no one was there to ask."

## Evidence

- The run's status while waiting is a distinct, real value (not
  overloaded onto "running") — visible identically whether the client is
  polling the REST API or watching the live event stream.
- Cancelling a run while it's mid-execution (not just while waiting for
  approval) interrupts it at the next safe point — a real, tested
  behavior, not just a status flag that a running process ignores.
- The same approval mechanism is what "human takeover" builds on: a human
  can pause an entire autonomous run mid-flight, take manual control, and
  hand it back to the agent later.
