# state-machine

`state_machine` from the smallest useful machine to durable and human-driven ones. The feature
set is documented in moco-doc's [State Machines](../../moco-doc/docs/concepts/state-machines.md).

```bash
moco run 01-simple-state-machine.yaml
moco run 04-human-in-the-loop.yaml
```

| Spec | Teaches | Needs |
|---|---|---|
| [01-simple-state-machine](01-simple-state-machine.yaml) | states, `on_enter`/`on_exit`, terminal states, automatic (`trigger: {}`) and conditional transitions | — |
| [02-continue-as-new](02-continue-as-new.yaml) | `checkpoint_policy.event_count`: continue-as-new after N events, resume in the same state; unnamed state `timeout_sec` → `sys.timer` | Temporal |
| [03-continue-as-new-timer](03-continue-as-new-timer.yaml) | `checkpoint_policy.timeout_sec` plus a named state timer (`sys.timer.<name>`); ~20s run, ~3 checkpoints | Temporal |
| [04-human-in-the-loop](04-human-in-the-loop.yaml) | expense approval that waits for a person: event-driven transitions, reading `event["data"]`, internal transitions, reminder timer with `max_timeout_attempts` → `.final` escalation. Its companion [04-human-in-the-loop.send-decision](04-human-in-the-loop.send-decision.yaml) is not a machine: it is the human's side, `emit_event` with `target_workflow_id` into a running 04 | — |
| [05-self-driven-order](05-self-driven-order.yaml) | work in `on_enter`, emit the outcome to yourself; one `event_type`, two transitions told apart by a `condition` on `event["data"]`; `trigger.action` copying the payload | network |
| [06-global-transitions](06-global-transitions.yaml) | wildcard transitions (no `from_state`), a state-specific transition winning over the wildcard, `from_state` lists, events handled in emit order | — |
| [07-callback-order](07-callback-order.yaml) | `on_exit` → `action` → `on_enter`; internal transitions skip callbacks and keep timers; self-transitions re-enter; `current_state` inside an action | — |
| [08-timers](08-timers.yaml) | several named timers in one state routed differently, the unnamed timer's `sys.timer.final`, the machine-wide `timeout_sec` caught with `on_error` | — |

`02` and `03` need the Temporal runtime, because continue-as-new is a Temporal feature. `05`
calls https://httpbin.org; behind a TLS-inspecting proxy add `"skip_cert_verify": true` to its
input.

Triggering a transition on an **async activity's completion** (an expression trigger on the
activity's token) is shown in
[basics/18-async-activity-state-machine](../basics/18-async-activity-state-machine.yaml).

## Human in the loop, for real

`04` ships with a simulated approver so it finishes on its own. To be the approver yourself:

```bash
moco start 04-human-in-the-loop.yaml -i '{"simulate_approver": false}'   # prints a workflow id
moco run 04-human-in-the-loop.send-decision.yaml -i '{"workflow_id": "<id>", "event_type": "add_comment", "comment": "receipt?"}'
moco run 04-human-in-the-loop.send-decision.yaml -i '{"workflow_id": "<id>", "event_type": "approve"}'   # or reject
moco status <id>
```

If nobody decides, the machine sends 3 reminders, one every `reminder_sec` (30s by default), and
then ends in `expired`.
