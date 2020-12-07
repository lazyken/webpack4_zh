# Code Splitting 代码分离

> 本指南继续沿用起步和管理输出中的代码示例。请确保你至少已熟悉其中提供的示例。  
> This guide extends the examples provided in `Getting Started` and `Output Management`. Please make sure you are at least familiar with the examples provided in them.

代码分离是 webpack 中最引人注目的特性之一。此特性能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间。

> Code splitting is one of the most compelling features of webpack. This feature allows you to split your code into various bundles which can then be loaded on demand or in parallel. It can be used to achieve smaller bundles and control resource load prioritization which, if used correctly, can have a major impact on load time.

有三种常用的代码分离方法：

- 入口起点：使用 `entry` 配置手动地分离代码。
- 防止重复：使用 `SplitChunksPlugin` 去重和分离 chunk。
- 动态导入：通过模块的内联函数调用来分离代码。

> There are three general approaches to code splitting available:
>
> - Entry Points: Manually split code using entry configuration.
> - Prevent Duplication: Use the SplitChunksPlugin to dedupe and split chunks.
> - Dynamic Imports: Split code via inline function calls within modules.

## Entry Points 入口起点

这是迄今为止最简单、最直观的分离代码的方式。不过，这种方式手动配置较多，并有一些陷阱，我们将会解决这些问题。先来看看如何从 main bundle 中分离另一个模块：

> This is by far the easiest and most intuitive way to split code. However, it is more manual and has some pitfalls we will go over. Let's take a look at how we might split another module from the main bundle:

project

```text
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
+ |- another-module.js
|- /node_modules
```

another-module.js

```js
import _ from 'lodash';

console.log(_.join(['Another', 'module', 'loaded!'], ' '));
```

webpack.config.js

```js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: {
    index: './src/index.js',
+   another: './src/another-module.js',
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

这将产生以下构建结果：

> This will yield the following build result:

```text
...
            Asset     Size   Chunks             Chunk Names
another.bundle.js  550 KiB  another  [emitted]  another
  index.bundle.js  550 KiB    index  [emitted]  index
Entrypoint index = index.bundle.js
Entrypoint another = another.bundle.js
...
```

正如前面提到的，这种方法存在一些问题:

- 如果入口 chunks 之间包含重复的模块，那些重复模块都会被引入到各个 bundle 中。
- 这种方法不够灵活，并且不能将核心应用程序逻辑进行动态拆分代码。

以上两点中，第一点对我们的示例来说无疑是个问题，因为之前我们在 `./src/index.js` 中也引入过 `lodash`，这样就在两个 bundle 中造成重复引用。接着，我们通过使用 `CommonsChunkPlugin` 来移除重复的模块。

> As mentioned there are some pitfalls to this approach:
>
> - If there are any duplicated modules between entry chunks they will be included in both bundles.
> - It isn't as flexible and can't be used to dynamically split code with the core application logic.
>
> The first of these two points is definitely an issue for our example, as `lodash` is also imported within `./src/index.js` and will thus be duplicated in both bundles. Let's remove this duplication by using the `SplitChunksPlugin`.

## Prevent Duplication 防止重复

SplitChunksPlugin 插件可以将公共的依赖模块提取到已有的入口 chunk 中，或者提取到一个新生成的 chunk。让我们使用这个插件，将之前的示例中重复的 lodash 模块去除：

> webpack v4 legato 中已删除 CommonsChunkPlugin。 要了解最新版本中如何处理块，请查看 SplitChunksPlugin。

> The SplitChunksPlugin allows us to extract common dependencies into an existing entry chunk or an entirely new chunk. Let's use this to de-duplicate the lodash dependency from the previous example:
> The CommonsChunkPlugin has been removed in webpack v4 legato. To learn how chunks are treated in the latest version, check out the SplitChunksPlugin.

webpack.config.js

```js
  const path = require('path');

  module.exports = {
    mode: 'development',
    entry: {
      index: './src/index.js',
      another: './src/another-module.js',
    },
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist'),
    },
+   optimization: {
+     splitChunks: {
+       chunks: 'all',
+     },
+   },
  };
```

这里我们使用 `optimization.splitChunks` 之后，现在应该可以看出，从 index.bundle.js 和 another.bundle.js 中删除了重复的依赖项。需要注意的是，该插件将 lodash 分离到单独的 chunk，并且将其从 main bundle 中移除，减轻了大小。执行 npm run build 查看效果：

> With the optimization.splitChunks configuration option in place, we should now see the duplicate dependency removed from our index.bundle.js and another.bundle.js. The plugin should notice that we've separated lodash out to a separate chunk and remove the dead weight from our main bundle. Let's do an npm run build to see if it worked:

```text
...
                          Asset      Size                 Chunks             Chunk Names
              another.bundle.js  5.95 KiB                another  [emitted]  another
                index.bundle.js  5.89 KiB                  index  [emitted]  index
vendors~another~index.bundle.js   547 KiB  vendors~another~index  [emitted]  vendors~another~index
Entrypoint index = vendors~another~index.bundle.js index.bundle.js
Entrypoint another = vendors~another~index.bundle.js another.bundle.js
...
```

以下是由社区提供的，一些对于代码分离很有帮助的插件和 loaders：

- `mini-css-extract-plugin`: 用于将 CSS 从主应用程序中分离。
- `bundle-loader`: 用于分离代码和延迟加载生成的 bundle。
- `promise-loader`: 类似于 bundle-loader ，但是使用的是 promises。

> Here are some other useful plugins and loaders provided by the community for splitting code:
>
> - mini-css-extract-plugin: Useful for splitting CSS out from the main application.
> - bundle-loader: Used to split code and lazy load the resulting bundles.
> - promise-loader: Similar to the bundle-loader but uses promises.

## Dynamic Imports 动态导入

当涉及到动态代码拆分时，webpack 提供了两个类似的技术。对于动态导入，第一种，也是优先选择的方式是，使用符合 ECMAScript 提案 的 `import()` 语法。第二种，则是使用 webpack 特定的 `require.ensure`。让我们尝试使用第一种……

> Two similar techniques are supported by webpack when it comes to dynamic code splitting. The first and recommended approach is to use the import() syntax that conforms to the ECMAScript proposal for dynamic imports. The legacy, webpack-specific approach is to use require.ensure. Let's try using the first of these two approaches...

> `import()` 调用会在内部用到 `promises`。如果在旧有版本浏览器中使用 `import()`，记得使用 一个 polyfill 库（例如 `es6-promise` 或 `promise-polyfill`），来 shim `Promise`。
>
> `import()` calls use promises internally. If you use `import()` with older browsers, remember to shim `Promise` using a polyfill such as `es6-promise` or `promise-polyfill`.

在我们开始本节之前，先从配置中移除掉多余的 `entry` 和 `optimization.splitChunks`，因为接下来的演示中并不需要它们：

> Before we start, let's remove the extra `entry` and `optimization.splitChunks` from our config as they won't be needed for this next demonstration:

webpack.config.js

```js
  const path = require('path');

  module.exports = {
    mode: 'development',
    entry: {
      index: './src/index.js',
-     another: './src/another-module.js',
    },
    output: {
      filename: '[name].bundle.js',
+     chunkFilename: '[name].bundle.js',
      publicPath: 'dist/',
      path: path.resolve(__dirname, 'dist'),
    },
-   optimization: {
-     splitChunks: {
-       chunks: 'all',
-     },
-   },
  };
```

注意，这里使用了 `chunkFilename`，它决定非入口 chunk 的名称。想了解 `chunkFilename` 更多信息，请查看 [output 相关文档]()。接着，更新我们的项目，移除掉那些现在不会用到的文件:

> Note the use of `chunkFilename`, which determines the name of non-entry chunk files. For more information on `chunkFilename`, see output documentation. We'll also update our project to remove the now unused files:

project

```test
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
- |- another-module.js
|- /node_modules
```

现在，我们不再使用静态导入 `lodash`，而是通过使用动态导入来分离一个 chunk：

> Now, instead of statically importing lodash, we'll use dynamic importing to separate a chunk:

src/index.js

```js
- import _ from 'lodash';
-
- function component() {
+ function getComponent() {
-   const element = document.createElement('div');
-
-   // Lodash, now imported by this script
-   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   return import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default: _ }) => {
+     const element = document.createElement('div');
+
+     element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+
+     return element;
+
+   }).catch(error => 'An error occurred while loading the component');
  }

- document.body.appendChild(component());
+ getComponent().then(component => {
+   document.body.appendChild(component);
+ })
```

我们需要 `default` 的原因是，自 webpack 4 起，在导入 CommonJS 模块时，导入将不再解析为 `module.exports` 的值，而是将为 CommonJS 模块创建一个人为的命名空间对象。 有关其背后原因的更多信息，请阅读 `webpack 4: import() and CommonJs`

> The reason we need default is that since webpack 4, when importing a CommonJS module, the import will no longer resolve to the value of module.exports, it will instead create an artificial namespace object for the CommonJS module. For more information on the reason behind this, read webpack 4: import() and CommonJs

注意，在注释中使用了 `webpackChunkName`。这样做会导致我们的 bundle 被命名为 `lodash.bundle.js` ，而不是 `[id].bundle.js` 。想了解更多关于 `webpackChunkName` 和其他可用选项，请查看 `import()` 相关文档。让我们执行 webpack，查看 `lodash` 是否会分离到一个单独的 bundle：

> Note the use of `webpackChunkName` in the comment. This will cause our separate bundle to be named `lodash.bundle.js` instead of just `[id].bundle.js`. For more information on `webpackChunkName` and the other available options, see the `import()` documentation. Let's run webpack to see `lodash` separated out to a separate bundle:

```text
...
                   Asset      Size          Chunks             Chunk Names
         index.bundle.js  7.88 KiB           index  [emitted]  index
vendors~lodash.bundle.js   547 KiB  vendors~lodash  [emitted]  vendors~lodash
Entrypoint index = index.bundle.js
...
```

由于 `import()` 会返回一个 `promise`，因此它可以和 `async` 函数一起使用。但是，需要使用像 Babel 这样的预处理器和`Syntax Dynamic Import Babel Plugin`。下面是如何通过 `async` 函数简化代码：

> As `import()` returns a promise, it can be used with `async` functions. However, this requires using a pre-processor like Babel and the `Syntax Dynamic Import Babel Plugin`. Here's how it would simplify the code:

src/index.js

```js
- function getComponent() {
+ async function getComponent() {
-   return import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default: _ }) => {
-     const element = document.createElement('div');
-
-     element.innerHTML = _.join(['Hello', 'webpack'], ' ');
-
-     return element;
-
-   }).catch(error => 'An error occurred while loading the component');
+   const element = document.createElement('div');
+   const { default: _ } = await import(/* webpackChunkName: "lodash" */ 'lodash');
+
+   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+
+   return element;
  }

  getComponent().then(component => {
    document.body.appendChild(component);
  });
```

> 当您稍后可能需要基于计算变量导入特定模块时，可以为 import() 提供一个动态表达式。
> It is possible to provide a `dynamic expression` to `import()` when you might need to import specific module based on a computed variable later.

## Prefetching/Preloading modules 预请求/预加载模块

`webpack 4.6.0+` 增加了对预请求和预加载的支持。  
在声明您的导入时使用这些内联指令可以使 webpack 输出`"Resource Hint"`，它告诉浏览器：

- prefetch 预请求：将来可能需要一些导航资源
- preload 预加载：当前导航期间可能需要资源

简单的预请求示例可以包含一个 `HomePage` 组件，该组件呈现一个 `LoginButton` 组件，然后按需在单击后加载 `LoginModal` 组件。

> webpack 4.6.0+ adds support for prefetching and preloading.
>
> Using these inline directives while declaring your imports allows webpack to output “Resource Hint” which tells the browser that for:
>
> - prefetch: resource is probably needed for some navigation in the future
> - preload: resource might be needed during the current navigation
>
> Simple prefetch example can be having a `HomePage` component, which renders a `LoginButton` component which then on demand loads a `LoginModal` component after being clicked.

LoginButton.js

```js
//...
import(/* webpackPrefetch: true */ 'LoginModal');
```

这会使`<link rel ="prefetch" href ="login-modal-chunk.js">`附加在页面顶部，这将指示浏览器在空闲时间预请求 `login-modal-chunk.js`文件

> This will result in <link rel="prefetch" href="login-modal-chunk.js"> being appended in the head of the page, which will instruct the browser to prefetch in idle time the login-modal-chunk.js file.

> 一旦 `parent chunk` 被加载，webpack 将添加预请求提示。
> webpack will add the prefetch hint once the parent chunk has been loaded.

与预请求 `prefetch` 相比， 预加载 `Preload` 指令有很多区别：

- 预加载的 chunk 与 `parent chunk` 并行同时开始加载。 预请求的 chunk 在 `parent chunk` 完成加载后开始加载。
- 预加载的 chunk 具有中等优先级，可以立即下载。 预请求的 chunk 在浏览器空闲时下载。
- 预加载的 chunk 应该被 `parent chunk` 立即请求。 预请求的 chunk 可以在将来的任何时候使用。
- 浏览器的支持也不同。

一个简单的预加载示例，它包含一个 `Component`，该 `Component` 始终依赖于一个应放在单独 chunk 中的大型 lib。  
让我们设想一个 `ChartComponent` 组件，它依赖巨大的 `ChartingLibrary` 。它会渲染出一个 `LoadingIndicator`，并立即按需导入 `ChartingLibrary`：

> Preload directive has a bunch of differences compared to prefetch:
>
> - A preloaded chunk starts loading in parallel to the parent chunk. A prefetched chunk starts after the parent chunk finishes loading.
> - A preloaded chunk has medium priority and is instantly downloaded. A prefetched chunk is downloaded while the browser is idle.
> - A preloaded chunk should be instantly requested by the parent chunk. A prefetched chunk can be used anytime in the future.
> - Browser support is different.
>
> Simple preload example can be having a Component which always depends on a big library that should be in a separate chunk.
>
> Let's imagine a component ChartComponent which needs huge ChartingLibrary. It displays a LoadingIndicator when rendered and instantly does an on demand import of ChartingLibrary:

ChartComponent.js

```js
//...
import(/* webpackPreload: true */ 'ChartingLibrary');
```

当使用 `ChartComponent` 的页面被请求时，也会通过<link rel =“ preload”>请求 `charting-library-chunk`。 假设 `page-chunk` 较小并且完成较快，该页面将显示一个 `LoadingIndicator`，直到已经请求的 `charting-library-chunk` 完成为止。 这将增加一点加载时间，因为它只需要一个 round-trip 而不是两个。 特别是在高延迟环境中。

> When a page which uses the `ChartComponent` is requested, the `charting-library-chunk` is also requested via `<link rel="preload">`. Assuming the page-chunk is smaller and finishes faster, the page will be displayed with a `LoadingIndicator`, until the already requested charting-library-chunk finishes. This will give a little load time boost since it only needs one round-trip instead of two. Especially in high-latency environments.

> 错误地使用 webpackPreload 实际上会影响性能，因此使用时请务必小心。
> Using webpackPreload incorrectly can actually hurt performance, so be careful when using it.

## Bundle Analysis

一旦开始分割代码，分析输出来检查模块在哪里结束将很有用。官方分析工具是一个很好的入门。下面是一些社区支持(community-supported)的可选工具：

- webpack-chart：用于 webpack 统计信息的交互式饼图。
- webpack-visualizer：可视化地分 bundles，以查看哪些模块占用了空间，哪些可能是重复的。
- webpack-bundle-analyzer：一个 CLI 插件，将 bundle 内容展示为方便的交互式可缩放树形图。
- webpack bundle optimize helper：此工具将分析 bundle，并为您提供可行的建议，以改善 bundle 的大小。
- bundle-stats：生成 bundle 报告（bundle 大小，资源，模块），并比较不同版本之间的结果。

> Once you start splitting your code, it can be useful to analyze the output to check where modules have ended up. The official analyze tool is a good place to start. There are some other community-supported options out there as well:

- webpack-chart: Interactive pie chart for webpack stats.
- webpack-visualizer: Visualize and analyze your bundles to see which modules are taking up space and which might be duplicates.
- webpack-bundle-analyzer: A plugin and CLI utility that represents bundle content as a convenient interactive zoomable treemap.
- webpack bundle optimize helper: This tool will analyze your bundle and give you actionable suggestions on what to improve to reduce your bundle size.
- bundle-stats: Generate a bundle report(bundle size, assets, modules) and compare the results between different builds.

## Next Steps 下一步

关于「如何在真正的应用程序和[缓存]()中 `import()` 导入」以及学习「如何更加高效地分离代码」的具体示例，请查看[懒加载]()。

> See Lazy Loading for a more concrete example of how can be used in a real application and Caching to learn how to split code more effectively.

## Further Reading

- [`<link rel=”prefetch/preload”>` in webpack](https://medium.com/webpack/link-rel-prefetch-preload-in-webpack-51a52358f84c)
- [Preload, Prefetch And Priorities in Chrome](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf)
- [Preloading content with rel="preload"](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)
