# Hi, I'm XiaoLinYa 👋

I build **LLM routing infrastructure** — the unglamorous layer that keeps multi-upstream AI gateways
from silently failing.

## What I work on

Most of my code lives in private repos running in production. The theme is consistent:
**when you fan out requests across dozens of providers, the hard part is not the happy path — it's
detecting the failure modes that still return HTTP 200.**

Concrete things I've built and run:

| Project | What it solves |
|---|---|
| **XLauto** (Go, ~1.2k LoC) | Multi-upstream LLM router. Fault isolation at **(upstream × model)** granularity — a common design isolates by upstream *or* by model; this isolates every combination independently, each with its own ok/fail/empty/timeout counters, exponential backoff, and concurrency cap. Four-stage fallback: ordered chain → race group → fallback chain → final retry. |
| **ops-console** (Python, public) | Operational control plane for LLM relay stacks. Read-only audit → whitelisted execution → rollback → full evidence trail. |
| **WB Hub limit tracker** (Python) | Empirical study: 44,193 calls / 7B tokens over 23 days across 17 accounts to reverse-engineer per-account daily limits. Result: the limit is **token-based (200M/account/day)**, not request-count-based — verified against 44 rate-limit events, all landing in the 132.5M–199.6M band. |
| **Seckill line guard** (Python) | Classifies upstream health into ALIVE / EXHAUSTED / RATE-LIMITED / AUTH-FAILED / DOWN so one-off trial endpoints can be cleaned up safely instead of silently rotting. |

## Engineering positions I actually hold

**1. "HTTP 200" is not success.**
An agent-style backend with `maxTurns: 1` will accept a request that carries `tools`, try to call one,
fail to complete the turn, and return 200 with an empty body. Any client that only checks status codes
sees a hang. I peek the first stream chunk for `content` / `reasoning_content` / `tool_calls` and
re-route when all three are empty.

**2. Client cancellation must not count as upstream failure.**
Treating `context.Canceled` as a failure trips the circuit breaker on a perfectly healthy upstream.
This is a one-line distinction with outsized consequences.

**3. Buffering is not streaming.**
A proxy that waits for the full response and then slices it into SSE frames has a first-byte time equal
to total generation time. Clients time out. The fix is not "more timeout" — it's emitting a real SSE
head plus keepalives before the upstream answers.

**4. Configuration drift is the default state.**
A process that started before your edit is running the old code. Comparing *process creation time*
against *file mtime* catches this in one second; `curl /health` returns 200 either way.

## Currently

Running a self-hosted relay stack: New API + PostgreSQL, a Go gateway, a Qoder account pool,
and a CloudBase-backed proxy for Tencent Hunyuan / DeepSeek models.

## Reach me

Open an issue on [ops-console](https://github.com/qweqqqqqqq/ops-console) — that's the public surface.
