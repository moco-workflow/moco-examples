# rules-engine

`rules_engine`: a one-shot evaluation over a fixed set of facts, and a live session that keeps
evaluating as facts stream in. All of them run on both runtimes.

```bash
moco run 01-simple-rules-engine.yaml
moco run 03-live-rules-engine.yaml
```

| Spec | Teaches |
|---|---|
| [01-simple-rules-engine](01-simple-rules-engine.yaml) | `if` (`with_facts` + `expression`) / `then` (`set_facts`), fact chaining, forward vs backward, `terminate_facts` |
| [02-loan-approval](02-loan-approval.yaml) | a larger rule set classifying an applicant, feeding a scoring `transform` |
| [03-live-rules-engine](03-live-rules-engine.yaml) | `keep_alive: true`: a long-lived session fed `set_facts` events on `fact_source_topic`, ending on `terminate_facts` or `timeout_sec`; rule `actions` |
| [04-send-facts](04-send-facts.yaml) | the producer side: push facts into a running session with `emit_event` |

## Feeding the live engine yourself

`03` ships with a simulated metrics feed so it finishes on its own. To push facts yourself:

```bash
moco start 03-live-rules-engine.yaml -i '{"simulate_feed": false}'   # prints a workflow id
moco run 04-send-facts.yaml -i '{"workflow_id": "<id>", "facts": {"cpu_pct": 97}}'
moco run 04-send-facts.yaml -i '{"workflow_id": "<id>", "facts": {"p99_ms": 2000}}'   # -> incident
moco status <id>
```

## Gotchas (verified against the engine)

- A fact that doesn't exist yet reads as `None`, so `with_facts` does **not** make a rule
  wait. A comparison like `x >= 10` raises on `None`, and a raising expression counts as "no
  match", so comparisons wait anyway. A rule that only tests presence must write
  `x is not None`.
- `set_facts` values are stored as written. `{{ ... }}` there is **not** evaluated.
- Rule `actions` run in the workflow's scope, not the facts'. Reference workflow variables
  only. A NameError on a fact is swallowed into the worker log.
- Derived facts are never retracted, and a live session returns only the derived facts.
