# brainfables

An agent skill that gives you a **live dashboard of what your coding agent is doing**. You keep your project's [BrainFables](https://brainfables.com) dashboard open in a tab; your agent keeps it current over MCP as it works:

- **Last task** — what the agent is working on right now
- **Copy & paste** — commands, env var names, and URLs it needs you to copy
- **Todos** — manual actions only you can do (set a secret, grant a permission), checked off on the page
- **Architecture** — a mermaid diagram of the project, updated as the structure changes
- **Timeline** — an append-only entry for every completed task, written in the fable format: a self-contained interactive explainer of the problem, the journey, and the fix

The dashboard polls every few seconds, so updates appear while the agent works — it's how your agent communicates with you when it needs something.

Fables — standalone publishable explainers — still exist on BrainFables as a personal collection, but this skill is about the dashboard loop.

## Setup

Two things: the **brainfables MCP server** (how the agent reads and writes the dashboard) and **the instruction** to keep it current (this skill, or a `CLAUDE.md` prompt).

### 1. Add the MCP server

The server lives at `https://brainfables.com/api/mcp` (stateless Streamable-HTTP). Authenticated tools need a Bearer API key — create one on your profile page at [brainfables.com](https://brainfables.com) after signing up, then create a project for your codebase on the [dashboard](https://brainfables.com/dashboard).

```bash
claude mcp add brainfables --transport http https://brainfables.com/api/mcp \
  --header "Authorization: Bearer YOUR_API_KEY"
```

> The `get_fable_format` and `check_fable` tools are public (no key). The key is needed for everything that touches your dashboards.

### 2. Tell your agent to keep the dashboard current

**Option A — install as a skill** (any agent supporting the [Agent Skills](https://github.com/anthropics/skills) format):

```bash
npx skills add jtmuller5/brainfables-skill
```

**Option B — paste the prompt into your `CLAUDE.md`** so it happens automatically (see below).

## The CLAUDE.md prompt

Drop this into your project (or global) `CLAUDE.md`:

```markdown
## Keeping the dashboard current (BrainFables)

Keep the BrainFables dashboard for this repo current via the `brainfables` MCP server:

- **When starting work**: call `set_last_task` with a one-line description of the task.
- **Timeline entry when done**: after each substantive request (a feature implemented, an
  issue debugged — not typo fixes), record one entry: `list_projects` (then
  `create_project` once if absent) → `get_fable_format` → draft a fable-format entry →
  `check_fable` until it returns ok → `add_timeline_entry` with the projectId and the
  files touched. Title it by the problem, not the diff. One entry per request.
- **When I must act manually**: `add_todo` for action items; `add_copy_item` for values
  I must copy (commands, env var names, SQL snippets). Never put secret values in any
  dashboard section.
- **Diagram**: call `set_diagram` with updated mermaid source if the architecture changed.
```

## The tools

| Tool | What it does |
| --- | --- |
| `get_fable_format` | authoring guide for the fable format used by timeline entries (no key) |
| `check_fable` | compile-check a timeline entry draft without saving (no key) |
| `list_projects` / `create_project` | find or create the project for a codebase |
| `get_dashboard` | full dashboard state — call at session start to see open todos and where work left off |
| `set_last_task` | update what the agent is working on right now |
| `set_diagram` | replace the mermaid architecture diagram |
| `add_copy_item` / `remove_copy_item` | surface things the owner should copy |
| `add_todo` / `complete_todo` | file and resolve manual actions for the owner |
| `add_timeline_entry` / `list_timeline` | append and read the per-task history |

- **Timeline entries are full fable documents** (frontmatter + body) and go through the same compiler and rate limits as the web editor — content is data, not code.
- **Never put secrets anywhere on the dashboard** — copy items carry commands and env var *names*, not values.
- **Dashboards are private to the owner.** Publishing fables to the community feed remains a separate, human act on the web.

## License

MIT © Joe Muller
