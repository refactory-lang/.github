<p align="center">
  <img src="https://raw.githubusercontent.com/refactory-lang/.github/main/assets/refactory-logo.svg" alt="Refactory" width="400">
</p>

<p align="center">
  <strong>Semantic Profile-Guided Code Generation</strong><br>
  Write reviewable Python or TypeScript. Ship production Rust.
</p>

<p align="center">
  Powering <strong>Ferrum EDM</strong> (compiled data pipelines) and <strong>Sinter</strong> (compiled workflow automation)
</p>

## Repositories

### Translation Pipelines

| Repository | Description | Status |
|-----------|-------------|--------|
| [`python-to-rust`](https://github.com/refactory-lang/python-to-rust) | Python → Rust translation pipeline (8-step: validate → normalize → shadow-rewrite → T1 → T2 → T3 → format → verify) | Phase 1 |
| [`typescript-to-rust`](https://github.com/refactory-lang/typescript-to-rust) | TypeScript → Rust translation pipeline | Phase 1 |
| [`core`](https://github.com/refactory-lang/core) | Shared transform utilities (`@refactory/core` — type mapping, case conversion, struct emission) | Phase 1 |
| [`rust-ir`](https://github.com/refactory-lang/rust-ir) | Typed Rust IR builder for JSSG transforms — grammar-faithful, render-then-validate | Phase 1 |

### Shadow Libraries

| Repository | Description | Status |
|-----------|-------------|--------|
| [`shadows-python`](https://github.com/refactory-lang/shadows-python) | 19 PyO3/maturin crates (Cargo workspace) + import hook — API-identical Python wrappers around Rust crates | Phase 0.5 |
| [`shadows-ts`](https://github.com/refactory-lang/shadows-ts) | TypeScript shadow libraries (napi-rs) | Phase 1 |
| [`sinter-n8n-helpers`](https://github.com/refactory-lang/sinter-n8n-helpers) | n8n IExecuteFunctions shadow — maps ~30-40 n8n helpers to Sinter equivalents | Phase 1 |

### Product SDKs

| Repository | Description | Status |
|-----------|-------------|--------|
| [`ferrum-sdk`](https://github.com/refactory-lang/ferrum-sdk) | Ferrum Python Component SDK — validate → test → translate → compile | Phase 2 |
| [`sinter-sdk`](https://github.com/refactory-lang/sinter-sdk) | Sinter Step SDKs (Python + TypeScript) — workflow step protocol | Phase 2 |

### Shared

| Repository | Description |
|-----------|-------------|
| [`refactory-template`](https://github.com/refactory-lang/refactory-template) | Common config template (speckit, Claude/agents, MCP) for all repos |

## Pipeline Architecture

The translation pipeline has 8 composable steps:

```
    Developer writes                                           Pipeline produces
    vanilla Python/TS                                          compiled Rust
          │                                                          ▲
          ▼                                                          │
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Normalize   │  │   Validate   │  │  Transform   │  │   Verify     │
│              │  │              │  │              │  │              │
│ Normalize-Det│→ │ Profile      │→ │ T1: syntax   │→ │ cargo build  │
│ Normalize-LLM│  │ Validator    │  │ T2: structural│  │ cargo clippy │
│              │  │ (ast-grep)   │  │ T3: LLM stubs│  │ cargo test   │
│ Shadow       │  │              │  │    (Rust→Rust)│  │              │
│ Rewrite      │  │              │  │ cargo fmt    │  │              │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
       │                                    │
       │  Shadow library repos provide      │  Tier Promotion Feedback Loop:
       │  Rust-backed runtime libs          │  T3 patterns → candidate JSSG →
       │  (PyO3 for Python, napi-rs for TS) │  CI validate → PR → merge →
       └────────────────────────────────────┘  shrinks T3 surface over time
```

## How It Works

1. **Profile** — A strict subset of the source language (e.g. "Python-as-Rust", "TypeScript-as-Rust") that makes code structurally translatable. Enforced by ast-grep rules.
2. **Normalize-Det** — Deterministic rewrites: idiomatic patterns → profile-compliant forms (`try/except` → `Result`, `throw` → `Err`, etc.)
3. **Normalize-LLM** — LLM-assisted normalization for patterns too complex for deterministic rules (class → readonly interface, complex control flow)
4. **Validate** — ast-grep profile validator confirms all code is profile-compliant before translation
5. **Shadow Rewrite** — Replace stdlib imports with shadow library paths that map 1:1 to Rust crates
6. **Transform T1/T2** — Deterministic AST transforms via JSSG (T1: types/structs/syntax, T2: modules/control/generators)
7. **Transform T3** — LLM-assisted Rust→Rust pass for constructs with no Python/TS representation (lifetimes, trait bounds, async patterns). Emits `todo!("t3:*")` stubs resolved by Claude.
8. **Verify** — `cargo build`, `cargo clippy`, `cargo test` on the formatted output

## Tier Promotion

The **Tier Promotion Feedback Loop** continuously shrinks the LLM-dependent surface: T3 structured outputs are clustered by AST fingerprint, candidate JSSG rules are generated, validated through CI, and surfaced as PRs. Approved rules become permanent T1/T2 transforms.

## License

Apache-2.0
