---
description: Start a new SDD change with interactive brainstorming — runs brainstorm dialogue then creates a proposal
argument-hint: <change-name>
---

Follow the SDD orchestrator workflow for starting a new change named "$ARGUMENTS" with brainstorming.

WORKFLOW:
1. Launch sdd-brainstorm sub-agent with the change description
2. Handle NEEDS_CONTEXT relay loop: if sdd-brainstorm returns NEEDS_CONTEXT, present the question to the user, collect the answer, and re-launch sdd-brainstorm with the accumulated context (max 5 iterations)
3. Once brainstorm returns DONE or DONE_WITH_CONCERNS, launch sdd-propose sub-agent (sdd-propose will read the brainstorm artifact from engram automatically)
4. Present the proposal summary and ask the user if they want to continue with specs and design

CONTEXT:
- Working directory: !`echo -n "$(pwd)"`
- Current project: !`echo -n "$(basename $(pwd))"`
- Change name: $ARGUMENTS
- Artifact store mode: engram

ENGRAM NOTE:
Sub-agents handle persistence automatically. Each phase saves its artifact to engram with topic_key "sdd/$ARGUMENTS/{type}".
The brainstorm artifact is saved as "sdd/$ARGUMENTS/brainstorm".

Read the orchestrator instructions in CLAUDE.md to coordinate this workflow. Do NOT execute phase work inline — delegate to sub-agents via the Task tool.
