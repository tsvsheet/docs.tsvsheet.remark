---
title: tsvsheet.remark
---

**tsvsheet.remark** brings [tsvsheet](https://github.com/tsvsheet/tsvsheet) to JS markdown pipelines: a [remark](https://github.com/remarkjs/remark) plugin and a [markdown-it](https://github.com/markdown-it/markdown-it) plugin that replace every fenced ` ```sheet ` block with its **computed** table — evaluated server-side by the embedded tsvsheet engine ([tsvsheet.js](https://github.com/tsvsheet/tsvsheet.js), the Go engine as WebAssembly) and emitted as static HTML, so documents show results, not formulas, with no client-side JavaScript.

It shares its rendering contract with the Go host ([tsvsheet.goldmark](https://github.com/tsvsheet/tsvsheet.goldmark)): the same block renders identically in every markdown pipeline — a `<table class="tsvsheet">` by default, with the raw source optionally appended in a collapsible pane.

- Source: [tsvsheet/tsvsheet.remark](https://github.com/tsvsheet/tsvsheet.remark)
- Engine: [tsvsheet/tsvsheet.js](https://github.com/tsvsheet/tsvsheet.js)
- Language: [tsvsheet/tsvsheet](https://github.com/tsvsheet/tsvsheet)

## remark

`remarkTsvsheet` is an mdast transformer. remark transformers may be async, so the plugin loads the engine itself on first use — or reuses one you pass in:

```js
import { remark } from "remark";
import remarkHtml from "remark-html";
import remarkTsvsheet from "@tsvsheet/tsvsheet-remark/remark";

const out = await remark()
  .use(remarkTsvsheet)
  .use(remarkHtml, { sanitize: false })
  .process(markdown);
```

Every ` ```sheet ` code block in the tree is swapped for a raw HTML node holding the computed table; every other node — including code blocks in other languages — is left untouched.

## markdown-it

markdown-it renders synchronously, so it cannot load the engine mid-render: load once, pass it in.

```js
import MarkdownIt from "markdown-it";
import { load } from "@tsvsheet/tsvsheet";
import markdownItTsvsheet from "@tsvsheet/tsvsheet-remark/markdown-it";

const engine = await load();
const md = new MarkdownIt().use(markdownItTsvsheet, { engine });
const html = md.render(markdown);
```

The plugin overrides the `fence` renderer rule for `sheet` fences and delegates every other language to the previously-registered fence renderer.

## Options

Both hosts take the same options, matching the goldmark host's contract:

- `className` — CSS class for the rendered `<table>` (default `"tsvsheet"`)
- `showSource` — append the raw `.tsvt` source in a collapsible `<details>` pane (default `false`)
- `output` — remark only: `"html"` (default) renders a table; `"markdown"` bakes the computed sheet into a portable GFM pipe table (markdown-it, like goldmark, is an HTML-only host and ignores it)
- `engine` — a pre-loaded engine (required for markdown-it; optional for remark)

## Install

The package is consumed from the GitHub repo — it is not published to npm:

```sh
npm install github:tsvsheet/tsvsheet.remark
```
