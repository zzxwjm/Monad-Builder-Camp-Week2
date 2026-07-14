# Moss GitHub Exploration Log

Date: 2026-07-15  
Repository: [nishuzumi/moss](https://github.com/nishuzumi/moss)

## 1. Project structure

| Directory or file | Role observed from the repository |
|---|---|
| `.github/ISSUE_TEMPLATE/` | Separates bug reports, feature requests, and protocol-onboarding requests. |
| `.github/PULL_REQUEST_TEMPLATE.md` | Defines the evidence and checklist expected in a contribution. |
| `.github/workflows/ci.yml` | Main visible GitHub Actions CI entry point. |
| `docs/` | Documentation index, getting started, MCP reference, Agent rules, protocol onboarding, ADRs, and glossary. |
| `examples/` | Runnable examples: `simple-flow` and `agent-swap`. |
| `packages/core` | Core machinery such as Registry, capabilities, Plans, and shared semantics. |
| `packages/simulator` | Trace simulation, effects extraction, and reconciliation against `expects`. |
| `packages/erc` | Generic ERC-20/ERC-721 interfaces and approval-related helpers. |
| `packages/system` | Monad-specific runtime data and system adapters such as WMON. |
| `packages/protocols/` | One package per protocol; `main` currently shows `_template` and `kuru`. |
| `packages/mcp-server` | Exposes Moss capabilities as MCP tools and assembles the served catalog. |
| `.changeset/` | Release metadata for user-facing package changes. |
| `CONTRIBUTING.md` / `SECURITY.md` / `CONTEXT.md` | Contribution rules, security boundaries, and project vocabulary. |

## 2. How the GitHub modules work together

### README

The README is the product-level entry point. It explains the problem, the four-step flow `discover → load → action → simulate`, the current protocol capabilities, the alpha status, the safety boundaries, and the package layout.

### Docs

The Docs index is intentionally short and acts as a navigation layer:

- Getting Started: run the whole flow and then inspect each stage.
- MCP Tools Reference: exact contracts for the four Agent-facing tools, Plan fields, and warning codes.
- Agent Skill Guide: rules for external Agents, including mandatory simulation, stopping on warnings, and intent alignment.
- Protocol Onboarding: how to turn a protocol's contracts into a Moss adapter.
- ADRs: design decisions and trade-offs.
- Glossary: shared project vocabulary.

### Issues

The Issues list functions partly as a roadmap. It contains protocol-adapter tasks such as PancakeSwap, Uniswap v4, Clober, Aave, Morpho, Euler, Pendle, and FastLane, as well as interface-layer tasks like ERC-1155 and ERC-4626, vocabulary/design tasks, documentation work, and bug reports. Labels such as `good first issue`, `adapter`, `dex`, `lending`, and difficulty levels help communicate scope.

### Pull Requests

The PR template asks contributors to explain motivation, classify the change, report CI evidence, add a changeset for user-facing changes, and provide capability-specific proof. For a new capability, the contributor must declare semantic intent/params/risk, quantified `expects`, discover/load coverage, a reproducible `discover → load → action → simulate` path, and zero-warning simulation for the happy path.

### Discussions

The repository API reports `has_discussions: false` on 2026-07-15. The Discussions module is not enabled for Moss at the time of this exploration, so there is no project discussion history to summarize.

## 3. Selected feature: PR #24

I selected [PR #24: `feat(protocols): add PancakeSwap V3 single-hop swap adapter`](https://github.com/nishuzumi/moss/pull/24). It is an open, non-draft PR that adds a new protocol adapter and is linked to [Issue #6: `Adapter: PancakeSwap swap`](https://github.com/nishuzumi/moss/issues/6).

### What it tries to add

The PR adds a PancakeSwap V3 adapter for Monad with:

- a `swap` capability and a `quote` query;
- native MON as input through an inline WMON wrap step;
- ERC-20 output, with native MON output deliberately rejected in v1;
- exact approvals, slippage-protected minimum output, and transaction deadlines;
- protocol-specific ABI files, an adapter implementation, tests, MCP registration, README update, and changesets.

### What the PR reveals about maintainer practice

The PR is not only a code diff. It documents:

1. why the feature is needed;
2. the exact v1 scope and deferred features;
3. how contract addresses and ABIs were verified;
4. offline test results and live-chain verification;
5. a Monad-specific `sqrtPriceLimitX96` quirk;
6. a current limitation: the full V3 simulation path exceeds the simulator's `debug_traceCall` gas limit, so the author narrows the e2e claim and records an Anvil-fork backend as follow-up work.

The changed-file list crosses the repository layers: release metadata, README, MCP server registration, a new protocol package, ABI, implementation, token metadata, tests, TypeScript/Vitest configuration, and lockfile.

## 4. My findings

### Facts

- Moss organizes protocol support as one package per protocol.
- New capabilities are expected to be discoverable, loadable, simulatable, tested, documented, and registered in the MCP server.
- The repository has explicit templates and checklists for recurring contribution types.
- Discussions are currently disabled.

### Inferences

- The maintainers treat a protocol adapter as a vertical slice rather than an isolated contract wrapper.
- The project values reproducible evidence and explicit limitations as part of the contribution itself.
- The repository structure separates generic machinery, chain-specific data, protocol-specific behavior, and the Agent-facing product surface to control dependencies.

### Personal learning

I used to read GitHub mainly as a place to find source code. This exploration showed me that a real open-source project also exposes its management system: issue labels define possible work, templates define the expected shape of contributions, CI defines a minimum quality gate, changesets connect code to releases, and ADRs preserve design reasoning. In Moss, the maintainer process mirrors the product's own safety philosophy: claims about a capability should be accompanied by scope, tests, simulation evidence, and known limitations.

## 5. References

- [Moss README](https://github.com/nishuzumi/moss)
- [Moss Docs index](https://github.com/nishuzumi/moss/blob/main/docs/README.md)
- [Getting Started](https://github.com/nishuzumi/moss/blob/main/docs/getting-started.md)
- [MCP Tools Reference](https://github.com/nishuzumi/moss/blob/main/docs/mcp-tools.md)
- [Protocol Onboarding](https://github.com/nishuzumi/moss/blob/main/docs/protocol-onboarding.md)
- [Agent Skill Guide](https://github.com/nishuzumi/moss/blob/main/docs/agent-skill.md)
- [Contributing Guide](https://github.com/nishuzumi/moss/blob/main/CONTRIBUTING.md)
- [PR #24](https://github.com/nishuzumi/moss/pull/24)
- [Issue #6](https://github.com/nishuzumi/moss/issues/6)

