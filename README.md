# pi-lens-lean

<!-- token-benchmark:summary:start -->
> **Token benchmark: Lean 394, upstream `pi-lens@4.5.0` 2,733 — 85.6% fewer.**
<!-- token-benchmark:summary:end -->
> **Full setup reference:** [Pi Lean Setup](https://github.com/kunkun9527/my-lean-pi-setup)

[简体中文](README.zh-CN.md)

A lean wrapper around [`pi-lens`](https://github.com/apmantza/pi-lens). The upstream runtime runs unchanged (post-write checks, LSP, formatting, read-guard, findings delivery); only the context the model sees on every request is cut down.

## Installation

```bash
pi install git:github.com/b1299065633-bot/pi-lens-lean
```

Not published to npm yet. To pin a release, append a tag: `git:github.com/b1299065633-bot/pi-lens-lean@v4.5.0`. Remove `npm:pi-lens` first; loading both registers pi-lens twice. npm may warn that the `@ast-grep/cli` install script is not approved; ast-grep still works, so the warning can be ignored.

## Differences from upstream

* **Two fixed tools instead of 13**: `lens_code` (find and read code) and `lens` (diagnostics, LSP, ast-grep, project and config). Each `op` is the upstream tool name, so hints in pi-lens output such as "call `module_report`" still match.
* **The tool list never changes mid-session**: upstream's `pi_lens_activate_tools` changes the active tools and invalidates the whole prompt cache. The wrapper drops that tool and turns upstream `setActiveTools` calls into no-ops.
* **Skills declared statically**: upstream registers four skills through a `resources_discover` handler; the facade blocks that handler and declares the same four directories through `pi.skills` instead, so the skills stay discoverable without upstream's dynamic registration. They cost what upstream's listing costs (about 337 tokens, not counted in the benchmark below). The two usage guides are rewritten into `help`; the two contributor guides for writing pi-lens rules stay as skills.
* **No session-start usage note** (about 197 tokens, once per session). Post-write findings, test results and the "files were reformatted, re-read before editing" nudge are kept.
* **Arguments are validated locally**: the host validates only the facade schema, so after mapping fields the wrapper checks them against the upstream schema with pi-ai's `validateToolArguments`. Unknown argument names, malformed JSON and field/`input` conflicts fail before reaching upstream.

## Usage

| Tool | `op` | Common fields |
| --- | --- | --- |
| `lens_code` | `symbol_search` / `module_report` / `read_symbol` / `read_enclosing` / `ast_grep_outline` | `path`, `paths`, `query`, `symbol`, `line` |
| `lens` | `lens_diagnostics` / `lsp_navigation` / `ast_grep_search` / `ast_grep_replace` / `lens_diagnostic_mark` / `project_report` / `effective_config` | the same, plus `source` |

* `path` lands on the matching upstream field: `path`, `filePath`, `file`, or a one-item `paths`.
* Other upstream arguments go in `input` as a JSON object string, for example `{"operation":"definition"}`.
* The tool descriptions name each op's required arguments, so a first call works without `help`. A test checks them against the pinned upstream schemas. For position-based `lsp_navigation` operations, pass `symbol` or `character`; otherwise upstream uses column 1 and often returns nothing.
* `op: "help"` with `input` set to an op returns that op's upstream description, full JSON schema and, for diagnostics, LSP and ast-grep, a usage guide.

```json
{ "op": "read_enclosing", "path": "src/app.ts", "line": 42 }
{ "op": "lens_diagnostics", "source": "lsp", "paths": ["src/a.ts"], "input": "{\"scope\":\"paths\"}" }
{ "op": "lsp_navigation", "path": "src/app.ts", "line": 10, "symbol": "foo", "input": "{\"operation\":\"references\"}" }
```

Tools disabled with `tools.<name>.enabled: false` in the pi-lens config make their op report that it is disabled.

## Using with pi-hashline-edit-pro-lean

With the default settings of [pi-hashline-edit-pro-lean](https://github.com/kunkun9527/pi-hashline-edit-pro-lean), `replace` and `insert` pass only anchors, not a file path. pi-lens finds the edited file from the `path` argument, so these edits get no post-edit diagnostics. Only `write` does. Two config changes fix this:

`~/.config/pi-hashline-edit-pro/config.json`, the upstream config file that the lean wrapper also reads:

```json
{ "requirePath": true }
```

`replace` and `insert` then require `path`, and hashline checks that it matches the anchors. Diagnostics appear in the same tool result. The cost is the path in each edit call, a few tokens for a relative path.

`~/.pi-lens/config.json`:

```json
{ "readGuard": { "enabled": false } }
```

Once pi-lens can see the path, its read-guard blocks edits to files not `read` in the session. That breaks the `anchor_grep` → `replace` flow and costs an extra `read` per file. Hashline anchors already reject stale content, so the guard is redundant here. Without it, edits made with the built-in `edit` tool also skip the read check.

## Initialization context comparison

<!-- token-benchmark:benchmark:start -->
With only this extension enabled, its recurring model-facing initialization contribution is:

| Variant | Tool and prompt contribution | Total |
| --- | --- | ---: |
| Lean `pi-lens-lean@4.5.0` | `lens_code` (160) + `lens` (234) | **394** |
| Upstream `pi-lens@4.5.0` | `lens_diagnostics` (643) + `symbol_search` (384) + `effective_config` (209) + `project_report` (257) + `module_report` (565) + `read_symbol` (233) + `read_enclosing` (322) + `pi_lens_activate_tools` (120) | **2,733** |

This saves **2,339 tokens (85.6%)**.
Measured with Pi 1.5.0 in separate temporary processes with empty working directories and configuration. Built-in tools, skills, context files, session history, user messages, unrelated extensions, runtime UI, and slash commands are excluded; `before_agent_start` additions are included. Tokens are a fixed character-proxy estimate using `ceil(characters / 4)`, not provider tokenizer billing.
<!-- token-benchmark:benchmark:end -->

## Version

The upstream runtime is pinned to `pi-lens@4.5.0` from npm. This is a fork of [kunkun9527/pi-lens-lean](https://github.com/kunkun9527/pi-lens-lean), re-pinned to 4.5.0. This repository is a fork of upstream: the default `main` branch holds this wrapper, and the `upstream` branch mirrors upstream pi-lens as a source reference.

## Local development

```bash
npm ci
npm run check
```

## License and credits

MIT. Wraps the MIT-licensed [`pi-lens`](https://github.com/apmantza/pi-lens).
