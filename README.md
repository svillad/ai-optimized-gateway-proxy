# AI Optimized Gateway Proxy

An AI API gateway project focused on concurrency, resilience, and visibility
into inference latency and cost. The goal is to build a Go backend and a
real-time React dashboard through small, verifiable milestones.

## Current Status

**Design and documentation stage.** This repository currently contains the
technical guide and initial repository configuration. The gateway, dashboard,
tests, and CI pipeline have not been implemented yet.

The capabilities and stack below describe the intended direction, not features
that are available today. There are no installation or execution steps yet.

## Planned Capabilities

- Bounded concurrency and explicit backpressure to control resource usage.
- Concurrent token-bucket rate limiting.
- In-memory caching with a defined eviction policy and memory limits.
- Provider integrations for local inference through Ollama and external APIs
  such as OpenAI.
- Request routing, health checks, and controlled failover between providers.
- Response streaming with cancellation and timeout handling.
- Operational telemetry and a real-time dashboard for latency, throughput,
  cache effectiveness, estimated cost, and provider health.

## Planned Stack

| Area | Technology |
| --- | --- |
| Backend | Go, prioritizing the standard library and `net/http` |
| Dashboard | React, TypeScript, and Tailwind CSS |
| Providers | Ollama and OpenAI |
| Real-time telemetry | SSE or WebSocket, based on actual requirements |

Implementation choices will be revisited as requirements and measurements
become available. Correctness, bounded resource usage, and maintainability take
priority over premature optimization.

## Documentation

The technical guide is available in Spanish:

- [Read the technical guide (PDF)](docs/ai_optimized_gateway_proxy.pdf)
- [View the LaTeX source](docs/ai_optimized_gateway_proxy.tex)

It covers the proposed architecture, concurrency concepts, engineering risks,
and a milestone-based learning roadmap. Its examples and directory layout are
design references, not a description of implemented application code.

## Development Workflow

Development proceeds incrementally, with a clear scope and acceptance criteria
for each milestone:

1. Create a milestone branch from `main`.
2. Build in small steps with focused commits and appropriate validation.
3. Open a pull request documenting changes, decisions, and validation results.
4. Integrate the completed milestone into `main` using a merge commit to
   preserve its individual commits.

`main` is intended to remain stable. Tests, race detection, benchmarks, and CI
will be introduced alongside the implementation they validate.
