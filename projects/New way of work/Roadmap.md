---
type: project-note
project: New way of work
persona:
created: 2026-08-21
tags:
  - project
---

# Untitled

> Area `projects/<project>/<persona>.md` — this entity's single source of truth for the project. Never copied into the project repo; surfaced via a project-pointer skill.

## Goal
Refining a modern agentic development workflow

## Knowledge & decisions
- Using omp - harness has several features that claude cli does not have. Including proper debugging support (live debugger).
- Decided to fork omp -> oma (Oh-my-pi-agent) taking existing logic for persistent agent sessions from Prime Agent 
	- Still need to get context as a variable
- Want to use sandcastle for worktree and agent isoloation
- Have added Phi as a persistent agent to assist the user with refining workflow and assisting with the oma environment as a whole
- Using Obsidian for graph-based knowledge base, using a plugin (insert name here) for semantic search
- Poteto/Matt Pocock/David Andrej(very selectively) skills audited and imported and integrated into persistent personas/agents as appropriate. Initially found MPs skill setup too restrictive but some of the skills are truly ground-breaking and should not be dismissed. Need to allow end users to take primitive skills and combine them into user-specific workflows. But I suspect that the primitives(to a greater or lesser degree) will be somewhat consistent. 
- Have forked T3code to add support for omp and oma to better support multiple tasks concurrently. 
- Possibility of using whisperflow to streamline interactions with ai.

* Listing of attached persistent agents as well as whether the daemon is up and running to be added to oma welcome screen
## Open questions
-

## PR follow-ups
- Outstanding follow-ups from merged PRs live in `follow-ups.md` (this project
  folder), recorded at merge — see the vault convention.

## Links
-
