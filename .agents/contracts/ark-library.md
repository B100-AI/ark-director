# Ark library contract (all audio and video skills)

The Ark fix library is the workspace memory of what Seedance, Seed Audio and Seedream get wrong and how each problem was fixed. Every skill that writes a spoken line, builds a prompt, submits a render, reviews a take or fixes one follows this contract, so the library and its Library page stay in sync with real work.

## One live library, everything else is a copy

- Live library (the only place that is written): laptop `C:\Users\Emil V. Capino\DATA\EVC-AI\ark-library\` = device shell `$HOME/mnt/EVC-AI/ark-library` (needs the EVC-AI folder connected).
  - `ark-fix-database.json` is the master: fixes (F-entries), methods M0–M7, platform constraints, owner rules, QA rules and the lexicon.
  - `pronunciation-lexicon.json` is generated from the master. Never edit it by hand.
  - `fix-media/Fxxx/` holds the before/after clips for the Library page.
- Tool: `arklib.py`. In the device shell use the live copy `python3 $HOME/mnt/EVC-AI/ark-library/arklib.py`. Anywhere else use `python3 ${CLAUDE_PLUGIN_ROOT}/demo-kit/arklib.py` (same program; it finds the live library, or falls back to the read-only snapshot `${CLAUDE_PLUGIN_ROOT}/library/`).
- Read-only copies (never edit; refreshed by `arklib.py mirror`): the plugin `library/` snapshot, kit and campaign copies listed in `mirrors.json`, and the Project doc `claude/ark-fix-database.json`.
- Library page (claude.ai artifact, built from the live library): https://claude.ai/artifact/ApTkJFpRLr1o2D6bSFhoH5 — its sync bar shows the database version it was built from.

## The three hooks

### 1. Before writing or rendering (read)
- `arklib.py constraints` — platform limits, owner rules and QA rules. Apply them to the prompt and the task settings.
- Every spoken line, caption-read line or TTS text: `arklib.py lookup "<line>"` or `arklib.py lookup-file content-plan.json|lines.json`. An `avoid` word (exit 1) must be swapped or respelled before any paid render; a `watch` word is listened for after the render. Full procedure: `dialogue-pronunciation-guard`.
- Show normal spelling in captions and demos: `arklib.py display-map`.
- If the library cannot be reached at all, say so and use the snapshot; never skip the lookup silently.

### 2. When something fails (read)
- `arklib.py match "<symptom words>"` (for example `match "extra consonant before ang"`, `match "elevator buttons by door"`) before choosing a fix. Use the method that worked last time first. Ladder and splice steps: `dialogue-fix-ladder`.

### 3. After the owner approves a fix (write, laptop only)
1. `arklib.py add-fix fix.json --campaign <campaign-slug>` — one entry per fix, failed attempts included (fields: project, clip, kind, problem, word, original, attempts[{n, method, text, result}], final_text, method, lesson, files). Ids are assigned automatically.
2. New or changed words: `arklib.py add-word word.json`.
3. Before/after clips: `arklib.py media-add Fxxx <before.mp4> <after.mp4> [--before-label T] [--after-label T] [--windows a-b]`, or set `no_media` with a reason in the entry.
4. End of every production run: `arklib.py close-run <campaign folder>` — syncs the lexicon, refreshes all mirrors and the plugin snapshot, builds the Library page into `page/build/`, and fails if the campaign has fix takes (`*fix*`, `*patch*`, `voice-fix/`, `fixes/*.json`) with no F-entry.
5. Claude publishes `page/build/` to the Library page artifact (publish_files, patch mode), then runs `arklib.py page-published <version> <url>`.
6. Refresh the Project doc mirror: `arklib.py export-compact <file>` and write it to `claude/ark-fix-database.json`.
7. Report the new F-ids, the fix count and the `check` result to the owner.

## Never
- Edit `ark-fix-database.json`, `pronunciation-lexicon.json` or any copy by hand, or log a fix the owner has not approved.
- Treat a copy as the master, or write fixes into a campaign or kit copy.
- Finish a run with `arklib.py check` reporting a problem without telling the owner.
