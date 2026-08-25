<div align="center">
<img src="public/icon-128.png" alt="logo"/>
<h1> Chrome 扩展脚手架<br/>React + <b>Vite 8</b> + TypeScript</h1>

![](https://img.shields.io/badge/Vite-8-646CFF?style=flat-square&logo=vite&logoColor=white)
![](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black)
![](https://img.shields.io/badge/Typescript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![](https://badges.aleen42.com/src/vitejs.svg)
![GitHub action badge](https://github.com/webLiang/chrome-extension-boilerplate-react-vite/actions/workflows/build-zip.yml/badge.svg)
<img src="https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https://github.com/webLiang/chrome-extension-boilerplate-react-vite&count_bg=%23#222222&title_bg=%23#454545&title=😀&edge_flat=true" alt="hits"/>


> 基于 **Vite 8 + Rolldown**，生产构建与本地热重载更快。<br/>
> 收录于 [Awesome Vite](https://github.com/vitejs/awesome-vite)

**语言**

- **English**：[README.md](README.md)
- **中文**（当前）：[README.zh_CN.md](README.zh_CN.md)

</div>

## 目录

- [简介](#简介)
- [为什么用这个模板](#为什么用这个模板)
- [特性](#特性)
- [技术栈 / 工具链](#技术栈--工具链)
- [安装](#安装)
    - [步骤](#步骤)
        - [Chrome](#chrome)
        - [Firefox](#firefox)
- [接入样式库](#接入样式库)
    - [Twind](#twind)
    - [Chakra UI](#chakra-ui)
- [页面](#页面)
- [截图](#截图)
    - [NewTab](#newtab)
    - [Popup](#popup)
    - [Devtools](#devtools)
- [示例项目](#示例项目)
- [参考文档](#参考文档)

## 简介 <a name="简介"></a>

基于 React、TypeScript 与 **Vite 8** 的 Manifest V3 Chrome 扩展脚手架。

> **速度优先：** Vite 8 内置 [Rolldown](https://rolldown.rs/) 生产打包；多入口扩展构建通常在 **数百毫秒** 级完成，明显快于旧版 Rollup 时代的数秒级等待。开发态配合 HRR（热重建与刷新），迭代同样轻快。

## 为什么用这个模板 <a name="为什么用这个模板"></a>

[Jonghakseo/chrome-extension-boilerplate-react-vite](https://github.com/Jonghakseo/chrome-extension-boilerplate-react-vite) 曾是常用的 React + Vite 扩展脚手架。该仓库已于 **2026-02-14 归档（只读）**，不再迭代。后期变成 Turborepo 多包结构后，安装、`pnpm dev` 启动和构建反馈都偏慢。

本仓库是独立维护的 **精简、速度优先** 模板：

- 单包（无 Turborepo）
- Vite 8 + Rolldown — 生产构建通常约 100–300ms
- 开发态 HRR，改 popup / content / background 不必等一整套 monorepo 图

致谢原项目，本模板独立演进。

## 特性 <a name="特性"></a>

- **[Vite 8](https://vitejs.dev/) + Rolldown** — 生产构建更快（`build.rolldownOptions`），插件链路跟上最新版
- [React 18](https://reactjs.org/)
- [TypeScript](https://www.typescriptlang.org/) **5.7**
- [Vitest 4](https://vitest.dev/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [pnpm 11](https://pnpm.io/)
- [SASS](https://sass-lang.com/)
- [Prettier](https://prettier.io/)
- [ESLint](https://eslint.org/)
- [Husky](https://typicode.github.io/husky/getting-started.html#automatic-recommended)
- [Commitlint](https://commitlint.js.org/#/guides-local-setup?id=install-commitlint)
- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#summary)
- [Chrome Extension Manifest Version 3](https://developer.chrome.com/docs/extensions/mv3/intro/)
- HRR（热重建与刷新 / 重载）— HMR 辅助脚本用 Sucrase 编译，`pnpm dev` 启动更快

## 技术栈 / 工具链 <a name="技术栈--工具链"></a>

### 为什么用 Vite 8

| 收益 | 你得到什么 |
|------|------------|
| 生产构建更快 | Vite 8 底层 Rolldown；本模板多入口 `pnpm build` 在常见机器上约 **100–300ms** |
| `pnpm dev` 启动更快 | `build:hmr` 用 `@rollup/plugin-sucrase`（小脚本不再走完整 `tsc`） |
| React 工具链对齐 | `@vitejs/plugin-react` **6.x**，peer 匹配 Vite 8 |
| Content script 可用 | 处理 Vite 8 preload / `import.meta`；Firefox 相对路径 `import()` 自动改写 |

主要版本以 `package.json`、`.nvmrc` 为准：

| 工具 | 版本 |
|------|------|
| Node.js | `>= 20.19` 或 `>= 22.12`（Vite 8 要求）；推荐 **24**（`.nvmrc` = `v24.16.0`） |
| pnpm | **11.13.x**（`packageManager`: `pnpm@11.13.1`） |
| **Vite** | **8.1.x**（在 `pnpm-workspace.yaml` 中 override） |
| `@vitejs/plugin-react` | **6.x** |
| Vitest | **4.x** |
| TypeScript | **5.7.x**（`@vitejs/plugin-react` 6 类型声明所需） |

Vite 8 与 content script 相关说明：

- Content script 为经典脚本，不能使用 `import.meta`；由 `fixContentImportMeta` 改写 Vite preload 辅助代码。
- Firefox 下相对路径 `import()` 在 `renderChunk` 中由 `customDynamicImport` 改写（Vite 8 / Rolldown 已移除 `renderDynamicImport`）。
- Rolldown 下已停用 `inlineVitePreloadScript`（文件仅作参考保留）。

pnpm 11 的 `allowBuilds`、`overrides.vite`、`confirmModulesPurge` 写在 `pnpm-workspace.yaml`。

## 安装 <a name="安装"></a>

## 步骤： <a name="步骤"></a>

1. Clone 本仓库。
2. 修改 messages.json 中的 `extensionDescription` 与 `extensionName`。
3. 使用 Node **>= 20.19** / **>= 22.12**（推荐与 `.nvmrc` 一致，例如 Node 24）。通过 Corepack 启用 pnpm 11（`corepack enable`），或全局安装（`npm install -g pnpm@11`）。
4. 执行 `pnpm install`

## 然后按目标浏览器操作：

### Chrome： <a name="chrome"></a>

1. 运行：
    - 开发：`pnpm dev` 或 `npm run dev`
    - 生产：`pnpm build` 或 `npm run build`
2. 打开 `chrome://extensions`
3. 打开「开发者模式」
4. 点击「加载已解压的扩展程序」
5. 选择 `dist` 目录

### Firefox： <a name="firefox"></a>

1. 运行：
    - 开发：`pnpm dev:firefox` 或 `npm run dev:firefox`
    - 生产：`pnpm build:firefox` 或 `npm run build:firefox`
2. 打开 `about:debugging#/runtime/this-firefox`
3. 点击「临时载入附加组件…」
4. 选择 `dist` 目录下的 `manifest.json`

### <i>注意：Firefox 临时附加组件在关闭浏览器后会消失，下次启动需重新加载。</i>

## 接入样式库 <a name="接入样式库"></a>

> 重要：若不需要在 content script 中使用 CSS 文件，请从 `manifest.js` 中删除对应 css 项。

```js
content_scripts: [
  {
    // YOU NEED TO DELETE THIS
    css: ["assets/css/contentStyle<KEY>.chunk.css"]
  }
];
```

### Twind <a name="twind"></a>

> 体积小、速度快、功能完整的 Tailwind-in-JS 方案

**1. 安装依赖：**

```bash
$ pnpm install -D @twind/core @twind/preset-autoprefix @twind/preset-tailwind
```

**2. 在项目根目录创建 `twind.config.ts`**

<details>
<summary>twind.config.ts</summary>

```ts
import { defineConfig } from '@twind/core';
import presetTailwind from '@twind/preset-tailwind';
import presetAutoprefix from '@twind/preset-autoprefix';

export default defineConfig({
  presets: [presetAutoprefix(), presetTailwind()],
});
```

</details>

**3. 创建 `src/shared/style/twind.ts` 用于挂载样式**

<details>
<summary>src/shared/style/twind.ts</summary>

```ts
import { twind, cssom, observe } from '@twind/core';
import 'construct-style-sheets-polyfill';
import config from '@root/twind.config';

export function attachTwindStyle<T extends { adoptedStyleSheets: unknown }>(
  observedElement: Element,
  documentOrShadowRoot: T,
) {
  const sheet = cssom(new CSSStyleSheet());
  const tw = twind(config, sheet);
  observe(tw, observedElement);
  documentOrShadowRoot.adoptedStyleSheets = [sheet.target];
}
```

</details>

**4. 在项目中使用 Tailwind / Twind：**

<details>
<summary>src/pages/popup/index.tsx</summary>

```tsx
import { attachTwindStyle } from '@src/shared/style/twind';

...
attachTwindStyle(appContainer, document);
const root = createRoot(appContainer);
root.render(<Popup />);
```

</details>

**5. 若要在 content script 中使用 Twind，还需加入：**

<details>
<summary>src/pages/content/ui/index.tsx</summary>

```tsx
import { attachTwindStyle } from '@src/shared/style/twind';

...
attachTwindStyle(rootIntoShadow, shadowRoot);
createRoot(rootIntoShadow).render(<App />);
```
</details>

[更多示例](https://github.com/Jonghakseo/chrome-extension-boilerplate-react-vite/pull/244/)

### Chakra UI <a name="chakra-ui"></a>

**1. 安装依赖：**

```bash
$ pnpm install @chakra-ui/react @emotion/cache @emotion/react
```

**2. 在 `vite.config.ts` 中增加配置，用于
[忽略无关警告](https://github.com/TanStack/query/pull/5161#issuecomment-1506683450)**

<details>
<summary>vite.config.ts</summary>

```ts
export default defineConfig({
  build: {
    // Vite 8 uses Rolldown — prefer rolldownOptions (not rollupOptions)
    rolldownOptions: {
      // Add below code ~~~~~
      onwarn(warning, warn) {
        if (
          warning.code === "MODULE_LEVEL_DIRECTIVE" &&
          warning.message.includes(`"use client"`)
        ) {
          return;
        }
        warn(warning);
      },
      // Add above code ~~~~
    },
  },
});
```

</details>

**3. 在页面中使用 Chakra UI：**

<details>
<summary>src/pages/popup/Popup.tsx</summary>

```tsx
import { Button } from "@chakra-ui/react";

export default function Popup() {
  return <Button colorScheme="teal">Button</Button>;
}
```

</details>

---

> 若不在 content script 中使用 Chakra UI，可跳过第 4、5 步。

**4. 在 content script 中使用 Chakra UI 时，需要增加以下代码：**

<details>
<summary>src/pages/content/ui/CustomChakraProvider.tsx</summary>

```tsx
import { ReactNode, useCallback, useEffect, useState } from "react";
import {
  ColorMode,
  ColorModeContext,
  ColorModeScript,
  CSSReset,
  extendTheme,
  GlobalStyle,
  ThemeProvider
} from "@chakra-ui/react";

const theme = extendTheme();

const getCurrentTheme = () => {
  const isDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
  return isDark ? "dark" : "light";
};

type CustomChakraProviderProps = {
  shadowRootId: string;
  children: ReactNode;
};
export default function CustomChakraProvider({ children, shadowRootId }: CustomChakraProviderProps) {
  const [colorMode, setColorMode] = useState<ColorMode>(getCurrentTheme());

  useEffect(() => {
    const darkThemeMediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
    const onChangeColorSchema = (event: MediaQueryListEvent) => {
      const isDark = event.matches;
      setColorMode(isDark ? "dark" : "light");
    };

    darkThemeMediaQuery.addEventListener("change", onChangeColorSchema);

    return () => {
      darkThemeMediaQuery.removeEventListener("change", onChangeColorSchema);
    };
  }, []);

  const toggleColorMode = useCallback(() => {
    setColorMode(prev => (prev === "dark" ? "light" : "dark"));
  }, []);

  return (
    <ThemeProvider theme={theme} cssVarsRoot={`#${shadowRootId}`}>
      <ColorModeScript initialColorMode="system" />
      <ColorModeContext.Provider value={{ colorMode, setColorMode, toggleColorMode }}>
        <CSSReset />
        <GlobalStyle />
        {children}
      </ColorModeContext.Provider>
    </ThemeProvider>
  );
}
```

</details>
<details>
<summary>src/pages/content/ui/EmotionCacheProvider.tsx</summary>

```tsx
import createCache from '@emotion/cache';
import { CacheProvider, type EmotionCache } from '@emotion/react';
import { ReactNode, useEffect, useState } from 'react';

export default function EmotionCacheProvider({ children, rootId }: { rootId: string; children: ReactNode }) {
  const [emotionCache, setEmotionCache] = useState<EmotionCache | null>(null);

  useEffect(() => {
    function setEmotionStyles(shadowRoot: ShadowRoot) {
      setEmotionCache(
        createCache({
          key: rootId,
          container: shadowRoot,
        }),
      );
    }

    const root = document.getElementById(rootId);
    if (root && root.shadowRoot) {
      setEmotionStyles(root.shadowRoot);
    }
  }, []);

  return emotionCache ? <CacheProvider value={emotionCache}>{children}</CacheProvider> : null;
}
```

</details>

**5. 修改 `src/pages/content/index.tsx`：**

<details>
<summary>src/pages/content/index.tsx</summary>

```tsx
import CustomChakraProvider from '@pages/content/ui/CustomChakraProvider';
import EmotionCacheProvider from '@pages/content/ui/EmotionCacheProvider';

// ...

createRoot(rootIntoShadow).render(
  // Add Providers
  <EmotionCacheProvider rootId={root.id}>
    <CustomChakraProvider shadowRootId={rootIntoShadow.id}>
      <App />
    </CustomChakraProvider>
  </EmotionCacheProvider>,
);

```

</details>

## 页面 <a name="页面"></a>

### New Tab <a name="newtab"></a>

[覆盖 Chrome 页面](https://developer.chrome.com/docs/extensions/mv3/override/)<br/>manifest.json 中的
`chrome_url_overrides.newtab`

### Popup <a name="popup"></a>

[Browser actions](https://developer.chrome.com/docs/extensions/reference/browserAction/)<br/>manifest.json 中的
`action.default_popup`

### Devtools <a name="devtools"></a>

[Devtools](https://developer.chrome.com/docs/extensions/mv3/devtools/#creating)<br/>manifest.json 中的 `devtools_page`

### Background <a name="background"></a>

[Background](https://developer.chrome.com/docs/extensions/mv3/background_pages/)<br/>manifest.json 中的
`background.service_worker`

### ContentScript <a name="contentscript"></a>

[Content Script（contentInjected / contentUI）](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)<br/>manifest.json 中的
`content_scripts`

### Options <a name="options"></a>

[Options](https://developer.chrome.com/docs/extensions/mv3/options/)<br/>manifest.json 中的 `options_page`

### SidePanel（Chrome 114+） <a name="sidepanel"></a>

[SidePanel](https://developer.chrome.com/docs/extensions/reference/sidePanel/)<br/>manifest.json 中的
`side_panel.default_path`

## 截图 <a name="截图"></a>

### New Tab <a name="newtab"></a>

<img width="800" alt="newtab" src="https://github.com/Jonghakseo/chrome-extension-boilerplate-react-vite/assets/53500778/3e782e41-b605-4956-90e2-20cc48252820">

### Popup <a name="popup"></a>

| Black                                                                                                                                                          | White                                                                                                                                                          |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <img width="300" alt="black" src="https://github.com/Jonghakseo/chrome-extension-boilerplate-react-vite/assets/53500778/35423617-e6f5-4f65-adb3-03f068236648"> | <img width="300" alt="white" src="https://github.com/Jonghakseo/chrome-extension-boilerplate-react-vite/assets/53500778/99886d92-b6f0-4e41-b70e-5afc6d2f7eab"> |

### Devtools <a name="devtools"></a>

<img width="450" alt="devtools" src="https://github.com/Jonghakseo/chrome-extension-boilerplate-react-vite/assets/53500778/467d719d-a7db-4f77-8504-cd5ce7567793">

## 示例项目 <a name="示例项目"></a>

- https://github.com/Jonghakseo/drag-gpt-extension
- https://github.com/Jonghakseo/pr-commit-noti
- https://github.com/ariburaco/chatgpt-file-uploader-extended

## 参考文档 <a name="参考文档"></a>

- [Vite Plugin](https://vitejs.dev/guide/api-plugin.html)
- [Chrome Extension](https://developer.chrome.com/docs/extensions/mv3/)
- [Rollup](https://rollupjs.org/guide/en/)
- [Rollup-plugin-chrome-extension](https://www.extend-chrome.dev/rollup-plugin)

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=webLiang/chrome-extension-boilerplate-react-vite&type=Date)](https://star-history.com/#webLiang/chrome-extension-boilerplate-react-vite&Date)

## Contributors

贡献者列表与英文版 [README.md](README.md) 相同，请见英文文档中的 Contributors 一节。

---

## 致谢

| [Jetbrains](https://jb.gg/OpenSourceSupport)                                                                                               | [Jackson Hong](https://www.linkedin.com/in/j-acks0n/)                                            |
|--------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| <img width="100" src="https://resources.jetbrains.com/storage/products/company/brand/logos/jb_beam.png" alt="JetBrains Logo (Main) logo."> | <img width="100" src='https://avatars.githubusercontent.com/u/23139754?v=4' alt='Jackson Hong'/> |

---

[Jonghakseo](https://nookpi.tistory.com/)
