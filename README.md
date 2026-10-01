# claude-project-to-obsidian

A Claude skill that copies a Claude project into your Obsidian vault as plain markdown notes: the project's docs, uploaded files (with a description note for every image, so logos and screenshots become searchable), what Claude remembers in the project, and related Claude docs.

I built it to give my local AI assistant everything I had worked out in Claude. It works just as well as an Obsidian backup of your projects.

## What you get

```
<your vault>/
  Projects/<project name>/
    _About.md                 index with links to everything
    <docs>.md                 the project's docs, copied word for word
    Files/                    uploaded files + a .md note per file
    Memory/project-memory.md  what Claude remembers in this project
  Artifacts/<title>.md        related Claude docs, as markdown
```

Every note gets frontmatter (`source`, `exported`, `tags: [claude-export, ...]`), so you can filter exports in Obsidian.

## Requirements

- Claude's **desktop app** with a chat **attached to a project** and **linked to your computer**. The skill uses the project, artifact and computer tools that are available there; it won't work in a plain chat without them.
- An Obsidian vault folder on that computer.

## Install

1. Download this repo and zip the `claude-project-to-obsidian` folder (the one with `SKILL.md` inside).
2. Upload that zip as a custom skill in Claude's settings.

## Use

Open a chat inside the project you want to export and say:

> export this project to Obsidian

The first time it asks for your vault folder and for access to it. Run it again any time; it overwrites the previous copy.

## Good to know

- **Your vault is plain text.** The skill never writes passwords, API keys, tokens or card/bank numbers, and tells you what it skipped. Still, think before syncing that vault anywhere public.
- **Chats aren't exported.** Only docs, files, memory and artifacts are reachable. Save a chat's outcome as a project doc first if you want it in the vault.
- **Copies, not a sync.** Edit in Claude and export again; edits made in Obsidian get overwritten on the next export.

## License

MIT
