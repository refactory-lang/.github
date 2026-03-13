# refactory-lang

**Refactory** — Semantic Profile-Guided Code Generation.

Write reviewable Python or TypeScript. Ship production Rust.

## Repositories

### Translation Pipelines

| Repository | What | Stages | Status |
|-----------|------|--------|--------|
| [`python-to-rust`](https://github.com/refactory-lang/python-to-rust) | Python → Rust | Tier 0 normalize → Tier 0.5 shadow-rewrite → Tiers 1–3 translate → verify | Phase 0 complete |
| [`typescript-to-rust`](https://github.com/refactory-lang/typescript-to-rust) | TypeScript → Rust | Tier 0 normalize → Tier 0.5 shadow-rewrite → Tiers 1–3 translate → verify | Planned (Phase 1) |
| [`core`](https://github.com/refactory-lang/core) | Shared transform utilities | — | Planned |

### Shadow Libraries

| Repository | What | Status |
|-----------|------|--------|
| [`shadows`](https://github.com/refactory-lang/shadows) | Python shadow libraries — 19 PyO3/maturin crates (Cargo workspace) + import hook | Phase 0.5 |
| [`shadows-ts`](https://github.com/refactory-lang/shadows-ts) | TypeScript shadow libraries — napi-rs | Planned (Phase 1) |

### Product SDKs

| Repository | What | Status |
|-----------|------|--------|
| [`ferrum-sdk`](https://github.com/refactory-lang/ferrum-sdk) | Ferrum EDM Python Component SDK (PyO3 bindings) | Planned (Phase 1) |
| [`sinter-sdk`](https://github.com/refactory-lang/sinter-sdk) | Sinter Step SDKs (Python + TypeScript) | Planned (Phase 1) |
| [`sinter-n8n-helpers`](https://github.com/refactory-lang/sinter-n8n-helpers) | n8n IExecuteFunctions shadow library | Planned (Phase 1) |

## Pipeline Architecture

Each translation pipeline (`python-to-rust`, `typescript-to-rust`) is a monorepo containing three composable stages:

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
         │     shadows repo       │
         │  (PyO3 runtime libs)   │
         └────────┬───────────────┘
                  ▼
        ┌────────────────────┐
        │  refactory-shadows │
        │  Cargo workspace   │
        │  19 PyO3 crates    │
        │  import hook       │
        └────────────────────┘
```

## Products

| Product | Source Language | Profile | Pipeline Repo |
|---------|---------------|---------|---------------|
| **Ferrum EDM** | Python | Python-as-Rust | `python-to-rust` |
| **Sinter** | Python + TypeScript | Both profiles | Both repos |
| **Refactory** (standalone) | Any | Any registered | Any pipeline repo |

## License

Apache-2.0
