<p align="center">
  <img src="https://raw.githubusercontent.com/refactory-lang/.github/main/assets/refactory-logo.svg" alt="Refactory" width="400">
</p>

<p align="center">
  <strong>Semantic Profile-Guided Code Generation</strong><br>
  Write reviewable Python or TypeScript. Ship production Rust.
</p>

## Repositories

### Translation Pipelines

| Repository | Description | Status |
|-----------|-------------|--------|
| [`python-to-rust`](https://github.com/refactory-lang/python-to-rust) | Python → Rust translation pipeline (normalize → shadow-rewrite → transforms → verify) | Phase 0 |
| [`typescript-to-rust`](https://github.com/refactory-lang/typescript-to-rust) | TypeScript → Rust translation pipeline | Planned |
| [`core`](https://github.com/refactory-lang/core) | Shared transform utilities (type mapping, case conversion, struct emission) | Planned |

### Shadow Libraries

| Repository | Description | Status |
|-----------|-------------|--------|
| [`shadows-python`](https://github.com/refactory-lang/shadows-python) | 19 PyO3/maturin crates (Cargo workspace) + import hook — API-identical Python wrappers around Rust crates | Phase 0.5 |
| [`shadows-ts`](https://github.com/refactory-lang/shadows-ts) | TypeScript shadow libraries (napi-rs) | Planned |

### Shared

| Repository | Description |
|-----------|-------------|
| [`refactory-template`](https://github.com/refactory-lang/refactory-template) | Common config template (speckit, Claude/agents, MCP) for all repos |

## Pipeline Architecture

Each translation pipeline is a monorepo with three composable stages:

```
          Developer writes               Pipeline produces
          vanilla Python/TS              compiled Rust
                │                              ▲
                ▼                              │
┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐  ┌──────────┐
│     normalize/     │  │  *-shadow-rewrite/  │  │    transforms/     │  │  verify   │
│     (Tier 0)       │→ │  (Tier 0.5)        │→ │    (Tiers 1–3)     │→ │  (cargo)  │
│                    │  │                    │  │                    │  │          │
│  try/except→Result │  │  datetime→chrono   │  │  T1: types,structs │  │  build   │
│  raise→Failure()   │  │  re→regex          │  │  T2: modules,ctx   │  │  clippy  │
│  Optional→Maybe    │  │  json→serde_json   │  │  T3: traits,lifetm │  │  test    │
│  dataclass→frozen  │  │  ...19 libraries   │  │                    │  │          │
└────────────────────┘  └────────────────────┘  └────────────────────┘  └──────────┘
         │                        │
         │  shadows-python repo   │
         │  (PyO3 runtime libs)   │
         └────────┬───────────────┘
                  ▼
        ┌────────────────────┐
        │  shadows-python    │
        │  Cargo workspace   │
        │  19 PyO3 crates    │
        │  import hook       │
        └────────────────────┘
```

The `shadows-python` repo provides runtime PyO3 libraries that shadow rewrite rules point at. The import hook in `shadows-python/hook/` activates during development/testing so developers write vanilla Python against real Rust implementations.

## How It Works

1. **Profile** — A strict subset of the source language (e.g. "Python-as-Rust") that makes code structurally translatable. Enforced by ast-grep rules.
2. **Normalize** (Tier 0) — Rewrite idiomatic patterns into profile-compliant forms (`try/except` → `Result`, `Optional` → `Maybe`, etc.)
3. **Shadow Rewrite** (Tier 0.5) — Replace stdlib imports with shadow library paths that map 1:1 to Rust crates
4. **Transform** (Tiers 1–3) — Deterministic AST transforms (T1: types/structs, T2: modules/control), then LLM-assisted (T3: traits/lifetimes)
5. **Verify** — `cargo build`, `clippy`, `cargo test` on the output

## License

Apache-2.0
