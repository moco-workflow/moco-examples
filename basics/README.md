# basics (DSL v2)

The Moco DSL v2 one feature at a time: tasks, control flow, expressions and errors.
Numbered as a learning path. The DSL itself is specified in
[`moco-core/docs/dsl-v2`](../../moco-core/docs/dsl-v2/design.md).

Unlike the rest of `moco-examples-v2`, these files are **written by hand** to teach v2 idioms.
The v1 originals are in [`moco-examples/basics`](../../moco-examples/basics), and
`make convert-examples-v2` does not touch this folder.

No engine runs v2 yet. Check the files with the reference validator:

```bash
python moco-core/docs/dsl-v2/validate.py moco-examples-v2/basics
```

To run one today, lower it to v1 first. The converter reports anything v1 cannot express:

```bash
python -m scripts.dsl_convert moco-examples-v2/basics/08-condition.yaml -o /tmp/08-condition.v1.yaml
moco run /tmp/08-condition.v1.yaml --in-memory -i '{"age": 15, "country": "FR"}'
```

A variable whose name ends in `#` streams its value to the CLI when it is evaluated — the main
debugging tool, used throughout.

| Spec | Teaches |
|---|---|
| [01-single-activity](01-single-activity.yaml) | the minimal workflow: `header`, one task, `output.data` + `output.as` |
| [02-set](02-set.yaml) | `set`: ordered assignments, `_name` task-locals, `__user_info__`, `now()` |
| [03-tasks](03-tasks.yaml) | the `tasks` container, carrying a variable across steps |
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

## Lowering to v1

Most of these lower to v1 exactly. When run on the v1 engine, the lowered specs produce the
results the comments describe. Two do not:

- **11-raise:** `continue` from inside the nested `checks` group has no exact v1 form (W240).
  On v1 it behaves like `exit`.
- **16-error-handling:** v1 has no `on_error`, so the handlers are dropped (W210). The
  default inputs then fail at "card declined"; `-i '{"fail_payment": false}'` runs the
  success path.
