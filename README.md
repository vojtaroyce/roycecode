# RoyceCode

[![CI](https://github.com/vojtechhradsky/roycecode/actions/workflows/ci.yml/badge.svg)](https://github.com/vojtechhradsky/roycecode/actions/workflows/ci.yml)
[![Rust](https://img.shields.io/badge/rust-1.82%2B-orange)](https://www.rust-lang.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Native Rust whole-codebase analysis for AI agents.

RoyceCode scans mixed-language repositories, builds a semantic graph, and emits
machine-readable artifacts for structural triage. The current product surface is
the Rust CLI. The legacy Python implementation has been removed from this
repository.

## What RoyceCode Is For

RoyceCode is not only for huge monoliths.

It is useful on:

- small repositories when you want a structured machine contract instead of a
  shallow lint pass
- medium repositories when architectural drift, hidden runtime wiring, or AI
  review handoff starts becoming painful
- large polyglot repositories when you need graph-backed evidence, policy, and
  bounded agent context instead of loading the whole codebase into a prompt

Use it when you want answers like:

- what are the real runtime entrypoints?
- which files are structurally central or suspiciously isolated?
- where does a dangerous API live, and is it reachable from entry code?
- which loop-local expensive operations are on a real caller chain?
- what should an AI reviewer inspect first without reading the whole repo?

Do not think of RoyceCode as “only a big-codebase platform”.
Think of it as a layered analyzer:

- on a small repo, it gives you a precise artifact family and review contract
- on a large repo, it becomes a graph-backed reduction layer for human and AI review

## Quick Start

```bash
cargo install --git https://github.com/vojtechhradsky/roycecode.git roycecode --bin roycecode
cd your-project
roycecode analyze .
```

For local development from this repository:

```bash
cargo run --bin roycecode -- analyze .
```

For a quick human-readable summary after analysis:

```bash
roycecode report .
cat .roycecode/roycecode-report.md
```

For a graph-backed AI handoff without executing any agent:

```bash
roycecode agent .
```

## Recommended Usage By Repo Size

### Small repositories

Use:

- `roycecode analyze .`
- `roycecode report .`
- `roycecode info .`

Why:

- you usually want the summary, findings, surface, and guard decision
- full graph artifacts are still useful, but you may not need `agent-run` or `cypher`

### Medium repositories

Use:

- `roycecode analyze .`
- `roycecode surface .`
- `roycecode agent .`
- optionally `roycecode tune .`

Why:

- this is where architecture surface, topology, packets, and AI handoff start
  paying off
- policy tuning starts to matter

### Large or framework-heavy repositories

Use:

- `roycecode analyze . --output-dir <dir>`
- `roycecode agent .`
- `roycecode agent-run . --adapter <name>`
- `roycecode graph . --kuzu`
- `roycecode cypher .`

Why:

- large repos need bounded graph packets, topology, and queryable graph projections
- this is where the AI-facing surfaces become first-class, not optional

## Product Layers

RoyceCode is easiest to use correctly if you think in layers:

1. Parsing and resolution
- mixed-language source extraction
- symbol/reference resolution
- framework/runtime overlays

2. Graph truth
- `semantic-graph.json`
- `dependency-graph.json`
- `evidence-graph.json`

3. Detector and assessment truth
- dead code
- hardwiring
- native security
- architectural assessment
- secondary scanner evidence

4. Review and guard truth
- `architecture-surface.json`
- `review-surface.json`
- `convergence-history.json`
- `guard-decision.json`

5. AI handoff and execution
- `agentic-review.json`
- `graph-packets.json`
- `repository-topology.json`
- `agent-run`
- `agent-spider`

This matters because not every command is for the same job. `analyze` builds
truth. `surface` and `report` summarize it. `agent` packages it for AI. `graph`
and `cypher` expose lower-level graph access.

## Commands

```text
roycecode analyze <path>      Run deterministic analysis and write native artifacts
roycecode agent <path>        Print the graph-backed AI review contract
roycecode agent-run <path>    Execute the AI review through a concrete adapter and write agent reports
roycecode agent-spider <path> Crawl top task packets through a concrete adapter and write per-packet reports
roycecode report <path>       Compatibility alias that also writes roycecode-report.json
roycecode analyze-rust <path> Compatibility alias for analyze
roycecode info <path>         Inspect existing Rust-native artifact state
roycecode plugins             List built-in runtime/framework overlay plugins
roycecode tune <path>         Suggest a conservative policy patch from current analysis signals
roycecode surface <path>      Emit architecture surface JSON
roycecode mcp <path>          Start the native Rust stdio MCP server for one repository
roycecode version             Print CLI version
```

Use `--output-dir <path>` to write artifacts outside `.roycecode/`.
Use `--no-write` to print JSON without writing artifacts.
Use `--external-tool <name>` or `--external-tools all` to run native Rust
external adapters alongside deterministic analysis.

## Which Command Should I Use?

### `roycecode analyze <path>`

Use this first.

It runs the full deterministic pipeline and writes the full native artifact
family. If you are unsure what command to use, use this one.

Use it when you want:

- graph artifacts
- detector output
- review surface
- guard decision
- AI handoff artifacts

### `roycecode report <path>`

Use this when you want the same analysis pipeline but care mainly about the
consolidated report output.

It is a compatibility alias for `analyze` that still writes the full artifacts.

### `roycecode info <path>`

Use this when artifacts already exist and you want a quick structured view of
their current state without reasoning from scratch.

Good for:

- shell scripts
- CI inspection
- checking whether a previous run already produced the artifacts you need

### `roycecode surface <path>`

Use this when you mainly want the architecture-facing summary and not the raw
low-level graph files.

Good for:

- UI layers
- dashboards
- quick triage workflows

### `roycecode agent <path>`

Use this when the consumer is another AI agent, not just a human.

It runs the same analysis pipeline, then prints the graph-backed AI review
contract built around:

- bounded packets
- traces
- code flows
- source/sink paths
- semantic state-flow evidence
- topology summaries

### `roycecode agent-run <path>`

Use this when you want RoyceCode to execute the review through a concrete AI
adapter, not only prepare the contract.

Current adapters:

- `codex-exec`
- `responses-http`
- `codex-sdk`

### `roycecode agent-spider <path>`

Use this when you want multiple top task packets executed, not just a single
whole-repo AI review.

This is for crawling the highest-priority bounded investigations.

### `roycecode graph <path>`

Use this when you want graph artifacts without the full detector/report stack.

Good for:

- graph debugging
- graph export
- code-understanding workflows

Add `--kuzu` when you also want the optional Kuzu materialization.

### `roycecode cypher <path>`

Use this when you want to query the optional Kuzu graph index for code
understanding.

This is lower-level than `surface` or `agent`.

### `roycecode tune <path>`

Use this after analysis when you want a conservative starter patch for
`.roycecode/policy.json`.

### `roycecode plugins`

Use this when you want to know which built-in semantic model packs and runtime
plugins are active in the current binary.

### `roycecode mcp <path>`

Use this when another tool or agent wants to consume RoyceCode through MCP over
stdio instead of reading JSON files directly.

## Common CLI Options

### `--output-dir <dir>`

Write artifacts outside `.roycecode/`.

Use this when:

- you do not want to dirty the target repo
- you are comparing repeated runs
- you want to keep multiple artifact baselines

### `--no-write`

Print JSON to stdout without writing artifacts.

Use this for:

- shell pipelines
- smoke checks
- quick experiments

Do not use it if you want the full reusable artifact family on disk.

### `--external-tool <name>` / `--external-tools <csv>`

Run external analyzers and normalize them into the same report/review surface.

Use this when you want:

- OpenGrep / Trivy / Grype / Gitleaks / audit-tool enrichment
- unified review and policy handling across native and imported findings

## Example Workflows

### Fast local repo check

```bash
roycecode analyze .
cat .roycecode/roycecode-report.md
```

### Analyze without writing into the repo

```bash
roycecode analyze . --output-dir /tmp/my-repo-royce
cat /tmp/my-repo-royce/roycecode-report.md
```

### Prepare AI review context

```bash
roycecode agent . --output-dir /tmp/my-repo-royce
cat /tmp/my-repo-royce/agentic-review.json
```

### Execute the AI review

```bash
roycecode agent-run . --adapter codex-exec --output-dir /tmp/my-repo-royce
```

### Build graph-only artifacts

```bash
roycecode graph . --kuzu --output-dir /tmp/my-repo-graph
```

### Query the Kuzu graph

```bash
roycecode cypher . --output-dir /tmp/my-repo-graph
```

## Artifacts

`roycecode analyze` writes:

```text
.roycecode/deterministic-analysis.json
.roycecode/semantic-graph.json
.roycecode/dependency-graph.json
.roycecode/evidence-graph.json
.roycecode/contract-inventory.json
.roycecode/doctrine-registry.json
.roycecode/deterministic-findings.json
.roycecode/ast-grep-scan.json
.roycecode/external-analysis.json
.roycecode/architecture-surface.json
.roycecode/review-surface.json
.roycecode/convergence-history.json
.roycecode/guard-decision.json
.roycecode/roycecode-handoff.json
.roycecode/agentic-review.json
.roycecode/graph-packets.json
.roycecode/repository-topology.json
.roycecode/roycecode-report.json
.roycecode/roycecode-report.md
```

When external tools are enabled, raw scanner artifacts are archived under:

```text
.roycecode/reports/<run_id>/raw/
```

`roycecode surface` prints the architecture surface JSON and also writes:

```text
.roycecode/architecture-surface.json
```

`roycecode mcp` serves tools, resources, and prompts over stdio from the same
native artifact family.

## How To Read The Main Artifacts

If you only read a few files, start here:

- `roycecode-report.json`
  - consolidated machine summary
- `roycecode-report.md`
  - readable human summary
- `architecture-surface.json`
  - architecture-facing triage view
- `review-surface.json`
  - visibility/policy-adjusted review contract
- `guard-decision.json`
  - current allow/warn/block posture
- `agentic-review.json`
  - graph-backed AI review contract

Read the lower-level graph artifacts when you need deeper explanation:

- `semantic-graph.json`
  - canonical resolved graph
- `dependency-graph.json`
  - low-noise dependency projection
- `evidence-graph.json`
  - richer evidence-oriented projection
- `deterministic-findings.json`
  - raw detector truth before summarization

`roycecode agent` runs the normal analysis pipeline, writes the same artifact
family, and prints `agentic-review.json` as the primary machine contract for an
AI reviewer. The AI contract is graph-backed, includes diff-aware task packets,
trace-style evidence chains, bounded typed multi-path graph traces, and bounded
code-flow style evidence paths plus explicit source/sink endpoints, bounded
semantic state-flow evidence for mutable carriers when the slice supports it, and now
carries an adapter catalog with:
- local `codex exec`
- direct OpenAI Responses HTTP
- optional TypeScript Codex SDK sidecar

`ast-grep-scan.json` is the first secondary scanner-plane artifact. It carries
typed, provenance-rich structural rule hits from in-process `ast-grep`
evaluation. Today it covers three pilot families:
- loop-local expensive-operation rules (`collection scan`, `sort`, `regex compile`,
  `json decode/parse`, and `filesystem read/check` in loops) that strengthen
  `AlgorithmicComplexityHotspot`
- dangerous-API rules (`eval`, `exec/system`, unsafe deserialization, unsafe HTML
  output`) that strengthen native `SecurityDangerousApi` findings
- narrow framework-misuse rules (`raw env outside config/bootstrap`,
  `raw container lookup outside provider/bootstrap or injection boundaries`)
  that strengthen native `SanctionedPathBypass` findings

The engine stays in core, but framework-specific rule catalogs are now allowed
to contribute findings with explicit provenance such as
`ast_grep.pattern.laravel` and `ast_grep.pattern.django`, so framework misuse
can scale without turning the core scanner file into a framework registry.

It is secondary evidence, not semantic-graph truth, reachability truth, or
doctrine truth. `roycecode-report.json.summary` and
`architecture_surface.overview` now also break those scanner hits down by
family so the scanner mix is visible without opening the raw artifact.

`graph-packets.json` complements `agentic-review.json` with bounded,
doctrine-aware graph neighborhoods for the current top packets and focus files.
Fallback focus-file packets now also carry bounded traces, code flows,
source/sink paths, and semantic state-flow evidence when guardian packets are
absent, so the packet layer stays useful even on degraded or slice-only runs.

`repository-topology.json` complements both with a flatter orchestration map
over top-level zones, manifests, runtime entries, contract-bearing directories,
cross-zone links, direct zone-to-finding / zone-to-packet links, zone-level
triage briefs, structured triage steps, focus clusters for flat zones,
explicit cross-zone pressure summaries with linked-zone previews, direct causal
bridge summaries, topology-level semantic-state previews/counts and proof-aware
summaries for mutable carriers, a topology recommended start slice, spillover observations,
convergence-state hints, and lightweight ownership hints with explicit basis
metadata so agents can reason about repository layout, cross-zone drag,
semantic propagation, and the next slice without loading the full evidence
graph. Route-declared files now also promote runtime-entry shape here, so
modern Symfony/Laravel-style controller surfaces are no longer invisible in
topology. Scoped/cropped analyses now also expose explicit `boundary_truncated`
truth here instead of implying fake orphan debt for files
whose real callers live outside the analyzed slice.
Topology semantic-state previews now also expose stable flow IDs, flow kind,
and proof tier (`exact_resolved`, `receiver_typed`, `heuristic`), and
zones/steps/clusters carry proof summaries plus compact flow refs so agents can
tell strong propagation evidence from weak heuristic hints and jump back to one
exact flow instead of only reading labels.

`roycecode agent-run` is the first real executor. It materializes the normal
artifact family, selects an adapter, and writes:

```text
.roycecode/agent-review.json
.roycecode/agent-review.md
.roycecode/agent-output-schema.json
.roycecode/agent-execution.jsonl
```

`roycecode agent-spider` crawls the top task packets from `agentic-review.json`
through the same adapter boundary and writes:

```text
.roycecode/agent-spider-report.json
.roycecode/agent-spider/<packet>/agent-review.json
.roycecode/agent-spider/<packet>/agent-review.md
.roycecode/agent-spider/<packet>/agent-output-schema.json
.roycecode/agent-spider/<packet>/agent-execution.jsonl
```

Current working adapters:
- `codex-exec` for local Codex CLI execution
- `responses-http` for direct Rust `v1/responses` execution with `OPENAI_API_KEY`

Planned adapter:
- `codex-sdk` as a thin optional TypeScript sidecar around the official Codex SDK

## What It Finds

- Circular dependencies
- Bottlenecks and orphan files
- Boundary-truncated files on scoped analyses
- Dead code candidates
- Hardwired values
- Native dangerous-API security findings
- Graph-backed boundary/input-to-sink security evidence
- Algorithmic complexity hotspots with caller/callee pressure and operation provenance
- Declared routes, hooks, env keys, config keys, and symbolic runtime contracts
- Architecture-surface summaries for UI and agent workflows

Some of those are stronger than others today. The product is designed to emit:

- hard graph truth where it has it
- structured heuristic evidence where exact proof is not yet available

That is why some contracts carry explicit provenance, scanner support, flow
paths, or boundary-truncated markers.

## Policy And Rules

The Rust review/report layer now reads optional suppression files from the
target repository:

- `.roycecode/policy.json` for project-wide patterns such as `orphan_entry_patterns`,
  `abandoned_entry_patterns`, `skip_path_patterns`, `allowed_literals`, and
  `repeated_literal_min_occurrences`, plus external-finding controls like
  `external.skip_tools`, `external.skip_categories`, and `external.allowed_rule_ids`
- `.roycecode/rules.json` for narrow per-finding exclusions by finding type,
  file pattern, and optional symbol/value or external tool match

Raw deterministic analysis remains in `deterministic-analysis.json`. The
policy/rule overlay is reflected in `review-surface.json`, `roycecode-report.json`,
and the native MCP server.

## External Tools

The Rust CLI can also orchestrate external analyzers and normalize their output
into `external-analysis.json`, `review-surface.json`, `roycecode-report.json`,
and MCP finding workflows.

Current native adapters:

- `ruff`
- `gitleaks`
- `pip-audit`
- `osv-scanner`
- `composer-audit`
- `npm-audit`
- `cargo-deny`
- `cargo-clippy`

## Supported Languages

- Rust
- PHP
- Python
- TypeScript / JavaScript
- Ruby

Language support here means parsing and graph extraction in the Rust engine. Parity
is still in progress for some higher-level detectors.

## Current Plugins And Model Packs

Use `roycecode plugins` to inspect the live binary, but the current built-in
families include:

- semantic model packs for:
  - Django routes
  - Django signals
  - Django settings
  - PHP hook maps
  - WordPress REST routes
- runtime plugins for:
  - queue dispatch
  - Laravel container resolution
  - generic signal callbacks
  - WordPress hooks

These are overlays on top of the generic core. They add framework/runtime
meaning without turning the core parser and graph layers into framework-specific
code paths.

## Development

```bash
cargo fmt
cargo test
cd website && npm ci && npm run build
```

## Current Direction

- Rust is the only product runtime.
- Python packaging, CLI, MCP host, report shell, and tests have been removed.
- Public docs and website examples now target the Rust CLI.

## License

MIT. See [LICENSE](LICENSE).
