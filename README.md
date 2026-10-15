# 07 · Durable Agent Runtime

Recoverable agent execution with event journaling, idempotent side effects, and human interrupt/resume.

**Status: planning — implementation has not started.** This README describes intended scope and acceptance criteria. It does not establish working behavior, completed tests, security guarantees, or measured performance.

Week 7 of an eight-repository agent-systems portfolio.

## Planned scope

- An append-only execution journal and deterministic state reduction.
- Checkpoint/replay semantics with explicit handling of nondeterministic model and tool results.
- Idempotency keys, pending-effect reconciliation, and persisted human approval boundaries.

## Planned setup and observability

The implementation is intended to provide a `docker-compose.yml` and `Makefile` so a reviewer can use `make demo` for a deterministic offline demonstration and `make test` for offline regression checks. **These commands and their supporting files are not implemented yet.** Live-model runs will be optional and separately documented with configuration and cost assumptions.

Every model call and tool execution is intended to emit an OpenTelemetry span. Planned aggregate reporting includes token usage, estimated USD per 1,000 requests, p50/p95 latency, and p50/p95 time to first token (TTFT) where streaming applies. Offline fixtures can verify instrumentation and behavior; live-provider performance requires separate measurements.

Implementation choices will remain inspectable, with explicit contracts and documented dependencies.

See [PRODUCTION_CHECKLIST.md](PRODUCTION_CHECKLIST.md) for shared acceptance criteria.

## Proposed architecture and tradeoffs

An event journal records inputs, decisions, model/tool results, and workflow transitions. A deterministic reducer reconstructs state, and an executor performs only effects allowed by the current state. Effect records and idempotency keys support recovery, while human decisions are persisted before resume.

Event history improves auditability and recovery but increases storage and schema-migration complexity. Local idempotency records alone cannot guarantee exactly-once effects across external systems. Recovery behavior must state the external service assumptions and reconciliation policy.

## Roadmap and acceptance criteria

Complete the shared reproducibility and observability checklist, then implement and verify these project-specific criteria:

- [ ] Recover the same workflow state after injected crashes at declared journal and effect boundaries.
- [ ] Avoid duplicate effects for supported idempotent integrations and surface ambiguous outcomes for reconciliation.
- [ ] Persist interrupt state and require a recorded human decision before resuming protected work.
- [ ] Compare recovery time, completion rate, and duplicate-effect rate under a reproducible crash schedule.

## Candidate failure modes

These are design risks to investigate, **not tested or observed findings**:

- A crash occurs after an external effect but before its completion is journaled.
- Replay reissues a model call or a side effect instead of using the recorded result.
- A journal schema change breaks recovery, or an approval expires or is replayed incorrectly.

The implementation will add reproduction cases, actual outcomes, and mitigations as evidence becomes available.

## Baseline versus optimized metrics

No benchmark has been run. The following comparison is planned:

- **Baseline:** A non-durable runner under the same injected crash schedule, with failures counted.
- **Optimized:** Journaled execution and idempotency/reconciliation under the same task and fault schedule.

| Metric | Baseline | Optimized |
| --- | --- | --- |
| Input/output/total tokens per request | Not yet measured | Not yet measured |
| Estimated USD per 1,000 requests | Not yet measured | Not yet measured |
| End-to-end latency p50 | Not yet measured | Not yet measured |
| End-to-end latency p95 | Not yet measured | Not yet measured |
| Streaming TTFT p50, where applicable | Not yet measured | Not yet measured |
| Streaming TTFT p95, where applicable | Not yet measured | Not yet measured |
| Project-specific quality and failure rate | Not yet measured | Not yet measured |

Future results must identify the workload revision, model/provider configuration, environment, concurrency, sample counts, and quality criteria. Cost estimates must disclose pricing date and assumptions. Report the complete workload, including errors and retries, and retain raw results so baseline and optimized runs can be reproduced. Do not substitute target values for measurements.

## Portfolio

- [01 · Raw Agent Tool Loop](https://github.com/sn41lz/agent-systems-01-tool-loop)
- [02 · Context Window Engine](https://github.com/sn41lz/agent-systems-02-context-window)
- [03 · Hierarchical Agent Memory](https://github.com/sn41lz/agent-systems-03-hierarchical-memory)
- [04 · Multi-Agent Orchestrator](https://github.com/sn41lz/agent-systems-04-multi-agent-orchestrator)
- [05 · Agent Evaluation Harness](https://github.com/sn41lz/agent-systems-05-eval-harness)
- [06 · Secure Code Sandbox](https://github.com/sn41lz/agent-systems-06-secure-sandbox)
- [07 · Durable Agent Runtime](https://github.com/sn41lz/agent-systems-07-durable-runtime)
- [08 · Agent Platform API and SDKs](https://github.com/sn41lz/agent-systems-08-api-sdk)
