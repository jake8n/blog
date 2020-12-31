---
layout: blog.ejs
tags: blog
title: 4 ways to improve type checking in Vue components
---

TypeScript is increasingly becoming a go-to when starting new projects. It has proven to catch pesky bugs arising from loosely typed code and ease the refactoring of larger codebases. For that we are eternally grateful 🙌.

However, when it comes to using TypeScript with Vue there are some pain points. The main culprit being Single File Components, which do not come under full scrutiny by the TypeScript compiler. Type errors within a `<template>` tag are not detected and can slip through into a production environment. Component property violations, for example, are only detected at run time.

Here I will outline 4 methods to ensure 100% of your Vue component is type checked.

## 1. [Vue JSX Factory](https://www.npmjs.com/package/vue-jsx-factory)

The first two recommendations ditch Single File Components in favour of JSX.

Ultimately `.vue` files are a proprietary syntax known only by Vue and other community maintained projects. JSX is not a standard either, but in the world of TypeScript is can be considered as such. We can take advantage of that!

Vue JSX Factory is the only method where by we can directly use `tsc` (or esbuild) to compile our code. Babel is an optional enhancement - not a requirement - and so our project setup is very lightweight. In fact, the only necessary package is `vue-jsx-factory`. After updating `tsconfig.json` to the following:

```json
{
  "compilerOptions": {
    "jsx": "react",
    "jsxFactory": "j"
  }
}
```

components can be defined using JSX. Importantly, the `j` function must be within scope wherever we define components:

```tsx
// Header.tsx
import { j } from "vue-jsx-factory";
import { defineComponent } from "@vue/composition-api";

export const Header = defineComponent({
  name: "Header",
  props: {
    title: {
      type: String,
      required: true,
    },
  },
  render() {
    return (
      <header>
        <h1>{this.title}</h1>
      </header>
    );
  },
});
```

You do lose out on some directives (e.g. `v-model`), although these could enabled at a later date. Going down this route removes a lot of fluffy dependencies, such as Vue Loader or Vue Jest.

## 2. [Babel Preset JSX](https://github.com/vuejs/jsx)

JSX can also be written with the aid of a Babel preset. It is officially supported by the Vue core team and has more features than Vue JSX Factory. Presuming Babel is already integrated in your project, just install the packages and update the Babel configuration:

```
npm install -D @vue/babel-preset-jsx @vue/babel-helper-vue-jsx-merge-props
```

```js
// babel.config.js

module.exports = {
  presets: ["@vue/babel-preset-jsx"],
};
```

Thanks to some Babel magic, the render function is automatically imported in to scope, so no need for a manual import in component files.

## 3. [Vetur Terminal Interface (VTI)](https://vuejs.github.io/vetur/guide/vti.html) ⚠

An extension to the famous Vetur VS Code plugin and also available as a command line tool, VTI enables type checking on Single File Components. Unfortunately, it is still marked as unstable, although I am excited to see where it is headed! At the time of writing this article it can run via a single command:

```bash
npx vti # at project root
```

## 4. [TypeCheck for Vue](https://github.com/znck/vue-developer-experience/tree/master/packages/typecheck)

Last but not least, Vue Developer Experience provides us with TypeCheck, another method for checking Single File Components. Currently it runs as a command alongside your build and, like VTI, is still early days; the first public release being 28th December 2020!

## Conclusion

Using Vue together with TypeScript can at times feel premature. Relying on community maintained dependencies to achieve type safety is risky and in an ideal world avoidable. While there is hope with Vue 3 and Vite, it will take some time for the ecosystem to catch up. In the mean time, those using Vue 2 can find an extra layer of assurance with these additional tools.

If you would like to get those benefits early, without fully migrating to Vue 3, the Composition API is already available in Vue 2. `defineComponent` has massively better type inference than `Vue.extend` or the Class Component Syntax ([soon to be deprecated](https://github.com/vuejs/rfcs/pull/17#issuecomment-494242121)).

Thanks for reading!
