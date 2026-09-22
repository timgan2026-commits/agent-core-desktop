# API examples

Illustrative, non-sensitive examples of Agent Core's HTTP interface. These
describe the request/response *shape* only — not the implementation
behind any endpoint. Run against your own local instance
(`python main.py serve`, default `http://127.0.0.1:8765`, no auth needed
for local/trusted use).

## Submit a goal

```bash
curl -X POST http://127.0.0.1:8765/autonomous-runs \
  -H "Content-Type: application/json" \
  -d '{"goal": "Create a file called hello.txt with content Hello from Agent Core"}'
```

```json
{
  "run_id": "9aef3b164d86",
  "status": "RUNNING",
  "human_takeover": false,
  "live": false,
  "display_status": "running"
}
```

## Poll run status

```bash
curl http://127.0.0.1:8765/autonomous-runs/9aef3b164d86
```

```json
{
  "run_id": "9aef3b164d86",
  "status": "COMPLETED",
  "display_status": "completed",
  "goal": {
    "normalized_goal": "Create a file called hello.txt with content Hello from Agent Core",
    "success_criteria": [
      {
        "description": "File 'hello.txt' exists",
        "satisfied": true,
        "evidence": "filesystem.exists('hello.txt') -> True"
      }
    ]
  },
  "plan": {
    "steps": [
      { "id": 1, "tool": "filesystem", "status": "completed" }
    ]
  }
}
```

## Resolve a pending approval

```bash
# list what's waiting on a human decision
curl http://127.0.0.1:8765/approvals

# approve one
curl -X POST http://127.0.0.1:8765/approvals/<approval_id>/approve

# ...or deny it
curl -X POST http://127.0.0.1:8765/approvals/<approval_id>/deny
```

## Check system status (model mode, provider, capability states)

```bash
curl http://127.0.0.1:8765/status
```

```json
{
  "model_provider": "mock",
  "model_mode": "MOCK",
  "browser_provider": "mock",
  "tasks_by_status": { "COMPLETED": 3 },
  "pending_approvals": 0
}
```

## Live activity stream (Server-Sent Events)

```bash
curl -N http://127.0.0.1:8765/autonomous-runs/<run_id>/stream
```

Streams structured events (`PLAN_CREATED`, `TOOL_STARTED`,
`TOOL_FINISHED`, `VERIFICATION_PASSED`, ...) as they happen — the same
stream the web Control Center's own live activity feed uses.

---

Full endpoint reference for the private core's actual deployment lives in
its own internal documentation — this page covers the interface shape
only, useful for evaluating integration fit.
