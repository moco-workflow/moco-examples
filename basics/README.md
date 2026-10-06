# basics (DSL v2)

The Moco DSL v2 one feature at a time: tasks, control flow, expressions and errors.
Numbered as a learning path. The DSL itself is specified in
[`moco-core/docs/dsl-v2`](../../moco-core/docs/dsl-v2/design.md).

These files are **written by hand** to teach v2 idioms, and use only v2 names, comments
included (`scripts/dsl_convert/tests/test_curated_basics.py` checks this).

Run one (the default runtime is Temporal). 01 and 24 call https://httpbin.org; behind a
TLS-inspecting proxy add `"skip_cert_verify": true` to their input:

```bash
moco run moco-examples/basics/08-condition.yaml -i '{"age": 15, "country": "FR"}'
```

Check the files without running them:

```bash
python moco-core/docs/dsl-v2/validate.py moco-examples/basics
```

A variable whose name ends in `#` streams its value to the CLI when it is evaluated — the main
debugging tool, used throughout.

| Spec | Teaches |
|---|---|
| [01-single-activity](01-single-activity.yaml) | the minimal workflow: `header`, one `http.request` task (httpbin.org), `output.data` + `output.as` |
| [02-set](02-set.yaml) | `set`: ordered assignments, `_name` task-locals, `__user_info__`, `now()` |
| [03-multi-step-tasks](03-multi-step-tasks.yaml) | the `tasks` container, carrying a variable across steps |
| [04-data-flow](04-data-flow.yaml) | `context` vs `input` vs task-locals; `output.set` / `data` / `as`; `_`; no shadowing |
| [05-expression](05-expression.yaml) | reference sheet: builtins, comprehensions, every name modifier, `__context__` |
| [06-variable-modifier](06-variable-modifier.yaml) | the same modifiers as a narrative, over a parallel `loop` |
| [07-container-scope-variable](07-container-scope-variable.yaml) | `@` container scope in depth; per-iteration scopes |
| [08-condition](08-condition.yaml) | `skip_if`, `branches` (if / elif / else), `switch`, `{and:}` / `{or:}` / `{not:}` |
| [09-loop](09-loop.yaml) | `loop`: `foreach`, `index`, sequential vs parallel, nesting, collecting results |
| [10-parallel](10-parallel.yaml) | `tasks` with `props.execute_mode: parallel` and `join_type` |
| [11-raise](11-raise.yaml) | every `raise` type: `error`, `terminate`, `exit`, `continue`, `break` |
| [12-functions](12-functions.yaml) | `functions:` called with `workflow.function` |
| [13-child-workflow-modes](13-child-workflow-modes.yaml) | `workflow` sources (`ref`, `contents`, ...) and every `mode` |
| [14-data-resolver](14-data-resolver.yaml) | child workflow as data resolver; a loop of calls; `contents_yaml` picked at runtime |
| [15-events](15-events.yaml) | `emit_event` + `wait_for`, `match_expression`, timeout gives `null`, `async_mode` |
| [16-error-handling](16-error-handling.yaml) | retry policies, `on_error` (catch and cleanup), handling only expected errors |
| [17-async-activity](17-async-activity.yaml) | `options.async_mode`: the token, unique per run; overlapping work; collecting results in any order |
| [18-async-activity-state-machine](18-async-activity-state-machine.yaml) | a `state_machine` whose transition fires on an async activity's completion |
| [19-branches](19-branches.yaml) | `branches` in depth: order matters, no `else` gives `null`, a `tasks` container as `do`, nesting |
| [20-switch](20-switch.yaml) | `switch` in depth: scalar vs list `match`, `match` as an expression, typed values, no `default`, routing in a loop |
| [21-race](21-race.yaml) | `join_type: any` on `tasks` and `loop`; racing a deadline; why local activities do not race on Temporal |
| [22-sleep](22-sleep.yaml) | `wait_for` with no event as a durable timer; a poll loop with `raise: break` |
| [23-checkpoint](23-checkpoint.yaml) | `checkpoint` (continue-as-new): writing `main` so it resumes from the context |
| [24-task-fields](24-task-fields.yaml) | the fields every task shares: `task_name`, `skip_if`, `output`, `on_error`, and the order they run in |

## Lowering to v1

To run one on the v1 engine, lower it first. The converter reports anything v1 cannot express:

```bash
python -m scripts.dsl_convert moco-examples/basics/08-condition.yaml -o /tmp/08-condition.v1.yaml
```

Most of these lower to v1 exactly, and the lowered specs produce the results the comments
describe. These do not:

- **11-raise:** `continue` from inside the nested `checks` group has no exact v1 form (W240).
  On v1 it behaves like `exit`.
- **16-error-handling:** v1 has no `on_error`, so the handlers are dropped (W210). The
  default inputs then fail at "card declined"; `-i '{"fail_payment": false}'` runs the
  success path.
- **19-branches, 20-switch:** a `branches` / `switch` result has no v1 form; the lowered task
  returns the list of branch results instead (W251), so the `output.as` values differ.
- **21-race:** the all-fail race loses its `on_error` (W210) and fails the workflow.
- **24-task-fields:** the `on_error` handlers are dropped (W210), so the default inputs fail
  at the 503; `-i '{"status_code": 200}'` gets further, to the failing `output` of `read-title`.
