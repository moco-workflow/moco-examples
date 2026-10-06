# Conversion report

37 files converted, 4 copied. Codes are explained in `scripts/dsl_convert/README.md`.

| Code | Severity | Count | Example |
|---|---|---|---|
| I101 | info | 47 | wfspec_version dropped: the package version is the workflow version |
| I112 | info | 2 | expression output_data without output_name: v1 also merged the value into the context when it was a dict |
| I134 | info | 2 | catch_exception: true converted to on_error |
| I170 | info | 10 | embedded v1 spec text converted to v2 |

## I101

- `moco-examples/activities/authz.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-basics.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-deploy-lifecycle.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-deploy-lifecycle.yaml:52` wfspec_version dropped: the package version is the workflow version [in exercise_wfspec#literal]
- `moco-examples/activities/builtin-deploy-rbac.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-deploy-rbac.yaml:51` wfspec_version dropped: the package version is the workflow version [in exercise_wfspec#literal]
- `moco-examples/activities/builtin-deploy.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-event.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-execute-workflow.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-execute-workflow.yaml:34` wfspec_version dropped: the package version is the workflow version [in summing_child#literal]
- `moco-examples/activities/builtin-execute-workflow.yaml:46` wfspec_version dropped: the package version is the workflow version [in failing_child#literal]
- `moco-examples/activities/builtin-secret.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/builtin-state.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/claude-agent-query.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/email-send.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/gdrive.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/graphql-subscribe.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/http-request.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/k8s.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/kafka.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/kafka.yaml:49` wfspec_version dropped: the package version is the workflow version [in publisher_wfspec#literal]
- `moco-examples/activities/langfuse.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/langfuse.yaml:43` wfspec_version dropped: the package version is the workflow version [in task_wfspec#literal]
- `moco-examples/activities/langfuse.yaml:53` wfspec_version dropped: the package version is the workflow version [in evaluator_wfspec#literal]
- `moco-examples/activities/llama-index.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/mcp-call-tool.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/openai-chat.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/playwright.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/rabbit.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/rabbit.yaml:47` wfspec_version dropped: the package version is the workflow version [in publisher_wfspec#literal]
- `moco-examples/activities/selenium.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/shell-run.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/sql.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/typesafe-alert-triage.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/typesafe-system-one.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/activities/websocket-subscribe.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/rules-engine/01-simple-rules-engine.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/rules-engine/02-loan-approval.yaml:16` wfspec_version dropped: the package version is the workflow version
- `moco-examples/rules-engine/03-live-rules-engine.yaml:49` wfspec_version dropped: the package version is the workflow version
- `moco-examples/rules-engine/03-live-rules-engine.yaml:74` wfspec_version dropped: the package version is the workflow version [in feed_wfspec#literal]
- `moco-examples/rules-engine/04-send-facts.yaml:13` wfspec_version dropped: the package version is the workflow version
- `moco-examples/state-machine/01-simple-state-machine.yaml:2` wfspec_version dropped: the package version is the workflow version
- `moco-examples/state-machine/02-continue-as-new.yaml:11` wfspec_version dropped: the package version is the workflow version
- `moco-examples/state-machine/03-continue-as-new-timer.yaml:18` wfspec_version dropped: the package version is the workflow version
- `moco-examples/state-machine/04-human-in-the-loop.yaml:45` wfspec_version dropped: the package version is the workflow version
- `moco-examples/state-machine/04-human-in-the-loop.yaml:74` wfspec_version dropped: the package version is the workflow version [in approver_wfspec#literal]
- `moco-examples/state-machine/05-send-decision.yaml:16` wfspec_version dropped: the package version is the workflow version

## I112

- `moco-examples/activities/typesafe-alert-triage.yaml:203` expression output_data without output_name: v1 also merged the value into the context when it was a dict
- `moco-examples/activities/typesafe-system-one.yaml:89` expression output_data without output_name: v1 also merged the value into the context when it was a dict

## I134

- `moco-examples/activities/builtin-deploy-lifecycle.yaml:377` catch_exception: true converted to on_error
- `moco-examples/activities/builtin-deploy-rbac.yaml:270` catch_exception: true converted to on_error

## I170

- `moco-examples/activities/builtin-deploy-lifecycle.yaml:50` embedded v1 spec text converted to v2
- `moco-examples/activities/builtin-deploy-rbac.yaml:49` embedded v1 spec text converted to v2
- `moco-examples/activities/builtin-execute-workflow.yaml:32` embedded v1 spec text converted to v2
- `moco-examples/activities/builtin-execute-workflow.yaml:44` embedded v1 spec text converted to v2
- `moco-examples/activities/kafka.yaml:47` embedded v1 spec text converted to v2
- `moco-examples/activities/langfuse.yaml:41` embedded v1 spec text converted to v2
- `moco-examples/activities/langfuse.yaml:51` embedded v1 spec text converted to v2
- `moco-examples/activities/rabbit.yaml:45` embedded v1 spec text converted to v2
- `moco-examples/rules-engine/03-live-rules-engine.yaml:72` embedded v1 spec text converted to v2
- `moco-examples/state-machine/04-human-in-the-loop.yaml:72` embedded v1 spec text converted to v2
