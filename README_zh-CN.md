# vite ⚡

[![npm][npm-img]][npm-url]
[![node][node-img]][node-url]
[![unix CI status][unix-ci-img]][unix-ci-url]
[![windows CI status][windows-ci-img]][windows-ci-url]

Vite 是一个有态度的 web 开发构建工具，在本地开发时使用原生的 ES Module 特性导入你的代码。在生产环境使用 [Rollup](https://rollupjs.org/) 打包代码。

- 更快的冷启动速度
- 即时的热替换功能
- 真正的按需编译
- 更多详细信息请查看[How and Why](#how-and-why)

## 状态

目前在beta阶段, 预计很快发布1.0版本。

## 快速开始

> 对于 Vue 用户来说: Vite 目前仅支持 Vue 3.x. 这意味着你不能使用尚未与 Vue 3 兼容的库

```bash
$ npm init vite-app <project-name>
$ cd <project-name>
$ npm install
$ npm run dev
```

如果你使用yarn:

```bash
$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

> 虽然 Vite 主要是为了与 Vue 3 一起工作而设计的, 但它同时也能很好的支持其他框架。例如, 你可以尝试 `npm init vite-app --template react` 或者 `--template preact`.

### 使用 master 分支

如果你迫不及待的想要测试最新的特性，克隆 `vite` 到本地并且执行以下命令:

```
yarn
yarn build
yarn link
```

然后进入基于 vite 创建的项目并且执行 `yarn link vite`。 现在可以重启你的服务 (`yarn dev`) 去体验最新的特性！

## Browser Support

Vite 在开发环境下依赖原生[ES module imports](https://caniuse.com/#feat=es6-module)。在生产环境构建时依赖动态加载给你来做代码分割 (polyfill实现为[polyfilled](https://github.com/GoogleChromeLabs/dynamic-import-polyfill))

Vite 假设你的代码运行在现代浏览器上，默认将会使用 `es2019` 规范来转换你的代码。(这使得可选链功能在代码压缩后能被使用)。同时你能够手动的在配置选项中指定构建目标版本。最低的版本规范是 `es2015`。

## 功能

- [Bare Module Resolving](#bare-module-resolving)
- [Hot Module Replacement](#hot-module-replacement)
- [TypeScript](#typescript)
- [CSS / JSON Importing](#css--json-importing)
- [Asset URL Handling](#asset-url-handling)
- [PostCSS](#postcss)
- [CSS Modules](#css-modules)
- [CSS Pre-processors](#css-pre-processors)
- [JSX](#jsx)
- [Web Assembly](#web-assembly)
- [Inline Web Workers](#web-workers)
- [Custom Blocks](#custom-blocks)
- [Config File](#config-file)
- [HTTPS/2](#https2)
- [Dev Server Proxy](#dev-server-proxy)
- [Production Build](#production-build)
- [Modes and Environment Variables](#modes-and-environment-variables)

Vite 尽可能的尝试复用 [vue-cli](http://cli.vuejs.org/) 的默认配置。如果你之前使用过 `vue-cli` 或者其他基于 `Webpack` 的模版项目，你应该会非常熟悉这些。这意味着不要期望在这里和那里会有什么不同。

### Bare Module Resolving

原生的 ES imports 规范不支持裸模块的导入，例如

```js
import { createApp } from 'vue'
```

这种写法默认将会抛出错误。Vite 会检测到当前服务中的所有 `.js` 文件，并且重写他们的路径例如 `/@modules/vue`。 在这种路径下，Vite 将会从你安装的依赖中正确的解析执行模块。

对于 `vue` 这个依赖则有着特殊的处理。如果你没有在项目的本地依赖中安装，Vite 将会回退到它自身的依赖版本。这意味着如果你全局安装了 Vite, 它将更快的找到 Vue 实例而不需要在本地安装依赖。

### Hot Module Replacement

- `vue` `react` 以及 `preact` 模版已经在 `create-vite-app` 集成了热替换功能可以开箱即用

- 为了手动的控制热替换功能, 可以使用该 API `import.meta.hot`.

  如果模块想要接收到自身替换回调, 可以使用 `import.meta.hot.accept`:

  ```js
  export const count = 1

  // 这个条件判断使得热替换相关的代码将会在生产环境被丢弃
  if (import.meta.hot) {
    import.meta.hot.accept((newModule) => {
      console.log('updated: count is now ', newModule.count)
    })
  }
  ```

  一个模块同样能够接收到来自其他模块的更新通知而不需要重新加载。可以使用 `import.meta.hot.acceptDeps`：

  ```js
  import { foo } from './foo.js'

  foo()

  if (import.meta.hot) {
    import.meta.hot.acceptDeps('./foo.js', (newFoo) => {
      // 回调函数将会在 './foo.js' 被更新时触发
      newFoo.foo()
    })

    // 同样我们可以接受一个数组
    import.meta.hot.acceptDeps(['./foo.js', './bar.js'], ([newFooModule, newBarModule]) => {
      // 回调函数将会在这个数组中的模块被更新时触发
    })
  }
  ```

  接收自身更新通知的模块或者接收来自其他模块更新通知的模块可以使用 `hot.dispose` 来清理一些更新过程中产生的副作用

  ```js
  function setupSideEffect() {}

  setupSideEffect()

  if (import.meta.hot) {
    import.meta.hot.dispose((data) => {
      // cleanup side effect
    })
  }
  ```

  完整的API可以查看 [hmr.d.ts](https://github.com/vitejs/vite/blob/master/hmr.d.ts).

  Note that Vite's HMR does not actually swap the originally imported module: if an accepting module re-exports imports from a dep, then it is responsible for updating those re-exports (and these exports must be using `let`). In addition, importers up the chain from the accepting module will not be notified of the change.

  这个简洁的的热替换实现在很多开发场景是足够用的。这使得我们可以跳过昂贵的生成代理模块的过程。

### TypeScript

Vite 支持在Vue 单文件组件中使用 `<script lang="ts">` 导入 `.ts` 文件。

Vite 只进行 `.ts` 文件的转换而不会进行类型检查。它假设类型检查已经在你的 IDE 或者在构建命令中已经被加入进来了(例如你可以在构建脚本中执行 `tsc --noEmit`)。

Vite 使用 [esbuild](https://github.com/evanw/esbuild) 来转换 TypeScript 到 JavaScript 速度相比 `tsc` 快 20-30倍。 热替换的更新时间在浏览器中将小于50ms。

因为 `esbuild` 仅进行转换不附带类型信息。所以它不支持以下功能例如常量枚举以及隐式的类型导入。你必须在 `tsconfig.json` 文件的 `compilerOptions` 选项中设置 `"isolatedModules": true`, 这样 TS 将会警告你这些功能在这个选项下不能被使用。

### CSS / JSON Importing

You can directly import `.css` and `.json` files from JavaScript (including `<script>` tags of `*.vue` files, of course).

- `.json` 文件的内容将会以默认导出的形式导出一个对象

- `.css` 文件将不会导出任何东西除非它是以 `.module.css` 作为后缀名。(查看 [CSS Modules](#css-modules))。 在开发环境下导入它会产生副作用并且注入到页面当中，在生产环境最终会单独打包为 `style.css` 文件。

CSS 和 JSON 的导入都支持热替换功能。

### Asset URL Handling

你可以在 `*.vue` 模版 styles 标签以及 `.css` 文件中引用静态资源，通过绝对的静态目录路径(基于你的项目根目录) 或者相对路径 (基于当前文件)。后者的行为与你之前在 `vue-cli` 或者webpack的 `file-loader` 的使用方式非常像。

所有被引用的资源包括使用绝对路径引用的资源最终都会被复制到打包后的 dist 文件夹当中并且文件名包含 has h值, 没有被引用的文件将不会被复制。与 `vue-cli` 一样，小于 4kb 的图片资源将会以 base64 的形式内联。

所有的静态资源路径引用，包括绝对路径都是基于你当前的工作目录结构。

#### The `public` Directory

项目工程下的 `public` 目录提供不会在源码中引入静态文件资源文件(例如 `robots.txt`), 或者必须保留原始名称的文件(不附带 hash 值)。

`public` 目录中的文件将会被复制到最终的 dist 文件夹当中。

如果要引用 `public` 中的文件需要使用绝对路径，例如 `public/icon.png` 文件在源码中的引用方式为 `/icon.png`。

#### Public Base Path

如果你的项目以嵌套文件夹的形式发布。可以使用 `--base=/your/public/path/` 选项，这样所有的静态资源的路径会被自动重写。

为了动态的路径引用，这里有两种方式提供

- 你可以获得解析后的静态文件路径通过在 JavaScript 中 导入不它们。例如  `import path from './foo.png'` 将会以字符串的形式返回加载路径。

- 如果你需要在云端拼接完整的路径，可以使用注入的全局变量 `import.meta.env.BASE_URL` 它的值等于静态资源的基路径。这个变量在构建过程中是静态的，所以它必须以这种方式出现。 (`import.meta.env['BASE_URL']` 将不会生效)

### PostCSS

Vite 将自动的在 `*.vue` 文件中的所有 styles 标签 以及所有导入的 `.css` 文件中应用 PostCSS。只需要安装必要的插件并且在项目根目录下添加 `postcss.config.js` 配置文件。

### CSS Modules

如果你想使用 CSS Modules 你并不需要配置 PostCSS，因为这已经集成好是开箱即用的。在 `*.vue` 组件中你可以使用 `<stype module>`, 在 `.css` 文件中你可以使用 `*.module.css` 的后缀名便可以以带有 hash 值的形式来导入它们。

### CSS Pre-Processors

因为 Vite 期望你的代码将会运行在现代浏览器中，所以它建议使用原生的 CSS 变量结合 PostCSS 插件来实现 CSSWG 草案 (例如 [postcss-nesting](https://github.com/jonathantneal/postcss-nesting)) 使其变得简洁以及标准化。这意味着，如果你坚持要使用 CSS 预处理，你需要在本地安装预处理器然后使用。

```bash
yarn add -D sass
```

```vue
<style lang="scss">
/* use scss */
</style>
```

同样也可以在 JS 文件中导入

```js
import './style.scss'
```

#### Passing Options to Pre-Processor

> 要求版本大于等于 1.0.0-beta.9+
> 如果你需要修改默认预处理器的配置，你可以使用 config 文件中的 `cssPreprocessOptions` 选项(查看 [Config File](#config-file))
> 例如为你的 less 文件定义一个全局变量

```js
// vite.config.js
module.exports = {
  cssPreprocessOptions: {
    less: {
      modifyVars: {
        'preprocess-custom-color': 'green'
      }
    }
  }
}
```

### JSX

`.jsx` and `.tsx` files are also supported. JSX transpilation is also handled via `esbuild`.

`.jsx` 以及 `.tsx` files 同样是支持的。 JSX 文件同样使用 `esbuild` 来进行转换。

默认与 Vue3 结合的 JSX 配置是开箱即用的 (对 Vue 来说目前还没有针对 JSX 语法的热替换功能)

```jsx
import { createApp } from 'vue'

function App() {
  return <Child>{() => 'bar'}</Child>
}

function Child(_, { slots }) {
  return <div onClick={() => console.log('hello')}>{slots.default()}</div>
}

createApp(App).mount('#app')
```

Currently this is auto-importing a `jsx` compatible function that converts esbuild-produced JSX calls into Vue 3 compatible vnode calls, which is sub-optimal. Vue 3 will eventually provide a custom JSX transform that can take advantage of Vue 3's runtime fast paths.

同时这种写法将会自动导入与 `jsx` 兼容的函数，esbuild 将会转换 JSX 使其成为与 Vue 3 兼容并且能够在虚拟节点中被调用。Vue 3 最终将提供可利用Vue 3的运行时快速的自定义JSX转换。

#### JSX with React/Preact

我们准备了两种方案分别是 `react` 和 `preact`。你可以使用 Vite 执行下列命令来进行方案的选择使用 `--jsx react` or `--jsx preact`。

如果你需要一个自定义的 JSX 编译规则，JSX 支持自定义通过在 CLI 中使用 `--jsx-factory` 以及 `--jsx-fragment` 标志。或者使用 API 提供的 `jsx: { factory, fragment }`。例如你可执行 `vite --jsx-factory=h` 来使用 `h` 作为 JSX 元素被创建时候的函数调用。在配置文件中 (参考 [Config File](#config-file)), 可以通过下面的写法来指定。

```js
// vite.config.js
module.exports = {
  jsx: {
    factory: 'h',
    fragment: 'Fragment'
  }
}
```

在使用 Preact 作为预置的场景下， `h` 已经自动注入在上下文当中，不需要手动的导入。然而这会导致在使用 `.tsx` 结合 Preact 的情况下 TS 为了类型推断期望 `h` 函数能够被显示的导入。在这种情况下，你可以显示的指定 factory 配置来禁止 `h` 函数的自动注入。

### Web Assembly

> 1.0.0-beta.3+

预编译的 `.wasm` 文件能够被直接导入。默认的导出会作为初始化函数返回一个 Promise 对象来导出 wasm 实例对象:

``` js
import init from './example.wasm'

init().then(exports => {
  exports.test()
})
```

init 函数也能够获取到 `imports` 对象作为 `WebAssembly.instantiate` 的第二个参数传递。

``` js
init({
  imports: {
    someFunc: () => { /* ... */ }
  }
}).then(() => { /* ... */ })
```

在生产环境构建时，小于 `assetInlineLimit` 大小限制的 `.wasm` 文件将会以 base64 的形式内联。否则将会复制到 dist 文件夹作为静态资源被获取。

### Inline Web Workers

> 1.0.0-beta.3+

web worker 脚本能够被直接导入只需要在后面加上 `?workder`。默认的导出是一个自定义的 workder 构造函数。


``` js
import MyWorker from './worker?worker'

const worker = new MyWorker()
```

在生产环境构建时，workders 将会以 base64 的形式内联。

worker 脚本同样使用 `import` 而不是 `importScripts()`，在开发环境下这依赖于浏览器的原生支持，并且仅在 Chrome 中能够工作，但是生产环境已经被编译过了。

如果你不希望内联 worker 脚本，你可以替换你的 workder 脚本到 `public` 文件夹，然后初始化 workder 例如 `new Worker('/worker.js')`

### Config File

你可以在当前项目中创建一个 `vite.config.js` 或者 `vite.config.ts` 文件。Vite 将会自动的使用它。你也可以通过 `vite --config my-config.js` 显式的指定配置文件。

除了在 CLI 映射的选项之外，它也支持 `alias`, `transfroms`, `plugins` (将作为配置接口的子集)选项。在文档完善之前参考 [config.ts](https://github.com/vuejs/vite/blob/master/src/node/config.ts) 来获得更全面的信息。

### Custom Blocks

[自定义区块](https://vue-loader.vuejs.org/guide/custom-blocks.html) 在 Vue 的单文件组件中也是支持的。可以通过 `vueCustomBlockTransforms` 选项来指定自定义区块的转换规则:

``` js
// vite.config.js
module.exports = {
  vueCustomBlockTransforms: {
    i18n: ({ code }) => {
      // return transformed code
    }
  }
}
```

### HTTPS/2

通过 `--https` 的方式来启动服务将会自动生成自签名的证书。并且服务将会启用 TLS 以及 HTTP/2。

同样可以通过 `httpsOptions` 选项来自定义签名证书。与 Node 的 `https.ServerOptions` 一样支持接收以下参数 `key`, `cert`, `ca`, `pfx`。

### Dev Server Proxy

你可以使用配置文件中的 `proxy` 选项来自定义本地开发服务的代理功能。Vite 使用 `koa-proxies`](https://github.com/vagusX/koa-proxies), 它底层使用了[`http-proxy`](https://github.com/http-party/node-http-proxy)。键名可以是一个路径，更多的配置可以查看 [here](https://github.com/http-party/node-http-proxy#options)。

用例:

``` js
// vite.config.js
module.exports = {
  proxy: {
    // string shorthand
    '/foo': 'http://localhost:4567/foo',
    // with options
    '/api': {
      target: 'http://jsonplaceholder.typicode.com',
      changeOrigin: true,
      rewrite: path => path.replace(/^\/api/, '')
    }
  }
}
```

### Production Build

Vite does utilize bundling for production builds, because native ES module imports result in waterfall network requests that are simply too punishing for page load time in production.

You can run `vite build` to bundle the app.

Internally, we use a highly opinionated Rollup config to generate the build. The build is configurable by passing on most options to Rollup - and most non-rollup string/boolean options have mapping flags in the CLI (see [build/index.ts](https://github.com/vuejs/vite/blob/master/src/node/build/index.ts) for full details).

### Modes and Environment Variables

模式选项用于指定 `import.meta.env.MODE` 的值，同时正确的环境变量文件将会被加载。

默认存在两种模式:
  - `development` 使用于 `vite` 以及 `vite serve`
  - `production` 使用于 `vite build`

你可以通过 `--mode` 选项来覆盖默认的模式，例如你想以开发模式来进行构建：

```bash
vite build --mode development
```

当执行 `vite` 命令时，环境变量将会从当前目录的以下文件中被加载

```
.env                # 在任何情况下都被加载
.env.local          # 在任何情况下都被加载, 会被 git 忽略
.env.[mode]         # 仅在当前指定的模式下被加载
.env.[mode].local   # 仅在当前指定的模式下被加载, 会被 git 忽略
```

**Note:** only variables prefixed with `VITE_` are exposed to your code. e.g. `VITE_SOME_KEY=123` will be exposed as `import.meta.env.VITE_SOME_KEY`, but `SOME_KEY=123` will not. This is because the `.env` files may be used by some users for server-side or build scripts and may contain sensitive information that should not be exposed in code shipped to browsers.

## API

### Dev Server

You can customize the server using the API. The server can accept plugins which have access to the internal Koa app instance:

```js
const { createServer } = require('vite')

const myPlugin = ({
  root, // project root directory, absolute path
  app, // Koa app instance
  server, // raw http server instance
  watcher // chokidar file watcher instance
}) => {
  app.use(async (ctx, next) => {
    // You can do pre-processing here - this will be the raw incoming requests
    // before vite touches it.
    if (ctx.path.endsWith('.scss')) {
      // Note vue <style lang="xxx"> are supported by
      // default as long as the corresponding pre-processor is installed, so this
      // only applies to <link ref="stylesheet" href="*.scss"> or js imports like
      // `import '*.scss'`.
      console.log('pre processing: ', ctx.url)
      ctx.type = 'css'
      ctx.body = 'body { border: 1px solid red }'
    }

    // ...wait for vite to do built-in transforms
    await next()

    // Post processing before the content is served. Note this includes parts
    // compiled from `*.vue` files, where <template> and <script> are served as
    // `application/javascript` and <style> are served as `text/css`.
    if (ctx.response.is('js')) {
      console.log('post processing: ', ctx.url)
      console.log(ctx.body) // can be string or Readable stream
    }
  })
}

createServer({
  configureServer: [myPlugin]
}).listen(3000)
```

### Build

Check out the full options interface in [build/index.ts](https://github.com/vuejs/vite/blob/master/src/node/build/index.ts).

```js
const { build } = require('vite')

;(async () => {
  // All options are optional.
  // check out `src/node/build/index.ts` for full options interface.
  const result = await build({
    rollupInputOptions: {
      // https://rollupjs.org/guide/en/#big-list-of-options
    },
    rollupOutputOptions: {
      // https://rollupjs.org/guide/en/#big-list-of-options
    },
    rollupPluginVueOptions: {
      // https://github.com/vuejs/rollup-plugin-vue/tree/next#options
    }
    // ...
  })
})()
```

## How and Why

### How is This Different from `vue-cli` or Other Bundler-based Solutions?

The primary difference is that for Vite there is no bundling during development. The ES Import syntax in your source code is served directly to the browser, and the browser parses them via native `<script module>` support, making HTTP requests for each import. The dev server intercepts the requests and performs code transforms if necessary. For example, an import to a `*.vue` file is compiled on the fly right before it's sent back to the browser.

There are a few advantages of this approach:

- Since there is no bundling work to be done, the server cold start is extremely fast.

- Code is compiled on demand, so only code actually imported on the current screen is compiled. You don't have to wait until your entire app to be bundled to start developing. This can be a huge difference in apps with dozens of screens.

- Hot module replacement (HMR) performance is decoupled from the total number of modules. This makes HMR consistently fast no matter how big your app is.

Full page reload could be slightly slower than a bundler-based setup, since native ES imports result in network waterfalls with deep import chains. However since this is local development, the difference should be trivial compared to actual compilation time. (There is no compile cost on page reload since already compiled files are cached in memory.)

Finally, because compilation is still done in Node, it can technically support any code transforms a bundler can, and nothing prevents you from eventually bundling the code for production. In fact, Vite provides a `vite build` command to do exactly that so the app doesn't suffer from network waterfall in production.

### How is This Different from [es-dev-server](https://open-wc.org/developing/es-dev-server.html)?

`es-dev-server` is a great project and we did take some inspiration from it when refactoring Vite in the early stages. That said, here is why Vite is different from `es-dev-server` and why we didn't just implement Vite as a middleware for `es-dev-server`:

- One of Vite's primary goal was to support Hot Module Replacement, but `es-dev-server` internals is a bit too opaque to get this working nicely via a middleware.

- Vite aims to be a single tool that integrates both the dev and the build process. You can use Vite to both serve and bundle the same source code, with zero configuration.

- Vite is more opinionated on how certain types of imports are handled, e.g. `.css` and static assets. The handling is similar to `vue-cli` for obvious reasons.

### How is This Different from [Snowpack](https://www.snowpack.dev/)?

Both Snowpack v2 and Vite offer native ES module import based dev servers. Vite's dependency pre-optimization is also heavily inspired by Snowpack v1. Both projects share similar performance characteristics when it comes to development feedback speed. Some notable differences are:

- Vite was created to tackle native ESM-based HMR. When Vite was first released with working ESM-based HMR, there was no other project actively trying to bring native ESM based HMR to production.

  Snowpack v2 initially did not offer HMR support but added it in a later release, making the scope of two projects much closer. Vite and Snowpack has collaborated on a common API spec for ESM HMR, but due to the constraints of different implementation strategies, the two projects still ship slightly different APIs.

- Both solutions can also bundle the app for production, but Vite uses Rollup with built-in config while Snowpack delegates it to Parcel/webpack via additional plugins. Vite will in most cases build faster and produce smaller bundles. In addition, a tighter integration with the bundler makes it easier to author Vite transforms and plugins that modify dev/build configs at the same.

- Vue support is a first-class feature in Vite. For example, Vite provides a much more fine-grained HMR integration with Vue, and the build config is fined tuned to produce the most efficient bundle.

## Contribution

See [Contributing Guide](https://github.com/vitejs/vite/tree/master/.github/contributing.md).


## Trivia

[vite](https://en.wiktionary.org/wiki/vite) is the french word for "fast" and is pronounced `/vit/`.

## License

MIT

[npm-img]: https://img.shields.io/npm/v/vite.svg
[npm-url]: https://npmjs.com/package/vite
[node-img]: https://img.shields.io/node/v/vite.svg
[node-url]: https://nodejs.org/en/about/releases/
[unix-ci-img]: https://circleci.com/gh/vitejs/vite.svg?style=shield
[unix-ci-url]: https://app.circleci.com/pipelines/github/vitejs/vite
[windows-ci-img]: https://ci.appveyor.com/api/projects/status/0q4j8062olbcs71l/branch/master?svg=true
[windows-ci-url]: https://ci.appveyor.com/project/yyx990803/vite/branch/master
