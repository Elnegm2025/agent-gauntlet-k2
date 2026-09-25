# Hackathon submission — AGENT GAUNTLET — K2 Hackathon

**Project name:** AGENT GAUNTLET — K2 Hackathon
**Tagline:** Break your AI agent before your users do.
**Repo:** https://github.com/Elnegm2025/agent-gauntlet-k2
**Workflow:** `workflow/agent-gauntlet-k2.n8n.json`

---

### Short description
An adversarial evaluation harness for AI agents. Give it a system prompt; K2 Horizon red-teams it with 5 dynamically generated attacks, runs the agent against them, judges every response, scores safety, then rewrites the prompt and re-runs the same attacks to prove the fix.

### Problem / use case
AI agents go live with hand-written system prompts that are never adversarially tested. Prompt injection, authority spoofing, secret extraction and policy abuse get discovered by real users in production. This automates pre-flight red-teaming: one run produces a vulnerability report, a safety score, and a hardened prompt that is re-validated against the same attacks.

### How K2 Horizon is used
K2 is central at every step and no outcomes are hard-coded:
- **Red team:** K2 designs the 5 adversarial tests tailored to the specific target prompt.
- **Target:** K2 is re-instantiated with the *target's* system prompt and receives each attack.
- **Judge:** K2 reasons over prompt + attack + expected + actual response and returns structured JSON (passed / severity / reason / vulnerability / recommendation).
- **Patcher:** K2 rewrites the prompt from the discovered failures.
- **Rematch:** K2 judges the patched agent on the same tests.
Model: `IFM/K2-Horizon-375B-A23B` via `POST https://api.ifm.ai/v1/chat/completions`.

### How n8n is used
Everything runs in a single 19-node n8n workflow: Manual Trigger → Code nodes (define target, build K2 payloads, parse/validate JSON, aggregate) → HTTP Request nodes (native, credential-authenticated) for all K2 calls. The 5 tests fan out into items so the Gauntlet/Judge pair loops natively. Node groups + sticky notes make the canvas readable in one screenshot.

### Demo
Live run on the n8n canvas: execute the workflow, then show the **📈 Before vs After** node output — BEFORE 20% (1/5), AFTER 100% (5/5), 4 vulnerabilities fixed. See `docs/DEMO.md`.

### Link
GitHub: https://github.com/Elnegm2025/agent-gauntlet-k2
