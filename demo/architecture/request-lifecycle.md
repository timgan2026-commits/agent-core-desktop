# Request lifecycle

How a single goal moves through the system, end to end.

```mermaid
sequenceDiagram
    participant User
    participant UI as Control Center / API
    participant Planner
    participant Permission as Permission Engine
    participant Tool as Tool (filesystem / browser / ...)
    participant Verifier
    participant Log as Activity Stream

    User->>UI: submit goal (plain language)
    UI->>Planner: normalize goal, build plan
    Planner-->>UI: plan + declared success criteria
    UI->>Log: PLAN_CREATED

    loop each step
        UI->>Permission: classify(tool, params)
        alt ALLOW
            Permission-->>Tool: execute
        else ASK
            Permission-->>User: wait for approval
            User-->>Permission: approve / deny / timeout(=deny)
            Permission-->>Tool: execute (only if approved)
        else DENY
            Permission-->>UI: refused, never executes
        end
        Tool-->>UI: step result
        UI->>Log: TOOL_FINISHED
    end

    UI->>Verifier: check independent evidence
    Verifier-->>UI: VERIFIED / NOT_VERIFIED / INCONCLUSIVE
    UI->>Log: VERIFICATION_PASSED | VERIFICATION_FAILED
    UI-->>User: final status (COMPLETED only if verified)
```

## Run status lifecycle

```mermaid
stateDiagram-v2
    [*] --> QUEUED
    QUEUED --> RUNNING
    RUNNING --> BLOCKED: needs clarification or approval
    BLOCKED --> RUNNING: human answers / approves
    RUNNING --> PAUSED: human takeover requested
    PAUSED --> RUNNING: control released
    RUNNING --> COMPLETED: evidence-verified success
    RUNNING --> UNVERIFIED: finished, but evidence inconclusive
    RUNNING --> FAILED: a step failed, or verification found a real gap
    RUNNING --> MODEL_ERROR: configured model unreachable/unusable
    RUNNING --> CANCELLED: cancellation requested
    COMPLETED --> [*]
    FAILED --> [*]
    UNVERIFIED --> [*]
    MODEL_ERROR --> [*]
    CANCELLED --> [*]
```

`COMPLETED` is only reached through verified evidence — there is no path
from `RUNNING` straight to `COMPLETED` that skips the Verifier.

![A real BLOCKED run: the offline planner honestly reports it has no rule for an ambiguous request, with the specific reason shown](../screenshots/06-blocked-resume.png)

A `BLOCKED` run isn't a dead end — it carries the specific reason
clarification is needed, and answering it (via chat or
`POST /autonomous-runs/:id/answer`) resumes the same run rather than
starting a new one.
