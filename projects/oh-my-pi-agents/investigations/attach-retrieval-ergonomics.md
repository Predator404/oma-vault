---
type: reference
created: 2026-08-19
tags: [reference, investigation, daemon, cli, ergonomics]
---

# Attach / response-retrieval ergonomics — no non-interactive way to get an entity's reply

## Source
- Repo / upstream: https://github.com/Predator404/oh-my-pi-agent (OMA — `oma` build)
- Docs: [[personas/phi/prime-agent/docs/daemon.md]] — daemon attach/reconnect model (Prime Agent lineage)
- Observed live: 2026-08-19 (this session)

## Symptom

Getting a one-shot answer out of a live persona (Phi, entity id
`874638fc-1f26-43d0-a7d1-9041e0ec9615`) from a **non-interactive** context (a
coding-agent driving `oma` over `bash`) took ~15 tool-steps and still surfaced
no reply. There is no clean request→response path outside the interactive TUI.
This is distinct from — and compounds with — the [[investigations/prompt-wake-gap.md]]
(the idle worker also never ran a turn, so even the fallback of reading the
transcript came up empty).

## Concrete gaps observed

1. **`oma entity prompt <id> "…"` is fire-and-forget.** It prints `Prompt sent`
   and returns; it never surfaces the assistant reply and offers no
   `--wait`/synchronous mode to block for one.
2. **Output is TUI-only.** The sole way to view an entity's output is
   `oma entity attach` (interactive). Nothing is pipeable — there is no
   `oma entity logs|transcript|tail <id> [--last N]` action that dumps the last
   N turns to stdout.
3. **`history://<id>` does not resolve entity sessions.** It errors
   `Unknown agent: <id>` and lists only the in-session `Main`; the internal-URL
   reader has no view onto daemon-resident entity transcripts.
4. **Transcript file is a hunt, not a lookup.** It is keyed under the active
   config root (`~/.oma`, not `~/.omp`, because the `oma` shim sets
   `PI_CONFIG_DIR=.oma`), inside a path-mangled dir
   (`agent/sessions/-Work-ClaudeEnvSetup-persistentAgents/`), and the session
   UUID is **not** in the filename — so "read the entity's last answer from
   disk" requires guessing the directory and matching by mtime.
5. **Name vs UUID inconsistency.** `oma entity prompt <name>` fails
   `unknown_session: No such session: <name>`; only the session UUID is
   accepted. `ps`, `show`, and `sessions` all accept the entity name — so the
   prompt/steer/follow-up path is the odd one out.

## Impact

The persona persists correctly across client exits, but is effectively only
reachable through the interactive TUI. Any automation, scripting, or
agent-to-agent use (a coding-agent consulting Phi mid-task) has no supported,
low-friction path to send a prompt and read the answer.

## Fix direction

- A synchronous `oma entity prompt --wait` (or a dedicated `ask`) that blocks
  for and prints the reply.
- A non-interactive `oma entity transcript|logs <id|name> [--last N] [--json]`.
- Resolve daemon-resident entity sessions through `history://` so the
  internal-URL reader can page their transcripts.
- Accept the entity **name** everywhere the session UUID is accepted
  (prompt/steer/follow-up), matching `ps`/`show`/`sessions`.

Same daemon-delivery domain as PR #1 and the prompt-wake gap; not in either.

## Links

- [[investigations/prompt-wake-gap.md]] — compounding functional defect (idle worker not woken)
- [[personas/phi/prime-agent/docs/daemon.md]] — daemon attach/reconnect model
