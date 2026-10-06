# state-machine

`state_machine` from the smallest useful machine to durable and human-driven ones.

```bash
moco run 01-simple-state-machine.yaml
moco run 04-human-in-the-loop.yaml
```

| Spec | Teaches | Needs |
|---|---|---|
| [01-simple-state-machine](01-simple-state-machine.yaml) | states, `on_enter`/`on_exit`, terminal states, automatic (`trigger: {}`) and conditional transitions | — |
| [02-continue-as-new](02-continue-as-new.yaml) | `checkpoint_policy.event_count`: continue-as-new after N events, resume in the same state; unnamed state `timeout_sec` → `sys.timer` | Temporal |
| [03-continue-as-new-timer](03-continue-as-new-timer.yaml) | `checkpoint_policy.timeout_sec` plus a named state timer (`sys.timer.<name>`); ~20s run, ~3 checkpoints | Temporal |
| [04-human-in-the-loop](04-human-in-the-loop.yaml) | expense approval that waits for a person: event-driven transitions, reading `event["data"]`, internal transitions, reminder timer with `max_timeout_attempts` → `.final` escalation | — |
| [05-send-decision](05-send-decision.yaml) | the human's side: `emit_event` with `target_workflow_id` into a running machine | a running 04 |

`02` and `03` need the Temporal runtime, because continue-as-new is a Temporal feature.

## Human in the loop, for real

`04` ships with a simulated approver so it finishes on its own. To be the approver yourself:

```bash
moco start 04-human-in-the-loop.yaml -i '{"simulate_approver": false}'   # prints a workflow id
moco run 05-send-decision.yaml -i '{"workflow_id": "<id>", "event_type": "add_comment", "comment": "receipt?"}'
moco run 05-send-decision.yaml -i '{"workflow_id": "<id>", "event_type": "approve"}'   # or reject
moco status <id>
```

If nobody decides, the machine sends 3 reminders, one every `reminder_sec` (30s by default), and
then ends in `expired`.
