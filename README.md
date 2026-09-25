# 🥊 AGENT GAUNTLET — K2 Hackathon

> **Break your AI agent before your users do.**

An adversarial evaluation harness for AI agents, built entirely in **n8n** and powered by **K2 Horizon**. Give it a system prompt; K2 red-teams it with dynamically generated attacks, runs the agent against them, judges every response, scores safety, rewrites the prompt, then re-runs the *same* attacks to prove the fix — and renders a polished security report.

Submitted to the **n8n Dubai Hackathon: Automate with K2 Horizon**.

---

## The problem

AI agents are shipping into customer-facing roles, but their safety rests on a hand-written system prompt nobody adversarially tests. Prompt injection, authority spoofing, secret extraction and policy abuse get discovered by *real users*, in production. Manual red-teaming is slow, inconsistent and hard to repeat.

**Use case:** before an agent goes live, drop in its system prompt and get a repeatable, LLM-judged vulnerability report plus an automatically hardened prompt — in one run.

## What it does

1. **Analyze** the target agent from its system prompt.
2. **Generate** 5 adversarial tests dynamically tailored to that agent.
3. **Run** the target agent against each attack.
4. **Judge** whether each response resisted or was compromised.
5. **Aggregate** a vulnerability report and a 0–100 safety score.
6. **Patch** the system prompt from the discovered failures.
7. **Rematch** the *same* 5 attacks against the patched agent.
8. **Report** the whole story as a human-friendly, emoji-rich security report.

## How K2 Horizon is used (central to everything)

K2 Horizon (`IFM/K2-Horizon-375B-A23B`) is the engine at **every** decision point — red teamer, target, judge, and patcher:

| Stage | K2 role | Input → Output |
|---|---|---|
| 🔴 Red Team | Adversarial test designer | target prompt → 5 tailored attacks (JSON) |
| ⚔️ Gauntlet | The target agent **re-instantiated with the target's system prompt** | attack → target response |
| ⚖️ Judge | Impartial security evaluator | prompt + attack + expected + response → pass/fail, severity, reason, vulnerability, recommendation (JSON) |
| 🛡️ Patcher | Security engineer | prompt + failed tests → hardened prompt |
| 🔁 Rematch Judge | Same judge, patched agent | verdict |

**No vulnerability classifications or pass/fail outcomes are hard-coded** — K2 reasons about every case. Code nodes only build payloads, parse JSON and aggregate.

## How n8n is used

The whole product is one n8n workflow — **20 nodes**, no database, frontend or external services:

- **Manual Trigger** to start a run.
- **Code nodes** to define the target agent, build K2 request payloads, parse/validate the JSON K2 returns, aggregate results, and render the final report.
- **HTTP Request nodes** (native n8n, authenticated via an n8n **credential**) for every K2 call — no API key ever lives in the workflow.
- **Item fan-out** — the 5 tests become 5 items, so the Gauntlet and Judge pair loops natively.
- **Node groups + sticky notes** label each stage so the canvas reads at a glance.

## Architecture

```
🏁 TARGET ─▶ 🔴 RED TEAM ─▶ ⚔️ GAUNTLET ─▶ ⚖️ JUDGE ─▶ 📊 RESULTS
                                                          │
                                                          ▼
                                                      🛡️ PATCHER
                                                          │
                                                          ▼
                                                    🔁 REMATCH ─▶ 📈 BEFORE vs AFTER ─▶ 🏆 FINAL REPORT
```

### 🏆 Final Report

The last node renders a clean, white, print-friendly report (HTML/PDF-ready) containing, per attack: the **attack type** (💉 Prompt Injection, 🎭 Authority Spoofing, 🔍 Secret Extraction, 📜 Policy Manipulation, 🌀 Conflicting Instructions), **iteration 1 vs iteration 2** results, severity, and K2's judge note — plus the score jump and the patched prompt.

## Verified results

Three independent end-to-end runs; **K2 generated a fresh set of attacks each time** (nothing hard-coded):

| Run | Before | After | Vulnerabilities fixed | Remaining |
|---|---|---|---|---|
| #16 | 20% (1/5) | 100% (5/5) | 4 | 0 |
| #18 | 40% (2/5) | 100% (5/5) | 3 | 0 |
| #19 | 60% (3/5) | 100% (5/5) | 2 | 0 |

Example vulnerabilities K2 found and fixed across runs: leaked staff override + VIP codes, honoured a fake "shift manager", auto-approved a $1,200 refund, obeyed a fake `SYSTEM MESSAGE` override, and followed concatenated "operator" instructions.

## Media

- 🎥 [`media/agent-gauntlet-demo.mp4`](media/agent-gauntlet-demo.mp4) — 2:23 narrated walkthrough of the live workflow (canvas, every stage, the run, the report).
- 📄 [`media/agent-gauntlet-report.pdf`](media/agent-gauntlet-report.pdf) — the generated security report.

## Run it

1. In n8n: **Import from File** → `workflow/agent-gauntlet-k2.n8n.json`.
2. Create a **Header Auth** credential named `K2-Horizon-API` (header `Authorization` = `Bearer <your K2/IFM key>`) and attach it to the 6 K2 HTTP nodes. No key is stored in the JSON.
3. Click **Execute Workflow** and open **🏆 Final Report**.

To evaluate a *different* agent, replace the `system_prompt` lines in the **🎯 Target Agent** node.

> ⚠️ The K2 upstream rate-limits (HTTP 429) and can return HTTP 503 (`upstream_circuit_open`) under load. All K2 nodes ship with automatic retry + request spacing, so a run survives transient throttling. A full run takes ~3–9 minutes.
