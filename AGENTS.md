# AGENTS.md — vault working rules

This is an **Obsidian vault** (the OMA registry vault). Anything working inside it
— personas, agents, skills, scripts — MUST honor the note conventions below when
creating or editing notes.

## Note conventions (honor these)

Notes are organized into areas, each with a **Templater folder template** that
defines its frontmatter + section skeleton:

| Area | Template |
|------|----------|
| `agents/` | `Templates/Agent Note.md` |
| `personas/` | `Templates/Persona Note.md` |
| `projects/` | `Templates/Project Note.md` |
| `reference/` | `Templates/Reference Doc.md` |

Rules:

1. **Place a new note in the correct area folder.** Creating an empty note there
   auto-applies the folder template (Templater; requires the in-app "Trigger
   Templater on new file creation" toggle).
2. **When creating a note programmatically** (via the `write` tool, a shell copy,
   or vault MCP — Templater does not run for these), **seed it from the matching
   `Templates/<Template>.md`**: copy its frontmatter + skeleton, resolve the
   placeholders (`tp.file.title` → title, the `tp.date`/`tp.file.creation_date`
   calls → today's date, drop `tp.file.cursor()`), and fill the area frontmatter.
3. **Keep claims source-grounded** — cite the exact file, type, or contract.
4. **Do not template the verbatim doc mirrors** (`personas/phi/{oh-my-pi,prime-agent,pi,oh-my-pi-agent}/`,
   `reference/obsidian/templater/`). They are copied-in with content and are
   excluded from folder templating.
5. **Capture PR follow-ups.** Every project keeps
   `projects/<project>/follow-ups.md`; when a PR is merged, record its
   outstanding follow-ups there before treating it as done. Applies to every
   project — see the register convention in the conventions doc.

Full detail, the `tp.*` syntax reference, and the mirrored Templater docs:
[reference/obsidian/vault-conventions.md](reference/obsidian/vault-conventions.md).
