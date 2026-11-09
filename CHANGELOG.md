# Changelog

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
