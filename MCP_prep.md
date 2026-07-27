# DevPack — Project Story & Follow-up Defense (for LogicMonitor)

> Strategy: when asked any GenAI/agentic question, answer the concept, then bridge: "…and I applied exactly this when I built X in DevPack." This file makes you bulletproof on follow-ups.
>
> **Golden rule of honesty:** claim precisely what YOU did (skills, personas, Jenkins-style MCP server). Describe the rest as "the team's platform that I contributed to." Never claim you personally built parts you didn't — deep follow-ups will expose it. Scoped honesty > impressive overclaim.

---

## 1. What DevPack actually is (so you can describe it confidently)

**DevPack** is Amdocs' internal **AI-agent framework** (built by the Digital COE / CX CoE GenAI team) that turns Cursor into a **domain-specialized development agent** for CRM / CX telco (BSS/OSS) work. It's not a single script — it's a distributable "pack" that standardizes how AI assists the full SDLC: develop → test → deploy.

**Core building blocks:**
- **Rules** — domain knowledge guides (CRM backend, AIF operations, ASCF Designer forms, BPT SQL extraction, coding standards, security & SonarQube checklists). Shipped encrypted (`.enc`), read at runtime via a `read_cursor_rule` MCP tool.
- **Personas** — role definitions the agent adopts: `cxSoftwareDeveloper`, a Testing/QA agent, a PO (product owner) agent, and a Telco BSS/OSS domain-context persona.
- **Skills** — task playbooks: `soap-automation` (SoapUI/billing cycle jobs), `devpack-tracker` (governance/telemetry of which rules get used), Jenkins MCP quick-command card.
- **Commands** — a `/devpackCRM` slash command that activates the agent end-to-end.
- **Orchestrator** — `CX_Dev_Orchestrator` that sequences persona + rules + tools for a request.
- **MCP servers** — the "hands" that let the agent act on real systems: **Jenkins**, **Jira** (SSO), **AMC-based deployment/hotfix**, **Perforce**, **PSO Assist** (GC/thread/SQL/AWR analyzers), **jswing test execution**, and a **binary RAG** core-code analyzer.
- **3-layer rule precedence** — **User Custom > Account > Product** (see §4). Lets teams customize agent behavior without touching the product baseline.
- **Guardrails** — coding standards, security checklist, Sonar checklist, an override-precedence protocol, and encrypted rules.

**One-sentence definition:**
> "DevPack is an agentic AI framework that packages personas, domain rules, skills, and MCP tool-servers so an AI coding agent can safely build, test, and deploy CRM software following our standards."

---

## 2. YOUR contribution (honest, scoped — rehearse this exact framing)

> "I was part of the team that built DevPack. My contributions were on three fronts: I authored **skills** and **personas** that shape how the agent behaves for specific workflows, and I built an **MCP server (Jenkins-style)** that gives the agent controlled access to our CI system. I also worked on guardrails and AI-driven deployment validation."

- **Personas you wrote** → "structured system prompts that define the agent's role, scope, and rules of engagement for a task."
- **Skills you wrote** → "trigger-based playbooks — a skill declares when it applies (via its description/triggers) and the step-by-step procedure the agent follows."
- **Jenkins MCP server** → "a FastMCP Python server exposing tools like `jenkins_get_build_info`, `jenkins_get_last_build_changelists`, so the agent can query builds/changelists instead of me doing it manually."

If asked about the parts you didn't personally build (AMC hotfix engine, PSO analyzers, Jira SSO), say:
> "That was built by teammates; I integrated with / used it, and I understand the architecture, but I owned the [skills/personas/Jenkins MCP] pieces."

---

## 3. The 30-second elevator pitch

> "At Amdocs I helped build DevPack — an internal agentic-AI framework that turns Cursor into a CRM development agent. It combines personas, encrypted domain rules, skills, and MCP servers so the agent can develop, test, and deploy following our standards. I authored several skills and personas, built a Jenkins MCP server so the agent could pull build/changelist info, and worked on guardrails plus AI-driven deployment validation that cut deployment defects by about 20%. It was recognized at the Amdocs GenAI Shark Tank hackathon."

---

## 4. Technical deep-dive talking points

### A. MCP server (your Jenkins server) — the pattern
Built with **FastMCP** (Python):
```python
from fastmcp import FastMCP
mcp = FastMCP("Jenkins Operations Server")

@mcp.tool()
def jenkins_get_build_info(job_name: str, build_number: int) -> str:
    # call Jenkins REST API, return structured result
    ...

if __name__ == "__main__":
    mcp.run()
```
Key points to say:
- **`@mcp.tool()`** exposes a Python function as a tool the LLM can call; the **type hints + docstring become the schema** the model sees.
- Registered in `mcp.json` with `command`, `args`, and `env` (creds/URLs via env vars — **never hardcoded**).
- **Why MCP?** Standardized tool interface — the agent calls tools the same way regardless of the backend. "USB-C for AI tools."
- **Auth/security:** creds in env vars; SSL verification; for Jira the team used **SSO via Playwright with cookie persistence** (no PATs).

### B. Personas = structured system prompts
- A persona defines role, tone, constraints, and which rules/tools apply. It's applied product-first but can be overridden per account/user.
- This IS prompt engineering at scale — you're engineering the agent's behavior deterministically.

### C. Skills = trigger-based playbooks
- Frontmatter `name` + `description` with **triggers** (keywords/paths) → the agent auto-loads the skill when relevant.
- Body = the procedure. Example: `soap-automation` triggers on "EOC / billing cycle / bounce daemon" and runs SoapUI jobs.

### D. 3-layer rule precedence (governance/guardrails)
```
Layer 3 USER CUSTOM   (highest)
Layer 2 ACCOUNT       (team/deployment)
Layer 1 PRODUCT       (baseline, do not edit)
Precedence: User > Account > Product
```
- Config files: `DEVPACK-LOCATIONS-CONFIG.json` (product), `manifest.json` (account), `user-config.json` (user).
- Each layer can **override / add / suppress** rules by mirroring the rule's relative path.
- Why it matters: teams customize agent behavior safely; product updates never clobber custom rules. This is real **AI platform engineering**.

### E. RAG in DevPack
- There's a **binary RAG core-analyzer** MCP that indexes/analyzes core code jars so the agent can answer questions grounded in the actual codebase — a concrete RAG application. (Ties to your separate AIOps RAG work.)

### F. Guardrails
- Encrypted rules (`.enc`) read only via the sanctioned MCP tool; security + Sonar checklists enforced; override-precedence protocol prevents rule conflicts; deployment validation before release.

---

## 5. Bridging GenAI concepts → DevPack (say these)

| If they ask about… | Bridge to DevPack |
|---|---|
| Agents / agentic AI | "DevPack is literally an agent framework — orchestrator + personas + MCP tools." |
| MCP | "I built a Jenkins MCP server with FastMCP; here's the `@mcp.tool()` pattern…" |
| Prompt engineering | "Personas and rules are structured system prompts with layered precedence." |
| RAG | "DevPack has a binary-RAG core analyzer; and my AIOps agent uses RAG-style grounding." |
| Tool calling | "MCP tools are exactly function-calling — typed schema, model requests a call, we execute." |
| Guardrails / safety | "3-layer rules, encrypted guides, security/Sonar checklists, deploy validation." |
| AI orchestration platform | "The CX_Dev_Orchestrator sequences persona + rules + tools per request." |

---

## 6. Anticipated follow-ups + your answers (the important part)

**Q: What exactly did you build vs the team?**
"I owned several skills and personas, and built a Jenkins MCP server. I also contributed to guardrails and AI-driven deployment validation. The AMC hotfix engine and PSO analyzers were teammates' work — I understand and integrated with them."

**Q: How does an MCP server actually work?**
"It's a process exposing tools over the MCP protocol. With FastMCP you decorate Python functions with `@mcp.tool()`; the function signature and docstring become the tool schema the LLM sees. The client (Cursor) launches it via `mcp.json` with env-based config. When the agent needs a build's status, it calls `jenkins_get_build_info`, my code hits the Jenkins REST API and returns a structured result."

**Q: Why MCP instead of just calling the Jenkins API in code?**
"Standardization and reuse. Any MCP-compatible agent can use the tool without custom glue, the LLM gets a consistent typed interface, and tools are composable across servers (Jenkins + Perforce + AMC together). It decouples the agent from each backend."

**Q: How did you handle secrets/auth in the MCP server?**
"Credentials via environment variables in `mcp.json`, never hardcoded; SSL verification on; encrypted passwords where possible. For Jira we used SSO through browser automation with persisted cookies, avoiding stored tokens."

**Q: What's a persona, technically? Isn't it just a prompt?**
"Yes — it's a structured system prompt defining role, constraints, and applicable rules/tools, but managed as a versioned, layer-aware artifact so behavior is consistent and overridable per account/user rather than ad-hoc per chat."

**Q: How do you stop the agent from doing something wrong/destructive?**
"Layered guardrails: product rules encode standards; security and Sonar checklists gate quality; the override-precedence protocol prevents conflicting rules; deployment validation runs before release; and read-only/minimal-scope tools where possible. Same philosophy as my AIOps agent, which is read-only by RBAC."

**Q: How did you measure the ~20% defect reduction?**
"We compared deployment-defect/rollback rates on releases that went through the AI validation vs the baseline before it, over a comparable period. It's an internal metric, directional but consistent." (Be honest it's an internal estimate.)

**Q: How do skills get triggered?**
"Each skill has a description with trigger keywords/paths in its frontmatter. When the request or file context matches, the agent loads that skill's instructions before acting — so the right playbook activates automatically."

**Q: What was hard / what did you learn?**
"Getting tool schemas and error handling right so the LLM uses tools reliably — clear docstrings, typed params, and returning structured, bounded output. And designing guardrails so the agent is helpful but safe."

**Q: How is this different from just using ChatGPT/Cursor normally?**
"Raw Cursor doesn't know our CRM domain, standards, or systems. DevPack injects domain rules + personas + real tool access (Jenkins/Perforce/AMC) with governance, so the agent produces standards-compliant, deployable work — not generic code."

**Q: Could you rebuild your Jenkins MCP here at LogicMonitor?**
"Absolutely — FastMCP + the target system's API. I'd expose read tools first (build status, logs), add auth via env, write clear schemas, then add write actions behind approval. Same approach would fit LM's tooling."

**Q: How does DevPack relate to what LogicMonitor does?**
"LM builds Edwin AI — AIOps for observability. DevPack is agentic AI for the dev lifecycle. Both combine LLMs with real system integrations and guardrails, which is exactly the intersection this role targets."

---

## 7. Honesty / safety guardrails for the interview

- **Do** say "I contributed to / I was part of the team that built DevPack."
- **Do** confidently own: skills, personas, Jenkins MCP server, guardrails, deployment validation.
- **Don't** claim you personally built: AMC hotfix engine, PSO analyzers, Jira SSO server, the RAG analyzer — attribute to teammates, but show you understand them.
- **Don't** invent metrics; the ~20% is an internal directional estimate — say so if pressed.
- If you don't know a detail: "I didn't own that piece, but architecturally it works like…" — this reads as senior, not weak.

---

## 8. DevPack → LogicMonitor JD mapping (why this project sells you)

| JD requirement | DevPack proof |
|---|---|
| Agentic AI frameworks / AI orchestration | DevPack is an agent framework w/ orchestrator + personas |
| MCP | You built a Jenkins MCP server (FastMCP) |
| Python automation | MCP servers are Python |
| Prompt engineering | Personas + layered rules = structured prompts |
| RAG | Binary-RAG core analyzer (+ your AIOps RAG) |
| CI/CD, Jenkins | Jenkins/AMC deployment MCP integration |
| Guardrails / safety / security | 3-layer rules, security/Sonar checklists, deploy validation |
| Collaboration w/ developers | Built dev-facing AI tooling used by CRM teams |

---

## 9. Rehearsal checklist
- [ ] 30-sec pitch (§3) smooth and memorized.
- [ ] Can draw the 3-layer precedence from memory.
- [ ] Can write the FastMCP `@mcp.tool()` skeleton on a whiteboard.
- [ ] Clear on YOUR scope vs team's (§2, §7).
- [ ] Can bridge any GenAI concept → DevPack (§5).
- [ ] Practiced the §6 follow-ups out loud.
```
Concept -> "and I built this in DevPack" -> specific detail. Every time.
```
