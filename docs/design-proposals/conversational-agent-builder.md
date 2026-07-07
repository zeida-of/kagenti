# Conversational Agent Builder for Kagenti — Proposal

**Status:** Draft for review — seeking feedback on the direction

**Date:** 2026-07-07

**Author:** Zeid Houweling

---

I became interested in Kagenti while evaluating platforms for centralized governance, deployment,
observability, and lifecycle management of AI agents. 
The creation paths, however, all assume someone technical: the code frameworks Kagenti integrates with (LangGraph, CrewAI) serve developers,
and visual low-code tools, while useful, generate flow graphs that are difficult to review, diff, test, and promote through GitOps-style production controls. 
What I couldn't find was a way for the people who know what an agent should do — but don't write "agent code" — to author one safely.

Kagenti governs agents once they exist, but the current creation path still assumes a developer or platform engineer. 
A Conversational Agent Builder would let domain experts safely produce reviewable, 
versioned agent definitions without bypassing Kagenti’s security model.
In regulated domains such as finance, the people who understand the rules, exceptions,
and customer-facing behavior are often not the same people who can author LangGraph code, Kubernetes manifests, or MCP integration code.

> This proposal was partly inspired by seeing how effectively business specialists in the FinTech industry could tune an agent when given an OpenClaw workspace with access to relevant knowledge sources and APIs.

This proposal adds a **creation plane** to Kagenti for the declarative subset of agents:
(system prompt + skills + MCP tools + knowledge). 
I'm proposing a new CR: `AgentBuilder` which would get reconciled by a flag-gated controller in the existing operator.
The CR would deploy a **conversational builder web app**. The Web App 
essentially becomes the users Agent IDE, where the user can describe the agent in chat; 
the builder edits a validated spec, deploys previews, and promotes changes **as pull requests** to a git repository the
Platform & AI Teams own. With a Git-based flow we can streamline CI and evals. 
Merge to main is the only path to production. The Agents authored by the users are ordinary Kagenti workloads 
(`AgentRuntime`-enrolled, existing skills and MCP conventions) — no new runtime primitives.

Kagenti manages the full lifecycle of agents that already exist — enrollment,
identity, discovery, observability, and an ops console to deploy,
test, and monitor them. This is an addition to give users a lower easy barrier of entry, to use, 
update and communicate their wants, through the very same platform that takes their agent to production. 
The Conversational Agent Builder extends the End User Persona (the domain expert) to author AI Agents via a controlled flow.

I have a working prototype that runs on a kind cluster with Kagenti and GitHub integration. A demo is available on request.
The Builder Agent and user created agents themselves run on the **Kagenti ADK** (`kagenti-adk`) on the same unified Runtime.
I’d also be happy to join one of your meetups to discuss the proposal.


## User Experience with simple example

1. **Sign in**: With Git Provider (example: a GitHub App the platform team installs on the
   agents repository). Every user works in their own workspace; sessions
   map to branches (`session/<login>/<agent>`).
2. **Talk**: The user can start building an agent by prompting: 
   "I want an agent that helps customers check stock and always
   quotes prices in euros." The builder chat edits a validated draft spec
   via tools — it can only claim what tool results confirm.
3. **Test**: a preview deployment runs in the namespace; the user chats
   with the *deployed* agent side by side and gives feedback, for more deterministic 
   results, the builder can turn certain requests into skills.
4. **Submit**: the builder opens a pull request. The diff is human-readable
   (`agent.json`, prompt/skill files, workflow scripts, generated
   changelog). Reviewers are the platform/team owners of the repo.
5. **Merge = production**: main is rendered and applied via GitOps (ArgoCD/Flux); 
   the agent is a normal `AgentRuntime`-enrolled workload. 
   History, rollback, and audit are git. 


## Non-Goals

The Conversational Agent Builder does not:

- author arbitrary coded agents (LangGraph/CrewAI/AG2) source builds keep
  their existing Shipwright path;
- replace or overlap ui-v2;
- implement a new operator — the controller lives in the existing manager;
- implement agent marketplace/sharing features;

Let me know what you think of the idea, and how you think it fits into the Kagenti ecosystem!
Do we want an authoring-plane component with Kagenti, or should we keep this external?
Happy to answer any questions or elaborate more on design choices :) 
