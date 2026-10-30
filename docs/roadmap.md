# Sympozium Roadmap

Last updated: 2026-10-30, after v0.12.1.

This page says where the project is, what is being worked on, and what comes
next. It is maintained: when something here ships or is dropped, the page
changes in the same pull request. Detailed plans live in the linked design
notes and GitHub issues; this is the map, not the territory.

## Direction

Sympozium runs LLM agents as Kubernetes resources. Two things set the
direction for 0.12 and beyond:

- **Hardware-isolated execution.** Agent turns run in [Celln](https://github.com/sympozium-ai/celln)
  micro-VMs on a model-free fleet. An Agent brings its own
  `ModelConnection` and key; nothing on a node ever holds a model credential,
  and every model request leaves through the model gateway under that Agent's
  identity. See [Agent-bound model access](design/celln-agent-bound-model-access.md).
- **Bring-your-own harness.** External runtimes (Codex, Claude Code, Pi,
  Hermes, Goose and friends) run under an administrator-approved
  `AgentRuntime` with a versioned adapter contract, instead of the built-in
  agent-runner. See the [harness epic (#349)](https://github.com/sympozium-ai/sympozium/issues/349).

Everything else (channels, ensembles, schedules, memory, the console) is kept
working on top of those two, and grows where users ask for it.

## Shipped in 2026

| When | What | Where to read |
|---|---|---|
| Oct (0.12) | Agent-bound model access: model-free Celln fleet, per-Agent keys pinned to their origin, any OpenAI- or Anthropic-compatible provider, direct Agent API (messages and `chat/completions`), onboarding wizard | [design](design/celln-agent-bound-model-access.md), [guide](guides/celln-mediated-model-access.md) |
| Oct (0.12.1) | In-place gate retry for agent-runner and harness-mode runs (resume), memory update/forget, thinking and `maxTokens`/`temperature` tuning, faster Celln admission and turns | [CHANGELOG](https://github.com/sympozium-ai/sympozium/blob/main/CHANGELOG.md) |
| Sep–Oct | Celln tenancy: one-shot and enduring runs from any authorised namespace, one-install fleet packaging, trusted transport, operator and tenant runbooks | [epic #495](https://github.com/sympozium-ai/sympozium/issues/495), [fleet installation](guides/celln-fleet-installation.md) |
| Sep | Enduring AgentRuns: persistent parents, disposable turn cells, exact-turn cancellation, lease and budget accounting | [epic #464](https://github.com/sympozium-ai/sympozium/issues/464), [design](design/enduring-agentrun-celln-epic.md) |
| Aug–Sep | Harness mode: `AgentRuntime`, adapter contract `v1alpha1`/`v1alpha2`, reference adapter, conformance suite, run-detail provenance | [epic #349](https://github.com/sympozium-ai/sympozium/issues/349), [harness adapters](modes/harness-adapters.md) |
| Mar–Jul | Web console, `sympozium doctor`, token usage and cost tracking, ensemble delegation (`delegate_to_persona`, canvas, await/resume), Slack, Discord, Telegram and WhatsApp channels, MCP servers and skill packs | [Concepts](concepts/custom-resources.md), [ensemble roadmap](design/ensemble-coordination-roadmap.md) |

## In progress

- **Celln turn latency.** With the provider out of the picture, a one-shot
  turn costs about 2.5 s of Sympozium and Celln time (admission 1.5 s, the
  rest observation and cleanup). Next: cache the resolver's reads during
  admission, make the per-turn tool budget a tool error instead of ending the
  turn, and let an Agent lend a leaner toolbox than the whole profile.
  Tracked in [#686](https://github.com/sympozium-ai/sympozium/issues/686); measurements in the
  [design note](design/celln-agent-bound-model-access.md#delivery).
- **Harness epic remainder (#349).** Signature or attestation policy for
  approved runtime images; whether remote MCP credentials stay visible to
  adapters or are mediated by a local proxy; SkillPack MCP server conformance
  for Streamable HTTP clients; stable failure categories for one-shot harness
  runs; pinned examples for Codex, Claude Code and Goose with support tiers;
  scheduled smoke tests with two real adapters.
- **Ensemble policy and safety** (phase 4 of the
  [ensemble roadmap](design/ensemble-coordination-roadmap.md)):
  relationship-scoped delegation rules, cycle detection, timeouts.

## Next

Ordered by what unblocks the most users. Each has, or will get, an issue.

1. **Pi and Hermes in a cell** ([#687](https://github.com/sympozium-ai/sympozium/issues/687)).
   Deferred from 0.12 because the only host-to-guest data path is the dispatch
   frame protocol; needs a bounded workspace channel first.
   [Why it was deferred](design/celln-agent-bound-model-access.md#harnesses-in-a-cell-pi-hermes-deferred).
2. **Fleet operations.** A per-namespace spend ceiling and refusal events on the
   run ([#688](https://github.com/sympozium-ai/sympozium/issues/688)); a KVM CI
   lane that boots cells on an enforcing CNI and publishes release evidence
   ([#685](https://github.com/sympozium-ai/sympozium/issues/685)). On the Celln
   side: secret providers with live withdrawal, fleet-wide revocation, and a
   persistent execution model, tracked in its
   [roadmap project](https://github.com/orgs/sympozium-ai/projects/1).
3. **Observability.** OpenTelemetry `gen_ai.*` semantic conventions on
   agent-runner spans (maintainer take-over of an external PR, in review), and
   an OTel transport with durable execution audit retention for the fleet.
4. **Provider breadth without code.** Presets for OpenAI-compatible gateways
   (Databricks, OrcaRouter and others) on top of the `custom` connection,
   model discovery per provider where the API allows it, a reachability
   condition on `ModelConnection` status ([#15](https://github.com/sympozium-ai/sympozium/issues/15)),
   the Responses API for OpenAI reasoning models with tools
   ([#608](https://github.com/sympozium-ai/sympozium/issues/608)), and a guide
   for subscription auth through a Claude Code harness adapter
   ([#684](https://github.com/sympozium-ai/sympozium/issues/684)).
5. **Channels.** Ensemble-aware routing (a receiver persona and `@mention`
   delegation, [#257](https://github.com/sympozium-ai/sympozium/issues/257)),
   an agent that can ask a question through its channel and wait for the
   answer mid-turn ([#20](https://github.com/sympozium-ai/sympozium/issues/20)),
   and the long-standing requests for Signal, Teams, Matrix and WebChat, in
   the order people ask.

Small, well-scoped items carry the `good first issue` label; the current set
is listed on the
[issues page](https://github.com/sympozium-ai/sympozium/issues?q=is%3Aopen+label%3A%22good+first+issue%22).

## Not planned

- A model credential held by the fleet, a node, or the installer. 0.12 removed
  it on purpose; see the threat model in the design note.
- Agent pods with host networking, host PID, or privileged SkillPacks under an
  external runtime.
- Reviving the February 2026 OpenClaw gap list as a plan. Its items are either
  shipped (dashboard, doctor, usage tracking, delegation), folded into the
  lists above, or not wanted; the
  [original design](design.md) is kept as history.

## How work is tracked

- **Issues** are the backlog. Epics carry the `epic` label and a checklist;
  slices link back to their epic. Labels name the area (`celln`, `harness`,
  `console`, `channels`, `provider`) and the kind (`bug`, `enhancement`,
  `documentation`, `help wanted`, `good first issue`).
- **Pull requests** are reviewed against current `main`. One untouched for 45
  days is labelled `stale` and closes two weeks later unless someone speaks
  up; a maintainer may instead take it over, keeping the author's commits and
  credit. Issues get 120 days and three weeks of grace. `pinned` exempts an
  item from both.
- **Releases** are cut by release-please from conventional commits; the
  [CHANGELOG](https://github.com/sympozium-ai/sympozium/blob/main/CHANGELOG.md)
  is the record of what shipped when. Celln is pinned in
  `config/celln/release.json` and moves by its own release.
- **Design notes** under [Design & History](design/index.md) record why; this
  page records what and when.
