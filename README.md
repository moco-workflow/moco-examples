# moco-examples-v2

The specs in [`moco-examples`](../moco-examples), converted to the DSL v2 proposal
([`moco-core/docs/dsl-v2`](../moco-core/docs/dsl-v2/design.md)).

**These files are generated; do not edit them by hand.** The one exception is
[`basics/`](basics/README.md), which is written by hand to teach v2 idioms and new features
(`branches`, `switch`, `on_error`, `raise: continue`, `loop.index`...). The regeneration below
skips it. For everything else, change the v1 original, then regenerate:

```bash
make convert-examples-v2
```

This runs `python -m scripts.dsl_convert --to v2 moco-examples -o moco-examples-v2 --exclude 'basics/*'`. See
[`scripts/dsl_convert/README.md`](../scripts/dsl_convert/README.md) for how each v1 statement maps
to v2. The test suite (`make test-scripts`) fails when this copy is out of date.

## What to expect

**Running.** The engine runs DSL v2 next to v1 (it picks by the top-level key, and either version
may call the other as a child), so `moco run` and `moco test` work here as they do on
`moco-examples`. `moco test` finds a unit test's step by `task_name`, and a v2 mock may give its
raw output as `_`. The reference validator checks the files without running them:

```bash
python moco-core/docs/dsl-v2/validate.py moco-examples-v2
```

`moco-core/tests/integration/workflow/engine/v2/test_examples_v2_corpus.py` runs each conversion
next to its v1 original and compares the results. The known differences it lists are converter
gaps: v1 strips `#` modifiers from mapping keys at any depth while v2 only does at the top of a
write position, and v1 `output_name: x` tolerates an unset `x` where `output: "{{ x }}"` does not.

**Layout.** The folders and file names match `moco-examples`. Apart from `basics/`, the README in
each folder is copied from the v1 tree and still describes the v1 originals; its prose about v1
statements maps to v2 through the converter's table.

**Comments.** Comments are carried over from the v1 files. A comment that explains v1 syntax, such
as `iter_items[0]` or `output_data`, still uses the v1 names.

**Embedded specs are converted too.** This covers child specs given as `content#literal` text
(they become structured `workflow.contents`) and specs stored as strings, for example in
`context`. Activities such as `builtin.execute_workflow` and `builtin.deploy.*` therefore receive
v2 text, which only a v2-aware engine accepts.

**Conversion report.** [`CONVERSION_REPORT.md`](CONVERSION_REPORT.md) lists every diagnostic the
conversion produced. The warnings mark the places where the v2 form may behave differently from the
v1 original:

- W140: a `raise` payload field has no v2 field;
- W113: an `@` on `output_name` that v1 ignored.
