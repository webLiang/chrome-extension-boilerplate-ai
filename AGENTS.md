# Chrome Extension Boilerplate (AI)

Manifest V3 Chrome extension template for AI-assisted development. Stack: React 18, TypeScript 5.7, Vite 8 + Rolldown, pnpm 11. Single package (no Turborepo).

## Commands

- `pnpm install` — install (Node >= 20.19 / >= 22.12; recommended: match `.nvmrc`)
- `pnpm dev` — Chrome HRR (hot rebuild & refresh); load unpacked `dist`
- `pnpm build` — production build
- `pnpm dev:firefox` / `pnpm build:firefox` — Firefox temporary add-on

## Layout

Extension pages live under `src/pages/`: `background`, `popup`, `content` (`contentInjected` / `contentUI`), `options`, `newtab`, `devtools`, `sidepanel`. Manifest source is `manifest.js` (MV3).
