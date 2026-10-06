# activities

One runnable wfspec per activity **family**. Between them they call every activity in the catalog.
`builtin.deploy.*` is split three ways: a read-only tour, a publish-and-deploy lifecycle, and
RBAC. The two write specs work only on scratch data they create and delete in the same run. The
header comment in each file says what the spec teaches and what it needs.

These are standalone, run-from-source examples. There is no `moco.json`. Every `wfspec_name`
starts with `act-` so the names don't collide with `moco-examples` if you publish them.

## Quick start

```bash
cd build-my-workflow/moco-examples/activities
moco validate http-request.yaml                 # check a spec against the schema
moco run builtin-state.yaml                     # run it on Temporal
moco run builtin-state.yaml --in-memory         # ...or in-process, no Temporal
moco run openai-chat.yaml -i '{"city": "Oslo"}' # override any input_data default
```

Endpoints, secret names and credentials are all `input_data`, so `-i '{...}'` points a spec at
your own infrastructure. A variable whose name ends in `#` is streamed to the CLI as it is
evaluated, and the specs use this throughout.

## Catalog

| Spec | Activities covered | Needs |
|---|---|---|
| [`builtin-basics`](builtin-basics.yaml) | `builtin.now`, `builtin.delay` | — |
| [`builtin-state`](builtin-state.yaml) | all 9 `builtin.state.*` (set/get/get_with_ts/list/list_namespaces/update_topic/del/delete_by_topic/delete_namespace) | state store |
| [`builtin-secret`](builtin-secret.yaml) | `builtin.secret.list`, `get`; `upload` → `delete` only if you pass a pre-encrypted blob | an existing secret (default `MY_LLM_TOKEN`) |
| [`builtin-execute-workflow`](builtin-execute-workflow.yaml) | `builtin.execute_workflow` (inline content, chosen at runtime, `catch_exception`) | — |
| [`builtin-event`](builtin-event.yaml) | `builtin.event.emit_debug_event`, `emit_metric_event` | — (metrics need `MOCO_METRICS_TOPIC` + Kafka to be seen) |
| [`builtin-deploy`](builtin-deploy.yaml) | read-only tour: namespace, wfspec, package, stage, deployment, query, group, user, auth, audit; `apikey` create→get→lookup→revoke→delete on a scratch key | deployment DB |
| [`builtin-deploy-lifecycle`](builtin-deploy-lifecycle.yaml) | the write side, on scratch data it creates and deletes: namespace/wfspec set+delete, package set/create_with_files/get/get_files/delete, package.file set/get/delete, deployment set/get/activate/deactivate, deploy_package/undeploy_package, target add/list/remove, the `query.*` resolvers, group and user admin | deployment DB |
| [`builtin-deploy-rbac`](builtin-deploy-rbac.yaml) | all `auth.*` writes on scratch data: resource, privilege, role, role privilege and role member; `check_user_privilege` before, during and after a grant made through a group | deployment DB |
| [`authz`](authz.yaml) | all 4 `authz.*` | — |
| [`http-request`](http-request.yaml) | `http.request` (GET/POST json/form, bearer token, 404 as data) | network (`skip_cert_verify: true` behind a TLS-inspecting proxy) |
| [`shell-run`](shell-run.yaml) | `shell.run` | — |
| [`sql`](sql.yaml) | `sql.execute`, `sql.query` | Postgres conn-string secret (default `MOCO_PGVECTOR_CONN`) |
| [`email-send`](email-send.yaml) | `email.send` (dry run by default) | SMTP, only with `dry_run: false` |
| [`gdrive`](gdrive.yaml) | all 6 `gdrive.*` | gdrive creds secret + a shared folder + direct HTTPS to Google |
| [`k8s`](k8s.yaml) | all 8 `k8s.*` on a Deployment the spec creates and deletes | k8s cluster + token/CA secrets |
| [`kafka`](kafka.yaml) | `kafka.publish`, `kafka.consume` | Kafka broker |
| [`rabbit`](rabbit.yaml) | `rabbit.publish`, `rabbit.receive` | RabbitMQ (`MOCO_RMQ_CONNECTION_URL`) |
| [`websocket-subscribe`](websocket-subscribe.yaml) | `websocket.subscribe` (+ a local server started via `shell.run`) | — (or network to your own feed) |
| [`graphql-subscribe`](graphql-subscribe.yaml) | `graphql.subscribe` (+ a local graphql-transport-ws server) | — (or network to your own server) |
| [`mcp-call-tool`](mcp-call-tool.yaml) | `mcp.call_tool`, both blocking and streaming (+ a local FastMCP server) | — (or network to your own server) |
| [`openai-chat`](openai-chat.yaml) | `openai.chat.completions` (plain, JSON mode, tool calling) | LLM token (`MY_LLM_TOKEN`) |
| [`typesafe-system-one`](typesafe-system-one.yaml) | `typesafe.system_one` (noul / choice / score) | TypeSafe or local Laya endpoint |
| [`typesafe-alert-triage`](typesafe-alert-triage.yaml) | `typesafe.system_one` on JSON state: questions built from data, described labels, probability-margin guard → page / ticket / human review | TypeSafe or local Laya endpoint |
| [`claude-agent-query`](claude-agent-query.yaml) | `claude_agent.query` (text only, one Moco tool) | **agent worker** + Anthropic key secret (`MY_CLAUDE_APIKEY`) |
| [`llama-index`](llama-index.yaml) | `index_files`, `query` (retrieve + synthesize); opt-in `index_docs/web/site/github/gdrive` | pgvector secret + embeddings (local Ollama) + LLM token |
| [`langfuse`](langfuse.yaml) | `langfuse.create_score`; `run_experiment` opt-in | Langfuse; a dataset for the experiment |
| [`playwright`](playwright.yaml) | all 26 `playwright.*` against offline `data:` pages | Playwright browsers on the worker (`playwright install chromium`) |
| [`selenium`](selenium.yaml) | all 23 `selenium.*` against offline `data:` pages | Chrome + chromedriver on the worker |

## Patterns worth knowing

- **Long-running relay activities** (`kafka.consume`, `rabbit.receive`, `websocket.subscribe`,
  `graphql.subscribe`, streaming `mcp.call_tool`) start with `async_mode: true` and feed a state
  machine on their `relay_topic`. A `timeout_sec` on the listening state (a `sys.timer` event)
  makes sure the spec ends even when nothing arrives. Start the producer *after* the machine is
  idle in its listening state. Events that arrive while an `on_enter` is still running can be
  lost, so the kafka and rabbit specs publish from an async child workflow.
- **`builtin.delay` is a local activity.** Right after an `async_mode` activity it does *not*
  give that activity a head start on Temporal. Use `execute_locally: false`, or poll for
  readiness, which is what the websocket, graphql and mcp specs do.
- **`shell.run`'s `env` replaces the whole environment**, including `PATH`. To add one variable,
  prefix the command instead: `FOO=bar cmd`.
- **Some failures come back as data.** `kafka.publish` returns `result: fail`,
  `mcp.call_tool` returns `result: null`, and `claude_agent.query` returns `is_error: true`.
  The specs check for these explicitly.
