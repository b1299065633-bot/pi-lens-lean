# Changelog

## [4.5.0] - 2026-11-14

Re-pinned the upstream runtime from `pi-lens@4.4.12` to **`pi-lens@4.5.0`**. The facade (`index.ts`) is unchanged; only the pin, lockfile and documentation moved.

### Relevant upstream changes

- LSP navigation now accepts object-valued command arguments under non-strict schema conversion.
- Diagnostics and ast-grep fixes improve Rust, Deno and Elixir LSP handling, rule-relative scanning, and formatter selection.
- The Lean facade keeps its fixed two-tool surface and strips upstream activation and session-start guidance as before.

### Verification

- `npm run check` passes: TypeScript is clean and `tests/facade.test.mjs` passes 8/8, including routing against the real `pi-lens@4.5.0` schemas.
- Lean's two facade tools total **394 tokens** on the existing benchmark. The eight upstream tools total **2,733 tokens** on the same benchmark, a reduction of **85.6%**.
- Updated the three `@earendil-works` development/test packages from 1.0.3 to 1.5.2. This removes the vulnerable transitive `minimatch` and `undici` versions; `npm audit` reports **0 vulnerabilities**. Re-ran the same typecheck and 8/8 facade tests after the dependency update.

## [4.4.12] - 2026-11-10

Re-pinned the upstream runtime from `pi-lens@4.4.10` to **`pi-lens@4.4.12`**. The facade (`index.ts`) is unchanged; only the pin, the docs and the skill declaration moved.

| Variant | Tools | Tokens |
| --- | --- | ---: |
| Lean `pi-lens-lean@4.4.12` | `lens_code` + `lens` | **394** |
| Upstream `pi-lens@4.4.12` | 8 tools | **2,733** |

Saves **2,339 tokens (85.6%)** per request. The lean number is owned by the facade, which replaces upstream's tool descriptions and schemas wholesale, so it does not move with upstream releases unless `index.ts` changes.

### Skill declaration

Upstream registers its four skills through a `resources_discover` handler. The facade still blocks that handler, but `package.json` now declares the same four directories through `pi.skills` (`./node_modules/pi-lens/skills`). Without the declaration the skills only resolved because a second, full `pi-lens` install happened to sit in the global npm tree; once that copy was retired the four skills silently disappeared from the router.

### Upstream changes between 4.4.10 and 4.4.12

4.4.11 is a large release (the compare covers 428 commits) and 4.4.12 adds 54 more. The parts that touch this wrapper's contract:

- **MCP projection unified on `path`** (#5138): the six projected MCP tools now take `path` instead of `file`. The facade already routes every file argument onto `path`, so nothing changed here.
- **Tool binary resolver** (#5410): a broken or absent project shim falls through to the managed and PATH rungs instead of failing the check.
- **Write authorship** (#5417): `ast_grep_replace`, LSP edits and `lens_diagnostic_mark` share one source-file write seam.
- **Adopted project trust** (#5469) and **strict enum unions** in the projected MCP schemas (#5471, #5473).
- **Complexity delta metrics** (#5087) and runner timeout coalescing (#5413).

### Compatibility check

- `npm run check` passes: tsc clean, `tests/facade.test.mjs` 8/8 — including the case that resolves every facade `op` against the real pinned `pi-lens@4.4.12` schemas and the `renderSessionStartGuidance()` marker the wrapper strips.
- The four skill directories are present under `node_modules/pi-lens/skills` and are what `pi.skills` points at.

## [4.4.10] - 2026-11-09

Re-pinned the upstream runtime from `pi-lens@4.3.0` to **`pi-lens@4.4.10`** and re-measured on Pi 1.5.0. The facade itself is unchanged.

| Variant | Tools | Tokens |
| --- | --- | ---: |
| Lean `pi-lens-lean@4.4.10` | `lens_code` + `lens` | **394** |
| Upstream `pi-lens@4.4.10` | 8 tools | **2,733** |

Saves **2,339 tokens (85.6%)** per request, plus the 4 skill listings and the session-start guidance that the wrapper also drops.

### Compatibility check

- Every facade `op` still resolves to a real upstream tool and schema in 4.4.10 (covered by `tests/facade.test.mjs`, 8/8 passing).
- 4.4.10 renders the session-start guidance through `renderSessionStartGuidance()` instead of a literal array. The wrapper strips it by content prefix (`[pi-lens automated context — not a user request]\n\n📌 pi-lens active`), which is unchanged, so the behaviour holds; one static assertion in the test suite was updated to match the new source shape.

### Why this fork exists

`kunkun9527/pi-lens-lean` pins 4.3.0. Between 4.3.0 and 4.4.10 upstream shipped nine releases (new languages, new runners, config knobs), and this machine runs 4.4.10, so installing the original would have downgraded pi-lens.

## Earlier versions

See [kunkun9527/pi-lens-lean](https://github.com/kunkun9527/pi-lens-lean).
