---
title: "Nuxt3 + TypeScript環境のESLintの設定"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["nuxt3", "eslint", "typescript"]
published: true
---

# Nuxt3 + TypeScript環境のESLintの設定

## 依存パッケージのインストール

```bash
npm install -D @nuxtjs/eslint-config-typescript @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-plugin-vue
```

## .eslintrc.jsの記述

次の2つのドキュメントを参照します。
- [nuxt/eslint-config README](https://github.com/nuxt/eslint-config#typescript)
- [eslint-plugin-vue User Guide](https://eslint.vuejs.org/user-guide/#what-is-the-use-the-latest-vue-eslint-parser-error)

これらのドキュメントに記載されている設定をまとめると次のようになります。

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    "plugin:vue/vue3-recommended",
    "eslint:recommended",
    "@nuxtjs/eslint-config-typescript",
  ],
  parser: "vue-eslint-parser",
  parserOptions: {
    parser: "@typescript-eslint/parser",
    sourceType: "module",
  },
  plugins: ["vue", "@typescript-eslint"],
};
```