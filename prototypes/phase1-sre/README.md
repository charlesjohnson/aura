# Phase 1 — Default SRE Orchestration (prototype)

A vendor-neutral, opinionated SRE agent harness for AURA, intended to ship as an out-of-the-box
default config. **This is a prototype for review, not final.**

## What it is

`default-sre-orchestration.toml` is a generic SRE orchestration config derived from a production
7-worker incident-response config (Mezmo's internal `sre-agent-config`). The high-value parts — the
**coordinator decomposition** and the **per-worker prompts** — are real production context
engineering; we vendor-neutralized them off the Mezmo specifics.

**Roster:** a coordinator + 5 enabled workers — `incident-responder`, `metrics-analyst`,
`log-analyst`, `k8s-engineer`, `runbook-engineer` (the only writer) — plus a commented `service-ops`
stub (the slot for a customer's own primary-platform tooling, which is bespoke per shop).

## The two-phase plan

- **Phase 1 (this file): prompts + decomposition, no MCPs.** Workers ship as prompt/decomposition
  skeletons — no `[mcp.servers]`, no `mcp_filter`. The value is the best-practice structure; it runs,
  but reasons from general knowledge until tools are added.
- **Phase 2 (next): wire MCPs.** Help users attach MCP servers to the default workers (each worker
  gains its `[mcp.servers.*]` + allowlist), stand up the shared investigation memory
  (`basic-memory`) and a knowledge base, and map the generic workers onto their bespoke stack —
  starting every customer from this common best-practice baseline.

## Status

- ✅ **Validated** against AURA's config loader (`aura_config::load_config` — env resolution + parse
  + schema + worker-name checks). Parses and passes every validation gate.
- ⏳ **Not yet run** against a live LLM (no provider creds in the dev env). The open question Phase 1
  is meant to answer: does the coordinator route/decompose sensibly even tool-less?

## Run it

```bash
export LLM_PROVIDER=openai LLM_MODEL=gpt-4o LLM_API_KEY=sk-...   # your real key
AURA_CUSTOM_EVENTS=true cargo run -p aura-cli --features standalone-cli -- \
  --standalone --config prototypes/phase1-sre/default-sre-orchestration.toml
```

Then try an incident-shaped prompt and watch the routing, e.g.:

> Incident: `[prod-use1][Kafka]` consumer-group lag over 2M on the `payments` consumer group; pods
> may be unhealthy. Investigate and give me a status.

Expect: `incident-responder` first (creates the investigation record), then parallel
`metrics-analyst` + `k8s-engineer`, then `log-analyst` if things look unhealthy, synthesized in the
🔴🟡🟢 format. With no MCPs, you're judging **routing and structure**, not data.

## History / why this shape

This replaces an earlier `/init` "guided discovery" design (auto-detect cloud inventory → propose a
plan → wire MCP servers). We dropped it: it drifted from the goal and over-engineered around the
verbose `mcp_filter` allowlists. We now **accept the verbose allowlists** and lead with the
high-value prompts/decomposition instead.
