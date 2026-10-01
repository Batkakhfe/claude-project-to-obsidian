---
name: claude-project-to-obsidian
description: Export the current Claude project (docs, uploaded files like logos and PDFs, project memory, related Claude docs and artifacts) into an Obsidian vault on the user's computer as searchable markdown notes. Use when asked to export, back up or sync a Claude project to Obsidian.
---

# Export a Claude project into an Obsidian vault

Copies the CURRENT Claude project into `<vault>/Projects/<project name>/` as plain markdown notes, so the knowledge is browsable in Obsidian and searchable by any local tool that indexes the vault. Use it when the user says "export this project to Obsidian", "sync this project to my vault", "back up this project" or similar.

## Before you start
- This needs a Claude session that is **attached to a project** (Projects tool) and **linked to the user's computer** (the remote-devices tools in Claude's desktop app). If either is missing, say which one and stop.
- **Vault folder:** use the path the user gave. If none, ask once for the Obsidian vault folder (the folder that contains `.obsidian/`). Request access to exactly that folder (`device_request_folder_access`), nothing else.
- **Plain text on disk:** an Obsidian vault is not encrypted. Never write credentials into it: API keys, tokens, passwords, card or bank numbers, recovery codes. Leave such lines out and tell the user what you skipped.
- **Copy, don't rewrite:** no summarising, no rewording, nothing dropped. Never re-type a long file from tool output; copy files, or decode exports with a script.

## Steps
1. `project_info` -> name, description, instructions, docs, uploaded files. Target folder = `Projects/<name>/` with Windows-forbidden characters (`<>:"/\|?*`) removed.
2. **Docs.** `project_read` each doc.
   - Returned inline: write it to `<target>/<doc path>` exactly as returned.
   - Returned as a local file: copy it byte for byte.
   - Save non-.md docs as `.md`.
   - Put this frontmatter on top (if the doc already has YAML frontmatter, keep its keys and add these):
     ```
     ---
     source: Claude project "<name>" · <doc path>
     exported: <today, YYYY-MM-DD>
     tags: [claude-export, <project-slug>]
     ---
     ```
3. **Uploaded files.** `project_read` downloads each original.
   - Keep the original at `<target>/Files/<original name>`.
   - PDF, DOCX, PPTX, XLSX: also write `<target>/Files/<name>.md` with the extracted text (frontmatter + `original: "[[<original name>]]"`). Search tools usually index only `.md`.
   - **Images (logos, screenshots, photos): one note per image is mandatory**, even when another doc already describes the set. Look at the image yourself, then write `<target>/Files/<name>.md`: frontmatter, `![[<original name>]]`, then 2-5 plain sentences on what it shows. For a logo: the wordmark text, the symbol, colours (hex if clear), background, size, and which variant it is (light, dark, icon only).
   - **Same image in several formats** (for example `logo.png` + `logo.svg`): write ONE note `logo.md` that embeds the raster version and links all versions. Check they really match (render the SVG to the PNG's size and compare) before saying so.
4. **Project memory.** `project_memory_list`, then `project_memory_read` each file. Write one note `<target>/Memory/project-memory.md`: the facts as bullet lines grouped by file, without bookkeeping tags or YAML. Add nothing that isn't there.
5. **Artifacts.** `Artifact` list with scope all. Pick the ones whose title clearly belongs to this project; if unsure, ask the user once with the candidate titles.
   - A Claude doc: read it with the Claude Docs connector to get its tab ids, `export` each tab as markdown, base64-decode with a script, and save `Artifacts/<title>.md` at the vault root with frontmatter (`source:` = the artifact URL).
   - Any other artifact: `Artifact` read, save the HTML as `Artifacts/<title>.html` and its visible text as `Artifacts/<title>.md`.
   - `Artifacts/` is shared by all projects; the same title means overwrite.
6. **Index note** `<target>/_About.md`: project name, description, export date, the line "Copies - edit in Claude, not here; the next export overwrites them.", and a `[[link]]` to every exported note. Every link must point at a file that exists.
7. **Deliver.** Build everything in the cloud workspace first. Then `device_bash` `mkdir -p` the target folders under the mounted vault, and `device_commit_files` each file to its final path (max 50 per call). Overwriting an earlier export is expected.
8. **Check.** On the computer: hash every file under `Projects/` and `Artifacts/` and report byte-identical duplicates; confirm every `[[link]]` in `_About.md` resolves.
9. **Report** in 2-3 lines: counts of docs, files, image notes, memory facts and artifacts, plus anything skipped and why.

## Re-running
Safe. The same paths get overwritten, so the vault copy matches the project again. Files removed from the project stay in the vault until the user deletes them.

## What this can't export
The chat conversations themselves - only docs, uploaded files, project memory and artifacts are reachable. To carry a conversation over, write its outcome into a project doc first. Other projects: run this skill inside each of them.
