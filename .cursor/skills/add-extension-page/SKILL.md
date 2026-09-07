---
name: add-extension-page
description: >-
  Add a Chrome extension entry in this boilerplate: extra injected JS, a new
  content_scripts item (matches / run_at), multiple js[] files, popup/options
  HTML page, or background modules. Use when the user says 加入口, 加注入,
  content script, content_scripts, 多份 js, document_start, or add a page.
---

# Add an extension entry

This repo is a **single-package Vite 8** template. All JS/HTML entries go through `vite.config.ts` `build.rolldownOptions.input`. There is **no** standalone IIFE pipeline.

1. Read [AGENTS.md](../../../AGENTS.md) section **Add an entry** (paths A–F) and **Path formula**.
2. Classify:

| Need | Path |
|------|------|
| More logic in existing inject | **A** — `import()` in `content/injected/index.ts` or `content/ui/index.ts` |
| New site / `run_at` / `all_frames` | **B** — new folder + Vite key + new `content_scripts` `{}` |
| Two files, same matches | **C** — `js: [keyA, keyB]` |
| New popup-like UI | **D** — copy `src/pages/popup/` |
| Background code | **E** — static import from `background/index.ts` |

3. Remember: Vite **key** → `dist/src/pages/<key>/index.js` → that same path in `manifest.js`. Never register `src/pages/content/<folder>/index.ts` in the manifest.
4. Content **entry** file: dynamic `import()` only. `"world": "MAIN"` is unsupported here.
5. `pnpm build`, confirm the dist file exists, Reload unpacked `dist`.
