---
name: motion-studio
description: Entry point for making a video with the claude-motion toolkit from any project — sets up a workspace (Remotion, the sfx sound engine, review and capture scripts) if there isn't one, then runs the whole flow from storyboard to a mastered MP4. Use when the user asks for a motion graphic, animated explainer, launch video, social clip, or simulation video and the current directory is not already a claude-motion workspace, or when they ask which skill to start with.
---

# Motion studio

The other skills in this toolkit (`motion-design`, `review-loop`, `sound-design`, `web-sims`) assume you are inside a claude-motion workspace: `src/lib.ts`, `sfx/`, `scripts/sheet.mjs`, `sims/` and the npm scripts in `package.json`. This skill gets you there and then hands off to them.

## 1. Find or create the workspace

A directory is a workspace when it contains `sfx/__init__.py`, `scripts/sheet.mjs` and `src/lib.ts`. Check the current directory first; if it is one, skip to step 2.

Otherwise ask the user where the video project should live (default: `./motion` inside the current project) and create it from the toolkit:

```bash
SRC="${CLAUDE_PLUGIN_ROOT}"
DEST=./motion
if [ -f "$SRC/sfx/__init__.py" ]; then
  mkdir -p "$DEST"
  tar -C "$SRC" --exclude=.git --exclude=node_modules --exclude=out --exclude=media \
    --exclude=.claude --exclude=.claude-plugin -cf - . | tar -C "$DEST" -xf -
else
  git clone --depth 1 https://github.com/FuraPauta/claude-motion "$DEST"
fi
```

When this skill runs from an installed plugin, `${CLAUDE_PLUGIN_ROOT}` is the toolkit itself and the copy needs no network. The skills stay in the plugin, so the copy leaves out `.claude/` to avoid loading them twice.

Then, inside the workspace:

1. Check `node --version` (20+), `python3 --version` and `ffmpeg -version`. Install what is missing or tell the user exactly what to install.
2. `npm install`.
3. Smoke test: `python3 -m sfx --list` and `npm run typecheck`. For sims also `npx playwright install chromium` once (or set `$CHROMIUM`).
4. Optional, if the user wants to see what the toolkit produces: `npm run build` renders the example to `out/effort.mp4`.

Run every later command from the workspace root; all paths in the other skills are relative to it.

## 2. Pick the pipeline

| The idea is | Pipeline | Skill to follow |
|---|---|---|
| Type, UI, numbers, timed reveals, an explainer | Remotion (`src/`) | `motion-design` |
| Hundreds of agents or particles, a 3D object, a fly-through | Web sim (`sims/`) | `web-sims` |

## 3. Run the flow

`AGENTS.md` in the workspace is the full guide. In short:

1. Agree on the message, a storyboard and the one hero moment. Ask for format (1:1, 16:9, 9:16), length, brand colors and fonts.
2. Write every beat into a timeline JSON before any component code.
3. Build the scenes (`motion-design`) or the sim page (`web-sims`). Palette and fonts in one object, the way `C` and `F` are in `src/lib.ts`.
4. Run `review-loop` until a full pass over the contact sheets is clean. Never call a video done without it.
5. Write a cue sheet and render the sound (`sound-design`).
6. Render, mux, and check loudness on the final file (`npm run wave -- out/<name>.mp4 --timeline <json>`).

## Report

Give the user the path to the final MP4, the contact sheet you last reviewed, the measured loudness, and anything you chose not to fix.
