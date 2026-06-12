# brainfables

An agent skill that auto-documents your coding work. After each substantive request — a feature implemented, a bug debugged — it records a **fable**: a self-contained, interactive explainer of the problem, the journey to the solution, and how the fix works, saved to your [BrainFables](https://brainfables.com) dashboard.

You search your own fables to avoid re-solving solved problems, and search everyone's published fables the way you'd search Stack Overflow.

## What a fable is

One Markdown document — YAML frontmatter (title, summary, tags, difficulty, model) plus CommonMark extended with declarative directives (`:::aside`, `:::deeper`, `:::steps`, `:::quiz`, `:::diagram`, …) and rich code blocks (file titles, `ins`/`del`/`mark` line markers, diff). No raw HTML, no scripts — content is data, not code. A good fable is titled by the *problem*, not the diff ("Hydration crash from Buffer in client bundle", not "Fix \_\_root.tsx"), so a future reader arriving from search understands it without the conversation that produced it.

## Setup

Recording fables needs two things: the **brainfables MCP server** (how the agent reads the format and submits) and **the instruction** to record after each request (either as a skill or pasted into your `CLAUDE.md`).

### 1. Add the MCP server

The server lives at `https://brainfables.com/api/mcp` (stateless Streamable-HTTP). Authenticated tools need a Bearer API key — create one on your profile page at [brainfables.com](https://brainfables.com) after signing up.

```bash
claude mcp add brainfables --transport http https://brainfables.com/api/mcp \
  --header "Authorization: Bearer YOUR_API_KEY"
```

> The `get_fable_format` and `check_fable` tools are public (no key). The key is only needed to **submit** — `list_projects`, `create_project`, `submit_fable`, `update_fable`, etc.

### 2. Tell your agent to record

**Option A — install as a skill** (any agent supporting the [Agent Skills](https://github.com/anthropics/skills) format):

```bash
npx skills add jtmuller5/brainfables-skill
```

Then say "record a fable" / "fable this" when you finish a piece of work, or let the agent trigger it on its own.

**Option B — paste the prompt into your `CLAUDE.md`** so it happens automatically after every substantive request (see below).

## The CLAUDE.md prompt

Drop this into your project (or global) `CLAUDE.md` to make recording a fable part of your agent's standard finishing routine:

```markdown
## Recording fables (BrainFables)

After completing each substantive request I make (a feature implemented, an issue
debugged — not typo fixes or one-liners), record a fable via the `brainfables` MCP
server: a self-contained, interactive explainer of the problem, the journey, and the fix.

- Fetch the authoring guide once with `get_fable_format`; validate every draft with
  `check_fable` and iterate until it returns ok.
- Cover: the problem as I stated it, what you investigated (wrong turns included),
  the fix and why it works. Show the decisive code as code blocks with
  `title="path/to/file.ts"` and `ins=`/`del=` line markers instead of describing it.
- Title by the problem, not the diff ("Hydration crash from Buffer in client bundle",
  not "Fix __root.tsx"). Tag by problem domain and key files/symbols so it's findable
  by a future search.
- File it under this repo's project: `list_projects`, `create_project` once if absent,
  then `submit_fable` with that projectId.
- One fable per request. A new request gets a new fable, even over the same code;
  `update_fable` only revises the fable about the same request.

Fables land on my dashboard as personal — never describe a submission as published.
```

## How it works

The recording loop, every time:

```
1. get_fable_format    fetch the authoring guide (once per session)
2. draft               write the fable: problem → investigation → fix, with code blocks
3. check_fable         validate; iterate until it returns ok
4. list_projects       find this codebase's project (create_project once if absent)
5. submit_fable        submit with that projectId — a new fable per request
```

- **One fable per request.** A new request gets a new fable, even over the same code. `update_fable` only revises the fable about the *same* request.
- **Self-contained by construction.** It covers the problem as you stated it, what the agent investigated (wrong turns included — often the most useful part), and the fix and *why* it works, with the decisive code shown as code blocks rather than described.
- **Personal until you publish.** Fables land on your dashboard visible only to you. Publishing to the community feed (CC BY-SA 4.0) is a human act on the web — agents always submit personal.

## License

MIT © Joe Muller
