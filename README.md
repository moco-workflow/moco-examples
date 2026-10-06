# Moco examples

A collection of runnable workflow specs (wfspecs) showing how to build workflows with Moco.
Each file is self-contained: copy it, run it, and change it to suit your own use case.

| Folder | What you'll find |
|---|---|
| [`basics/`](basics/README.md) | The workflow language one feature at a time — tasks, data flow, expressions, conditions, loops, parallelism, events, error handling. Numbered as a learning path. **Start here** |
| [`activities/`](activities/README.md) | One spec per activity family (HTTP, SQL, Kafka, shell, Kubernetes, LLMs, MCP, browsers, state, secrets, and more), showing the inputs each activity takes and the output it returns |
| [`state-machine/`](state-machine/README.md) | `state_machine` workflows: automatic, conditional, internal and wildcard transitions, callback order, timers, long-running machines with checkpoints, and human-in-the-loop approval |
| [`rules-engine/`](rules-engine/README.md) | `rules_engine` workflows: facts, rule chaining, and live sessions that keep evaluating as new facts arrive |

Each folder has its own README with a table of the specs it contains and what each one teaches.

To try an example without installing anything, try it in the
[Moco Playground](https://www.moco-workflow.com/apps/playground).

## Before you start

You need the `moco` CLI, logged in to a Moco server:

```bash
moco login
```

See [Use the Moco CLI](../moco-doc/docs/guides/use-moco-cli.md) for installation and setup.

## Run an example

```bash
moco run moco-examples/basics/01-single-activity.yaml
```

Workflows run on the Temporal runtime by default. Add `--in-memory` to run in-process instead,
which is handy for quick experiments:

```bash
moco run moco-examples/basics/01-single-activity.yaml --in-memory
```

Most examples declare their inputs with sensible defaults. Override any of them with `-i`:

```bash
moco run moco-examples/basics/08-condition.yaml -i '{"age": 15, "country": "FR"}'
```

A variable whose name ends in `#` streams its value to the terminal as it is evaluated. The
examples use this throughout, so you can watch data move through a workflow as it runs.

## Validate an example

Check a spec against the schema without running it:

```bash
moco validate moco-examples/basics/08-condition.yaml
```

## Things to know

- **Network access.** Some examples call public endpoints such as https://httpbin.org. Behind a
  TLS-inspecting proxy, add `"skip_cert_verify": true` to their input.
- **External systems.** Activity examples that talk to a database, message broker, cluster or
  LLM take their endpoints, secret names and credentials as inputs, so `-i '{...}'` points them
  at your own infrastructure. The header comment of each file says what it needs.
- **Long-running and interactive workflows.** Examples that wait for a person or an outside
  feed (for example `state-machine/04-human-in-the-loop.yaml` or
  `rules-engine/03-live-rules-engine.yaml`) ship with a simulated counterpart so they finish
  on their own. Their folder README shows how to start them with `moco start` and drive them
  yourself.
- **Embedded workflows.** Some examples carry a child workflow inline (`workflow.contents`) or
  as a string in `context`, to be launched with `builtin.execute_workflow` or published with
  `builtin.deploy.*`. Those embedded specs use the same language as the files around them.

## Learn more

- [Writing workflows](../moco-doc/docs/guides/writing-workflows.md)
- [WorkflowSpec reference](../moco-doc/docs/reference/workflowspec-reference.md)
- [Activity catalog](../moco-doc/docs/reference/activity-catalog.md)
- [State machines](../moco-doc/docs/concepts/state-machines.md) and
  [rules engine](../moco-doc/docs/concepts/rules-engine.md)
- [Testing workflows](../moco-doc/docs/guides/testing.md)
