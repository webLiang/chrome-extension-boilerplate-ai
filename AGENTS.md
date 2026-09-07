# Chrome Extension Boilerplate (AI)

Manifest V3 Chrome extension **boilerplate** (not a framework). Single package: React 18, TypeScript 5.7, Vite 8 + Rolldown, pnpm 11. No Turborepo.

Human docs: [README.md](README.md) · [README.zh_CN.md](README.zh_CN.md). Cursor also loads `.cursorrules` and `.cursor/rules/*.mdc`. Adding a page / extra injected JS: [Add an entry](#add-an-entry).

## Hard constraints

- **Single package.** Do not add a monorepo / Turborepo / extra `packages/*`.
- **Vite 8 uses `build.rolldownOptions`**, not `build.rollupOptions`.
- **Do not enable** `inlineVitePreloadScript`.
- **Content script entries** (`src/pages/content/**/index.ts`): **dynamic `import()` only** — no top-level `import x from`. Chrome injects them as classic scripts.
- **Content React CSS:** `import styles from './file.css?inline'` (or SCSS `?inline`) into Shadow DOM. Never `import './file.css'`.
- **i18n:** `public/_locales/**/messages.json`. Comments and `console.*`: English.
- Commits: Conventional Commits. Husky runs commitlint.

## Commands

Node **>= 20.19** or **>= 22.12** (recommended: `.nvmrc` = v24). pnpm **11.13.x**.

| Command | What it does |
|---------|----------------|
| `pnpm install` | Install |
| `pnpm dev` | Chrome HRR → `dist` |
| `pnpm build` | `tsc --noEmit` + production build |
| `pnpm dev:firefox` / `pnpm build:firefox` | `__FIREFOX__=true` |
| `pnpm test` / `pnpm lint` / `pnpm prettier` | Vitest / ESLint / format |

Load unpacked from **`dist`**. After SW or content changes: **Reload** the extension, then hard-refresh the tab.

## Path formula (read this before adding files)

Vite **input key** decides the file on disk. Source folder name does **not**.

```text
vite.config.ts  rolldownOptions.input.<key>
        →  dist/src/pages/<key>/index.js     (or index.html for HTML entries)
        →  manifest.js  must use  src/pages/<key>/index.js
```

Built-in example — **do not** point manifest at `src/pages/content/injected/index.js` (that path is source-only):

| Source | Vite key | dist / manifest path |
|--------|----------|----------------------|
| `src/pages/content/injected/index.ts` | `contentInjected` | `src/pages/contentInjected/index.js` |
| `src/pages/content/ui/index.ts` | `contentUI` | `src/pages/contentUI/index.js` |
| `src/pages/content/style.scss` | `contentStyle` | `assets/css/contentStyle<KEY>.chunk.css` |
| `src/pages/background/index.ts` | `background` | `src/pages/background/index.js` |
| `src/pages/popup/index.html` | `popup` | `src/pages/popup/index.html` |

`entryFileNames` is `src/pages/[name]/index.js` (`[name]` = the key).

## Layout

```text
.
├── AGENTS.md / .cursorrules / .cursor/rules/
├── manifest.js              # MV3 source → dist/manifest.json
├── vite.config.ts           # aliases + rolldownOptions.input
├── public/_locales/en/messages.json
├── src/pages/
│   ├── background/          # service worker, type: module
│   ├── popup/ options/ newtab/ sidepanel/
│   ├── devtools/ + panel/
│   └── content/
│       ├── injected/        # classic CS (page DOM / theme demo)
│       ├── ui/              # classic CS (React + open Shadow DOM)
│       └── style.scss       # CSS injected into the HOST page
├── src/shared/              # storages, hooks, hoc
└── dist/                    # unpacked root
```

Aliases: `@root` `@src` `@pages` `@assets` (same in `tsconfig.json`).

## Add an entry

Pick **one** path. Do not add a new Vite key if you only need more code in an existing injection.

```text
Need more JS on the page?
├─ Same matches / run_at / world as an existing content_scripts item
│   ├─ Just more modules  →  A. extra import() in that entry
│   └─ Separate file, same injection  →  C. second js[] + new Vite key
└─ Different matches, run_at, all_frames, or world
    └─ B. new content_scripts {} + new Vite key + new src/pages/content/<name>/
```

HTML surfaces (popup / options / …) → **D**. Background → **E**.

### A. Extra modules in an existing injection (no Vite / manifest)

Same as shipping a second feature inside `contentInjected`. Edit the **entry only**:

```ts
// src/pages/content/injected/index.ts
import('@pages/content/injected/toggleTheme');
import('@pages/content/injected/myFeature'); // new file, static imports OK inside myFeature.ts
```

Do **not** add `myFeature.ts` to `rolldownOptions.input`. It is pulled in through the dynamic import graph and lands in `assets/js/*` (already in `web_accessible_resources`).

Same pattern for content UI: add `import('@pages/content/ui/anotherRoot')` in `content/ui/index.ts`.

### B. New injected script (new `content_scripts` item)

Use this when matches / `run_at` / `all_frames` differ (e.g. only `https://example.com/*`, or `document_start`).

**1. Source** — `src/pages/content/<name>/index.ts` (classic-script entry):

```ts
/**
 * Content script entry: dynamic import only (no top-level static import).
 */
import('@pages/content/<name>/main');
```

`main.ts` may use normal `import`. For HRR in a view-like module: `import refreshOnUpdate from 'virtual:reload-on-update-in-view'; refreshOnUpdate('pages/content/<name>');`

**2. Vite** — `vite.config.ts` `build.rolldownOptions.input` (key is **camelCase**, becomes the dist folder):

```ts
contentExample: resolve(pagesDir, 'content', 'example', 'index.ts'),
```

**3. Manifest** — new object in `manifest.js` `content_scripts`. Path uses the **key**, not the source folder:

```js
{
  matches: ['https://example.com/*'],
  js: ['src/pages/contentExample/index.js'],
  run_at: 'document_idle', // or document_start / document_end
  // all_frames: true,
}
```

Tighten `matches`. Do not keep `<all_urls>` unless you need it. Default `world` is `ISOLATED` (correct for this template).

**4. Build and check**

```bash
pnpm build
# must exist:
#   dist/src/pages/contentExample/index.js
# first line must NOT be a static:  import {  / import x from
```

Reload unpacked extension, then refresh the matched page.

### C. Several JS files in **one** `content_scripts` item

Same `matches` / `run_at` / `world`, Chrome runs `js[]` **in order**. Each file still needs its **own Vite key** (unless you used path A).

```js
{
  matches: ['https://example.com/*'],
  js: [
    'src/pages/contentInjected/index.js',
    'src/pages/contentExample/index.js',
  ],
}
```

Do not list source paths like `src/pages/content/example/index.ts`.

### D. New HTML page (popup / options / extra UI)

Copy `src/pages/popup/` (`index.html` + `index.tsx` + component). HTML must keep:

```html
<div id="app-container"></div>
<script type="module" src="./index.tsx"></script>
```

Add `myPage: resolve(pagesDir, 'myPage', 'index.html')` to `input`. Point `manifest.js` at `src/pages/myPage/index.html` (`action.default_popup`, `options_page`, `side_panel.default_path`, `chrome_url_overrides`, or `devtools_page`). Add `permissions` if needed (`sidePanel`, …). Call `refreshOnUpdate('pages/myPage')` in `index.tsx`.

### E. Background

One SW: `src/pages/background/index.ts` is already `type: 'module'` — **static imports are OK**. Do not add a second `background` key. Put extra logic in `@src/shared` or files imported from `background/index.ts`. SW has no `window` / DOM.

### F. Extra host-page CSS

Either append to `src/pages/content/style.scss` (already wired as `contentStyle` + `<KEY>`), or add another SCSS input key and another `css: ['assets/css/<baseName><KEY>.chunk.css']` on the right `content_scripts` item. `make-manifest` replaces every `<KEY>` in `content_scripts[].css`. If you drop content CSS entirely, delete the `css` array **and** `reloadOnUpdate('pages/content/style.scss')` in background.

## Content scripts (classic)

```ts
// ✅ entry
import('@pages/content/ui/root');

// ❌ entry — Chrome: Cannot use import statement outside a module
import Root from '@pages/content/ui/root';
```

Content UI (`ui/root.tsx`): open Shadow DOM + `injected.css?inline`. Host CSS: `content/style.scss` → `assets/css/contentStyle<KEY>.chunk.css`.

`fixContentImportMeta` rewrites Vite 8 `import.meta` in content chunks. Firefox `__FIREFOX__=true` rewrites relative `import()` via `customDynamicImport`. Keep WAR `assets/js/*.js` and `assets/css/*.css`.

**MAIN world:** this template **code-splits** content and loads chunks with `import()`. That is for **ISOLATED** world. Do not set `"world": "MAIN"` on these entries. To touch page `window`, stay isolated and use a `postMessage` / `window` event bridge, or inject a small inline `<script>` from isolated CS.

## Pages vs manifest vs Vite (stock template)

| Surface | Vite key | Manifest |
|---------|----------|----------|
| Background | `background` | `background.service_worker` |
| Popup | `popup` | `action.default_popup` |
| Options | `options` | `options_page` |
| New Tab | `newtab` | `chrome_url_overrides.newtab` |
| Side panel | `sidepanel` | `side_panel.default_path` |
| DevTools | `devtools` + `panel` | `devtools_page` |
| Content injected | `contentInjected` | `content_scripts[].js` |
| Content UI | `contentUI` | second `content_scripts` item |
| Content CSS | `contentStyle` | `content_scripts[].css` + `<KEY>` |

Remove a demo: delete the input key, the manifest field, and the folder.

## Manifest and i18n

Edit **`manifest.js`**, never `dist/manifest.json`. Version = `package.json` `version`. Names: `public/_locales/<locale>/messages.json`.

## Shared code and HRR

Reusable logic: `src/shared/` (see `storages`). HRR virtual modules: `virtual:reload-on-update-in-background-script`, `virtual:reload-on-update-in-view`. `pnpm dev` = `build:hmr` + `wss` + `build:watch` — not Vite’s default HTML-app HMR.

## Tests

Vitest + Testing Library: `*.test.ts` / `*.test.tsx`, jsdom, `test-utils/vitest.setup.js`.

## Common mistakes

| Symptom | Cause |
|---------|--------|
| Manifest `js` 404 / script missing | Used source path `content/foo/index.ts` instead of `src/pages/<viteKey>/index.js` |
| `Cannot use import statement outside a module` | Static `import` in a content **entry** |
| `Failed to fetch dynamically imported module` | Missing WAR `assets/js/*.js`, or chunk path wrong |
| Host page requests `/assets/*.css` | Bare `import './x.css'` instead of `?inline` |
| New file never appears in `dist` | Forgot the Vite `input` key |
| Injected on the wrong site | Reused `<all_urls>` instead of a new `content_scripts` item (path B) |
| MAIN world broken / `import.meta` | `"world": "MAIN"` on a split entry — stay ISOLATED |
| SW still old after `pnpm dev` | Need chrome://extensions **Reload**, then refresh the tab |
