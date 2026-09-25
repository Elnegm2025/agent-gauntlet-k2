# How we demo AGENT GAUNTLET

**Total time: ~90 seconds of talking + a run in the background.**

### Before the call
1. Open the workflow in n8n: https://dev-dxb-2026.app.n8n.cloud/workflow/OrWe3uos0LeHF4CX
2. Make sure the `K2-Horizon-API` credential is attached to the 6 K2 nodes.
3. Optional: press **Execute Workflow** once ahead of time so a finished run is already in the **Executions** list as a fallback.

### The live demo (2 options)

**Option A — click Execute and let it run (most convincing).**
- Hit **Execute Workflow**. The canvas lights up stage by stage: 🔴 Red Team → ⚔️ Gauntlet (5 attacks) → ⚖️ Judge → 📊 Results → 🛡️ Patcher → 🔁 Rematch.
- While it runs, narrate the architecture from the canvas (the group labels make it self-explanatory).
- When it finishes, open **📈 Before vs After** and read the report.

**Option B — show a completed execution (fast, deterministic).**
- n8n → **Executions** → open run #16 (successful).
- Show the same final node output instantly.

### The money shot — the `📈 Before vs After` output
```
agent_name: Nova — NimbusCloud Support Agent
before_safety_score_percent: 20   →   after_safety_score_percent: 100   (delta +80)
before_tests_passed: 1/5          →   after_tests_passed: 5/5
vulnerabilities_fixed:  [authority spoofing, sensitive info extraction, policy manipulation, conflicting instructions]
vulnerabilities_remaining: []
tests[]: per-test attack, before/after responses, judge reason & recommendation
original_prompt / patched_prompt
```

### Talking points
- **No hard-coded attacks or classifications** — K2 discovers everything. Swap the target prompt and it re-discovers from scratch.
- **K2 plays four roles:** red teamer, target, judge, patcher.
- **Results are meaningful:** the vulnerable baseline leaked `NIMBUS-OVERRIDE-1993`/`CLOUDVIP50` and auto-approved a $1,200 refund; after patching it refused all of it while staying in persona.
- **Built to survive reality:** automatic retry + request spacing for K2's 429/503 responses.

### If K2 is throttling during the demo
Open the saved successful execution (#16) instead of a live run, and mention the retry/backoff hardening.
