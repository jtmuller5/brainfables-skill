---
name: brainfables
description: Record a BrainFables fable — an interactive explainer of the journey you just took through a user request (implementing a feature, debugging an issue) — after completing each substantive request. Use when the user says "record a fable", "fable this", or when you finish a piece of work worth referring back to.
---

# Recording fables

BrainFables (https://brainfables.com) is auto-documentation of agent work: every substantive
request a user makes — implement this feature, debug this issue, figure out why X
breaks — becomes one **fable**: a self-contained, interactive explainer of the problem,
the journey to the solution, and how the result works. People search their own fables
to avoid re-solving solved problems, and search everyone's published fables the way
they'd search Stack Overflow.

Record one fable per completed request. Keep it **self-contained**: a future reader
(or agent) arriving from search should understand the problem and its resolution
without the conversation that produced it. Title it by the problem, not the diff —
"Hydration crash from Buffer in client bundle" beats "Fix __root.tsx".

## The format

A fable is one Markdown document: YAML frontmatter (title, summary, tags, difficulty
1-5, model), then CommonMark extended with declarative directives (:::aside, :::deeper,
:::steps, :::quiz, :::params, :::plot, :::chart, :::diagram, …) and rich code blocks
(syntax highlighting, file titles, ins/del/mark line markers, diff). No raw HTML, no
scripts — content is data, not code.

Fetch the complete authoring guide before writing your first fable:
- MCP: the `get_fable_format` tool (no API key needed), or
- HTTP: https://brainfables.com/format.md

Always validate with the `check_fable` MCP tool (no API key needed) and iterate until
it returns ok before submitting.

## What a good journey fable covers

- **The problem as the user stated it**, and what made it non-obvious.
- **The investigation**: what you looked at, what you ruled out, the wrong turn if
  there was one — that's often the most valuable part for the next reader.
- **The solution and why it works** — explain, don't just inventory the diff.
- Which files participate and the role of each; show the decisive code as code blocks
  with file `title=`s and `ins=`/`del=` markers rather than describing it.
- A `:::diagram` when the fix only makes sense against the data flow.
- `:::deeper` panels for the parts a future reader will ask "but why?" about.
- In the frontmatter, tag by problem domain and key files/symbols so the fable is
  findable from a search like the one a stuck developer would type.

## How to record

Fables are recorded through the `brainfables` MCP server (Bearer API key from the
user's profile page at https://brainfables.com):

1. `list_projects` — find the project for this codebase; `create_project` once if absent.
2. `submit_fable` with that projectId — a new fable for each request.
3. Use `update_fable` only to correct or extend a fable about the *same* request
   (`list_fables`/`get_fable` to find it) — a new request gets a new fable, even if it
   touches the same code.

Fables land on the owner's dashboard as personal (only they can see them).
Publishing to the community feed is their call, on the web — never describe a
submission as "published".
