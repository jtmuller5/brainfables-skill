---
name: brainfables
description: Keep your project's BrainFables dashboard current as you work — update the last task, surface copy/paste items and todos, keep the architecture diagram fresh, and record a timeline entry after each completed task. Trigger when starting a task, when the user must do something manually, or after completing a substantive request.
---

# BrainFables dashboard

BrainFables (https://brainfables.com) gives you and the owner a shared view of what you're working
on. The owner keeps their project page open at `https://brainfables.com/p/<project>` while you
work — it's how you communicate with them in real time. Updates appear as you make
them: the current task, things to copy, actions they need to take, the architecture
diagram, and a log of every completed task.

## Session start

1. `list_projects` — find the project for this codebase. If none exists, call
   `create_project` once (name = repo name; description and repoUrl optional).
2. `get_dashboard` — read the current state: open todos, the last task, copy items,
   diagram, and recent timeline. This tells you where work left off and what the owner
   is still waiting on.

## While working

- **`set_last_task`** — call when you start a task (and again when you finish, to
  clear or update it). Keeps the owner informed about what you're doing right now.
  Max 500 chars.

- **`add_copy_item`** — surface anything the owner should copy: a shell command,
  an env var name, a URL, a config snippet. Returns the item id so you can
  `remove_copy_item` it once it's no longer needed.
  **Never put secret values (tokens, passwords, keys) in a copy item — names only.**

- **`add_todo`** — file a manual action only the owner can perform, such as setting
  a secret in the host dashboard, granting a permission, or triggering a one-time
  migration. Returns the todo id. The owner checks it off on the dashboard; call
  `complete_todo` if you can verify it's done programmatically.

- **`set_diagram`** — replace the project architecture diagram (mermaid source).
  Update it whenever the architecture changes. Max 10k chars; empty string clears.

## After each completed task

Call `add_timeline_entry` once per completed task — it's append-only, one entry per
request. The `source` argument is a full fable document:

- **Frontmatter**: `title` = what changed (name it by the problem, not the diff),
  `summary` = one-line description, plus `tags`, `difficulty`, `model`.
- **Body**: the problem as the user stated it → what you investigated (including wrong
  turns) → the decisive fix and why it works. Show the key code as fenced blocks with
  `title="path/to/file.ts"` and `ins=`/`del=` line markers.
- **`files` arg**: list the repo-relative paths you touched.

Always validate the source first:

1. `get_fable_format` — read the authoring guide (no key needed; do this once).
2. `check_fable` — compile-check and iterate until it returns ok (no key needed).
3. `add_timeline_entry` — submit the validated source.

Returns `{ id, title, url }` — url is the project dashboard where the entry appears.

## API key

Get your key from the profile page at https://brainfables.com (API keys section). Send it as
`Authorization: Bearer bf_…`. `get_fable_format` and `check_fable` need no key.

**Never put secret values in any dashboard section** — copy items, todos, last task,
and diagram are visible to the project owner in the browser.
