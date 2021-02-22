# Use Vite Today

> out-of-box for vue-cli projects without any codebase modifications.

<p align="center">
  <img src="./logo.png" alt="logo" title="logo" width="300px" />
</p>


## ToC
- [Usage](#usage)
- [Motivation](#motivation)
- [Options](#options)
- [Underlying principle](#underlying-principle)
    - [Compatibility](#compatibility)
    - [Differences between vue-cli and vite](#differences-between-vue-cli-and-vite)
- [Milestones](#milestones)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)
    - [Some module response 404 not found](#some-module-response-404-not-found)
    - [Vite Build Support](#vite-build-support)
    - [Custom Style missing fonts](#custom-style-missing-fonts)
    - [JSX support](#jsx-support)
    - [Vue3 support](#vue3-support)
- [Benefits](#benefits)
    - [Best development-experience right now](#best-development-experience-right-now)
    - [Migration to vite smoothly](#migration-to-vite-smoothly)
    - [Lint the codebase](#lint-the-codebase)
- [Relevant Vite Plugins](#relevant-vite-plugins)
    - [vite-plugin-vue2@underfin](https://github.com/underfin/vite-plugin-vue2)
    - [vite-plugin-env-compatible](https://github.com/IndexXuan/vite-plugin-env-compatible)
    - [vite-plugin-vue-cli](https://github.com/IndexXuan/vite-plugin-vue-cli)
    - [vite-plugin-mpa](https://github.com/IndexXuan/vite-plugin-mpa)


## Usage
```sh
vue add vite
```
the plugin\'s generator will write some `main.html` for corresponding main.{js,ts}, since vite need html file for dev-server entry file


## Motivation
- We have lots of exists vue-cli(3.x / 4.x) projects.
- In Production: vue-cli based on webpack is still the best practice for bundling webapp(with code spliting, legecy-build for old browsers).
- In Development: instant server start and lightning fast HMR by vite is interesting.
- Why not we use them together ?


## [Options](https://github.com/IndexXuan/vue-cli-plugin-vite/blob/main/config/options.ts)
```js
// vue.config.js
{
  // ...
  pluginsOptions: {
    vite: {
      /**
       * will deprecated when we can auto resolve alias from vue.config.js
       * @ is setted by the plugin, you can set others used in your projects, like @components
       * Record<string, string>
       * @default {}
       */
      alias: {
        '@components': path.resolve(__dirname, './src/components'),
      },
      /**
       * Plugin[]
       * @default []
       */
      plugins: [], // other vite plugins list, will be merge into this plugin\'s underlying vite.config.ts
      /**
       * you can enable jsx support by setting { jsx: true }
       * @see https://github.com/underfin/vite-plugin-vue2#options
       * @default {}
       */
      vitePluginVue2Options: {}
    }
  },
}
```


## Underlying principle

### Compatibility
- **NO EXTRA** files, code and dependencies injected
    - injected corresponding main.html
        - SPA: `projectRoot/main.html`
        - MPA: `projectRoot/src/pages/*/main.html`(s)
    - injected one devDependency `vue-cli-plugin-vite`
    - injected one line code in `package.json#scripts#vite` and one file at `bin/vite`
- auto-resolved as much options as we can from `vue.config.js` (publicPath, alias, outputDir...)
- compatible the differences between vue-cli and vite(environment variables, special syntax...)


### Differences between vue-cli and vite

| Dimension                        |         vue-cli     |     vite           |
|----------------------------------|---------------------|--------------------|
|     Plugin                       | 1. based on webpack. <br />2. have service and generator lifecycles. <br />3. hooks based on each webpack plugin hooks | 1. based on rollup. <br />2. no generator lifecycle. <br />3. universal hooks based on rollup plugin hooks and vite self designed |
|     Environment Variables        | 1. loaded on process.env. <br />2. prefixed by `VUE_APP_`. <br />3. client-side use `process.env.VUE_APP_XXX` by webpack definePlugin | 1. not loaded on process.env. <br />2. prefixed by `VITE_`. <br />3. client-side use `import.meta.env.VITE_XXX` by vite inner define plugin |
|     Entry Files                  | 1. main.{js,ts}.    | 1. *.html          |
|     Config File                  | 1. vue.config.js    | 1. vite.config.ts. <br />2. support use --config to locate |
|     MPA Support                  | 1. native support by `options.pages`. <br />2. with history rewrite support | 1. native support by `rollupOptions.input` |
|     Special Syntax               | 1. require(by webpack) <br /> 2. require.context(by webpack) <br />2. use `~some-module/dist/index.css`(by `css-loader`) <br />3. module.hot for HMR | 1. import.meta.glob/globEager <br />2. native support by vite, use 'module/dist/index.css' directly <br />3. import.meta.hot for HMR  |


## Milestones
- ✅ Plugin
    - ✅ we can do nothing but rewrite corresponding vite-plugin, most code and tools can be reused 
- ✅ Environment Variables Compatible
    - ✅ load to process.env.XXX (all env with or without prefix will be loaded)
    - ✅ recognize `VUE_APP_` prefix (you can use other instead by config, e.g. `REACT_APP_`)
    - ✅ define as `process.env.${PREFIX}_XXX` for client-side
- ✅ Entry Files (we can do nothing)
- ⬜️ Config File (vue.config.js Options auto-resolved)
    - ✅ vite#base - resolved from `process.env.PUBLIC_URL || vue.config.js#publicPath || baseUrl`
    - ✅ vite#css - resolved from vue.config.js#`css`
        - ✅ preprocessorOptions: `css.loaderOptions`
    - ✅ vite#server- resolved from vue.config.js#`devServer`
        - ✅ host - resolved from `process.env.DEV_HOST || devServer.public`
        - ✅ port - resolved from `Number(process.env.PORT) || devServer.port`
        - ✅ https - resolved from `devServer.https`
        - ✅ open - resolved from `process.platform === 'darwin' || devServer.open`
        - ✅ proxy - resolved from `devServer.proxy`
        - ❌ before
            - maybe we cannot, webpackDevServer before is a express app instance while viteDevServer is a connect instance which is not have a router function ( e.g. we can use something like app.post('/login', xxx) )
            - or transform the connect app instance to express instance compatible ?
    - ✅ vite#build
        - ✅ outDir - resolved from vue.config.js#`outputDir`
        - ✅ cssCodeSplit - resolved from `css.extract`
        - ✅ sourcemap - resolved from `process.env.GENERATE_SOURCEMAP === 'true' || productionSourceMap || css.sourceMap`
    - ⬜️ Alias - resolved from configureWebpack or chainWebpack (WIP)
- ✅ MPA Support
    - ✅ same development experience and build result
- ⬜️ Special Synatax
    - ❌ require('xxx') or require('xxx').default, most of the case, it can be replaced by dynamicImport ( import('xxx') or import('xxx').then(module => module.default) )
    - ❌ '~some-module' syntax for Import CSS (will not support, we have workaround)
    - ✅ require.context compatibility
    - ⬜️ module.hot compatibility (WIP)

## Examples
- [simple vue-cli SPA project](https://github.com/IndexXuan/vue-cli-plugin-vite/tree/main/examples/my-mpa-ts-app)
- [simple vue-cli MPA TypeScript project](https://github.com/IndexXuan/vue-cli-plugin-vite/tree/main/examples/my-mpa-ts-app)
- [(WIP)complex chrisvfritz/vue-enterprise-boilerplate project](https://github.com/chrisvfritz/vue-enterprise-boilerplate)

you can clone/fork this repo, under examples/*

## Troubleshooting

### Vite Build Support
- Currently only support vite dev for development, you should still use yarn build(vue-cli-service build)
- But you can use `BUILD=true MODERN=true yarn vite` to invoke vite build(no legacy and use esbuild minify, not recommended, please use yarn build instead)

### some module response 404 not found
- if not compiler errors, maybe you import vue file without '.vue' ext, added it and it is required for vite and recommended for vue-cli (and required in vue-cli@5.x)

### Custom Style missing fonts
- e.g. element-plus: https://element-plus.gitee.io/#/en-US/component/custom-theme

```scss
/* theme color */
$--color-primary: teal;

/* icon font path, required */
$--font-path: '~element-plus/lib/theme-chalk/fonts'; // changed to 'path/to/node_modules/element-plus/lib/theme-chalk/fonts;'

@import "~element-plus/packages/theme-chalk/src/index"; // remove '~', css-loader support it
```

### jsx support
- see options above, vitePluginVue2Options: { jsx: true }
- you may also see that `React is not defined`, it is you use jsx without set vitePluginVue2Options: { jsx: true }

### Vue3 support
- currently only support Vue2.x, since Vue3.x you can use vite directly


## Benefits

### Best development-experience right now
- Instant server start and lightning fast HMR

### Migration to vite smoothly
- In the future, migration to vite is only the replacement of special syntax between webpack and vite

### Lint the codebase
- lint dependencies, which is correctly use [main/module/exports](https://twitter.com/patak_js/status/1363514285180268550?s=21) field in package.json
- lint codebase, which is more esmodule compatible
    - use [import('xxx')](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#dynamic_imports) not `require('xxx')`
    - use [import.meta.xxx](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta) not `module.xxx`


## Relevant Vite Plugins
- [vite-plugin-vue2@underfin](https://github.com/underfin/vite-plugin-vue2)
- [vite-plugin-env-compatible](https://github.com/IndexXuan/vite-plugin-env-compatible)
- [vite-plugin-vue-cli](https://github.com/IndexXuan/vite-plugin-vue-cli)
- [vite-plugin-mpa](https://github.com/IndexXuan/vite-plugin-mpa)

