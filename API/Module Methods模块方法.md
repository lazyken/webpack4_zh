# Module Methods 模块方法

本节涵盖了使用 webpack 编译代码的所有方法。在 webpack 打包应用程序时，你可以选择各种模块语法风格，包括 [ES6](https://en.wikipedia.org/wiki/ECMAScript#6th_Edition_-_ECMAScript_2015), [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 和 [AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition)。

> 虽然 webpack 支持多种模块语法，但我们建议尽量遵循一致的语法，避免一些奇怪的行为和 bug。这是一个混合使用了 ES6 和 CommonJS 的[示例](https://github.com/webpack/webpack.js.org/issues/552)，但我们确定还有其他的 BUG 会产生。

<details>
<summary>英文</summary>

This section covers all methods available in code compiled with webpack. When using webpack to bundle your application, you can pick from a variety of module syntax styles including [ES6](https://en.wikipedia.org/wiki/ECMAScript#6th_Edition_-_ECMAScript_2015), [CommonJS](https://en.wikipedia.org/wiki/CommonJS), and [AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition).

> While webpack supports multiple module syntaxes, we recommend following a single syntax for consistency and to avoid odd behaviors/bugs. Here's [one example](https://github.com/webpack/webpack.js.org/issues/552) of mixing ES6 and CommonJS, but there are surely others.

</details>

## ES6 (Recommended) ES6（推荐）

webpack 2 支持原生的 ES6 模块语法，意味着你可以无须额外引入 babel 这样的工具，就可以使用 import 和 export。但是注意，如果使用其他的 ES6+ 特性，仍然需要引入 babel。webpack 支持以下的方法：

<details>
<summary>英文</summary>

Version 2 of webpack supports ES6 module syntax natively, meaning you can use `import` and `export` without a tool like babel to handle this for you. Keep in mind that you will still probably need babel for other ES6+ features. The following methods are supported by webpack:

</details>

### `import`

通过 `import` 以静态的方式，导入另一个通过 `export` 导出的模块。

<details>
<summary>英文</summary>

Statically `import` the `exports` of another module.

</details>

```js
import MyModule from './my-module.js';
import { NamedExport } from './other-module.js';
```

> 这里的关键词是**静态的**。标准的 `import` 语句中，模块语句中不能以「具有逻辑或含有变量」的动态方式去引入其他模块。关于 import 的更多信息和 `import()` 动态用法，请查看这里的[说明](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)。

<details>
<summary>英文</summary>

> The keyword here is **statically**. A normal import statement cannot be used dynamically within other logic or contain variables. See the [spec](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) for more information and `import()` below for dynamic usage.

</details>

### `export`

导出为`默认`模块或者具名模块

<details>
<summary>英文</summary>

Export anything as a `default` or named export.

</details>

```js
// Named exports
export var Count = 5;
export function Multiply(a, b) {
  return a * b;
}

// Default export
export default {
  // Some data...
};
```

### `import()` 动态加载模块 <a id="#import-1"></a>

`function(string path):Promise`

动态地加载模块。调用 `import()` 之处，被作为分离的模块起点，意思是，被请求的模块和它引用的所有子模块，会分离到一个单独的 chunk 中。

> [ES2015 loader 规范](https://whatwg.github.io/loader/) 定义了 `import()` 方法，可以在运行时动态地加载 ES2015 模块。

<details>
<summary>英文</summary>

`function(string path):Promise`

Dynamically load modules. Calls to `import()` are treated as split points, meaning the requested module and its children are split out into a separate chunk.

> The [ES2015 Loader spec](https://whatwg.github.io/loader/) defines `import()` as method to load ES2015 modules dynamically on runtime.

</details>

```js
if (module.hot) {
  import('lodash').then((_) => {
    // Do something with lodash (a.k.a '_')...
  });
}
```

> import() 特性依赖于内置的 `Promise`。如果想在低版本浏览器使用 import()，记得使用像 `es6-promise` 或者 `promise-polyfill` 这样 polyfill 库，来预先填充(shim) `Promise` 环境。

<details>
<summary>英文</summary>

> This feature relies on [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) internally. If you use `import()` with older browsers, remember to shim `Promise` using a polyfill such as [es6-promise](https://github.com/stefanpenner/es6-promise) or [promise-polyfill](https://github.com/taylorhakes/promise-polyfill).

</details>

## Dynamic expressions in import() / import()函数中的动态表达式

完全动态的语句（例如`import（foo`），因为 webpack 至少需要一些文件的路径信息，而 `foo` 可能是系统或项目中任何文件的任何路径，因此 `foo` 将会解析失败（因为 `foo` 可能是系统或项目中任何文件的任何路径）。  
`import()` 必须至少包含模块位于何处的路径信息，所以打包应当限制在一个指定目录或一组文件中。因此，当你使用动态表达式时，每个在`import()`中可能被请求的模块都会被打包。 例如，`` import(`./locale/${language}.json`) ``会导致 `.locale` 目录中的每个 `.json` 文件都被打包到新的 chunk 中。 在运行时，当计算出变量 `language` 时，任何文件（如 `english.json` 或 `german.json`）都可能会被用到。

<details>
<summary>英文</summary>

It is not possible to use a fully dynamic import statement, such as `import(foo)`. Because `foo` could potentially be any path to any file in your system or project.

The `import()` must contain at least some information about where the module is located. Bundling can be limited to a specific directory or set of files so that when you are using a dynamic expression every module that could potentially be requested on an `import()` call is included. For example, `` import(`./locale/${language}.json`) `` will cause every `.json` file in the `./locale` directory to be bundled into the new chunk. At run time, when the variable `language` has been computed, any file like `english.json` or `german.json` will be available for consumption.

</details>

```js
// imagine we had a method to get language from cookies or other storage
const language = detectVisitorLanguage();
import(`./locale/${language}.json`).then((module) => {
  // do something with the translations
});
```

> 使用 [`webpackInclude` 和 `webpackExclude`](https://github.com/lazyken/webpack4_zh/blob/main/API/Module%20Methods%E6%A8%A1%E5%9D%97%E6%96%B9%E6%B3%95.md#magic-comments) 选项可让您通过添加正则表达式，来减少 webpack 打包的文件数量。（具体参考下方的[**魔法注释**](https://github.com/lazyken/webpack4_zh/blob/main/API/Module%20Methods%E6%A8%A1%E5%9D%97%E6%96%B9%E6%B3%95.md#magic-comments)）

<details>
<summary>英文</summary>

> Using the [`webpackInclude` and `webpackExclude`](https://github.com/lazyken/webpack4_zh/blob/main/API/Module%20Methods%E6%A8%A1%E5%9D%97%E6%96%B9%E6%B3%95.md#magic-comments) options allows you to add regex patterns that reduce the number of files that webpack will bundle for this import.

</details>

## Magic Comments / 魔法注释 <a id="magic-comments"></a>

内联注释可以完成一些特定工作。可以通过给 import 添加注释来做一些事情，比如为 chunk 命名或选择不同的模式。下面的代码展示了支持的魔法注释的完整列表，及解释这些注释的作用。

<details>
<summary>英文</summary>

Inline comments to make features work. By adding comments to the import, we can do things such as name our chunk or select different modes. For a full list of these magic comments see the code below followed by an explanation of what these comments do.

</details>

```js
// Single target
import(
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  'module'
);

// Multiple possible targets
import(
  /* webpackInclude: /\.json$/ */
  /* webpackExclude: /\.noimport\.json$/ */
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  /* webpackPrefetch: true */
  /* webpackPreload: true */
  `./locale/${language}`
);
```

```js
import(/* webpackIgnore: true */ 'ignored-module.js');
```

`webpackIgnore`: 设为 true 时，禁用动态导入解析.

> 注意，将 `webpackIgnore` 设置为 `true` 会选择退出代码分割(code splitting)。

`webpackChunkName`：新 chunk 的名称。 从 webpack 2.6.0 开始，占位符`[index]`和`[request]`在给定的字符串中分别支持递增的数字或实际解析的文件名。 添加此注释将使我们单独的块命名为`[my-chunk-name].js`，而不是`[id].js`。

webpackMode：从 webpack 2.6.0 开始，可以指定以不同的模式解析动态导入。 支持以下选项：

- `"lazy"`(默认)：为每个 `import()` 导入的模块，生成一个可延迟加载(lazy-loadable) chunk。
- `"lazy-once"`：生成一个可以满足所有 `import()` 调用的单个可延迟加载(lazy-loadable) chunk。此 chunk 将在第一次 `import()` 调用时获取，随后的 `import()` 调用将使用相同的网络响应。注意，这种模式仅在部分动态语句中有意义，例如 `` import(`./locales/${language}.json`) ``，其中可能含有多个被请求的模块路径。
- `"eager"`：不会生成额外的 chunk，所有模块都被当前 chunk 引入，并且没有额外的网络请求。仍然会返回 `Promise`，但是是 resolved 状态。和静态导入相对比，在调用 `import()` 完成之前，该模块不会被执行。
- `"weak"`：如果该模块函数已经以其他方式加载（即，另一个 chunk 导入过此模块，或包含模块的 script 脚本被加载），尝试加载模块。仍然会返回 `Promise`，但是只有在客户端上已经有该 chunk 时才成功解析。如果该模块不可用，`Promise` 将会是 rejected 状态，并且网络请求永远不会执行。当需要的 chunks 总是在初始请求中(嵌入到页面中)被手动提供时，这对于大部分渲染很有用，但当应用导航将触发一个最初没有提供的导入时，则不可用。

`webpackPrefetch`：告诉浏览器将来可能需要某种资源来进行某些导航。 查阅指南，了解有关 [webpackPrefetch](https://github.com/lazyken/webpack4_zh/blob/main/Guides%E6%8C%87%E5%8D%97/CodeSplitting%E4%BB%A3%E7%A0%81%E5%88%86%E7%A6%BB.md/#prefetchingpreloading-modules) 如何工作的更多信息。
`webpackPreload`：告诉浏览器在当前导航期间可能需要该资源。 查阅指南，了解有关 [webpackPreload](https://github.com/lazyken/webpack4_zh/blob/main/Guides%E6%8C%87%E5%8D%97/CodeSplitting%E4%BB%A3%E7%A0%81%E5%88%86%E7%A6%BB.md/#prefetchingpreloading-modules) 如何工作的更多信息。

> 请注意，所有选项都可以组合起来使用，如 `/* webpackMode: "lazy-once", webpackChunkName: "all-i18n-data" */`，它包装在 JavaScript 对象中，并使用[node VM](https://nodejs.org/dist/latest-v8.x/docs/api/vm.html) 执行。 您不需要添加大括号。

`webpackInclude`：一个正则表达式，在导入解析期间将使用正则进行匹配。仅将匹配的模块被打包。
`webpackExclude`：一个正则表达式，在导入解析期间将使用正则进行匹配。任何匹配的模块都不会被打包。

> 请注意，webpackInclude 和 webpackExclude 选项不会干扰前缀。 例如：`./locale`。

> 在 webpack 中使用 [System.import](https://github.com/webpack/webpack/issues/2163) 不符合规范建议，因此在 webpack [2.1.0-beta.28](https://github.com/webpack/webpack/releases/tag/v2.1.0-beta.28) 中已弃用它，转而使用 `import()`。

<details>
<summary>英文</summary>

`webpackIgnore`: Disables dynamic import parsing when set to `true`.

> Note that setting `webpackIgnore` to `true` opts out of code splitting.

`webpackChunkName`: A name for the new chunk. Since webpack 2.6.0, the placeholders `[index]` and `[request]` are supported within the given string to an incremented number or the actual resolved filename respectively. Adding this comment will cause our separate chunk to be named [my-chunk-name].js instead of [id].js.

`webpackMode`: Since webpack 2.6.0, different modes for resolving dynamic imports can be specified. The following options are supported:

- `"lazy"` (default): Generates a lazy-loadable chunk for each `import()`ed module.
- `"lazy-once"`: Generates a single lazy-loadable chunk that can satisfy all calls to `import()`. The chunk will be fetched on the first call to `import()`, and subsequent calls to `import()` will use the same network response. Note that this only makes sense in the case of a partially dynamic statement, e.g. `` import(`./locales/${language}.json`) ``, where there are multiple module paths that could potentially be requested.
- `"eager"`: Generates no extra chunk. All modules are included in the current chunk and no additional network requests are made. A `Promise` is still returned but is already resolved. In contrast to a static import, the module isn't executed until the call to `import()` is made.
- `"weak"`: Tries to load the module if the module function has already been loaded in some other way (e.g. another chunk imported it or a script containing the module was loaded). A `Promise` is still returned, but only successfully resolves if the chunks are already on the client. If the module is not available, the `Promise` is rejected. A network request will never be performed. This is useful for universal rendering when required chunks are always manually served in initial requests (embedded within the page), but not in cases where app navigation will trigger an import not initially served.

`webpackPrefetch`: Tells the browser that the resource is probably needed for some navigation in the future. Check out the guide for more information on [how webpackPrefetch works](https://v4.webpack.js.org/guides/code-splitting/#prefetchingpreloading-modules).

`webpackPreload`: Tells the browser that the resource might be needed during the current navigation. Check out the guide for more information on [how webpackPreload works](https://v4.webpack.js.org/guides/code-splitting/#prefetchingpreloading-modules).

> Note that all options can be combined like so `/* webpackMode: "lazy-once", webpackChunkName: "all-i18n-data" */`. This is wrapped in a JavaScript object and executed using [node VM](https://nodejs.org/dist/latest-v8.x/docs/api/vm.html). You do not need to add curly brackets.

`webpackInclude`: A regular expression that will be matched against during import resolution. Only modules that match **will be bundled**.

`webpackExclude`: A regular expression that will be matched against during import resolution. Any module that matches **will not be bundled**.

> Note that `webpackInclude` and `webpackExclude` options do not interfere with the prefix. eg: `./locale`.

> The use of `System.import` in webpack [did not fit the proposed spec](), so it was deprecated in webpack [2.1.0-beta.28]() in favor of `import()`.

</details>

## CommonJS

CommonJS 的目标是为浏览器外部的 JavaScript 指定一个环境。 webpack 支持以下 CommonJS 方法：

<details>
<summary>英文</summary>

The goal of CommonJS is to specify an ecosystem for JavaScript outside the browser. The following CommonJS methods are supported by webpack:

</details>

### `require`

```ts
require(dependency: String)
```

以同步的方式检索其他模块的导出。由编译器(compiler)来确保依赖项在最终输出 bundle 中可用。

<details>
<summary>英文</summary>

Synchronously retrieve the exports from another module. The compiler will ensure that the dependency is available in the output bundle.

</details>

```js
var $ = require('jquery');
var myModule = require('my-module');
```

以异步的方式使用，可能不会达到预期的效果。

<details>
<summary>英文</summary>

> Using it asynchronously may not have the expected effect.

</details>

### `require.resolve`

```ts
require.resolve(dependency: String);
```

以同步的方式获取模块的 ID。由编译器(compiler)来确保依赖项在最终输出 bundle 中可用。更多关于模块的信息，请点击这里 [module.id](https://www.webpackjs.com/api/module-variables#module-id-commonjs-)。

> webpack 中模块 ID 是一个数字（而在 NodeJS 中是一个字符串 -- 也就是文件名）

<details>
<summary>英文</summary>

Synchronously retrieve a module's ID. The compiler will ensure that the dependency is available in the output bundle. See [module.id](https://v4.webpack.js.org/api/module-variables/#moduleid-commonjs) for more information.

> Module ID is a number in webpack (in contrast to NodeJS where it is a string -- the filename).

</details>

### `require.cache`

多处引用同一个模块，最终只会产生一次模块执行和一次导出。所以，会在运行时(runtime)中会保存一份缓存。删除此缓存，会产生新的模块执行和新的导出。

> 只有很少数的情况需要考虑兼容性！

<details>
<summary>英文</summary>

Multiple requires of the same module result in only one module execution and only one export. Therefore a cache in the runtime exists. Removing values from this cache causes new module execution and a new export.

> This is only needed in rare cases for compatibility!

</details>

```js
var d1 = require('dependency');
require('dependency') === d1;
delete require.cache[require.resolve('dependency')];
require('dependency') !== d1;
```

```js
// in file.js
require.cache[module.id] === module;
require('./file.js') === module.exports;
delete require.cache[module.id];
require.cache[module.id] === undefined;
// （这是理论上的操作不相等；在实际运行中，会导致栈溢出）
require('./file.js') !== module.exports; // in theory; in praxis this causes a stack overflow
require.cache[module.id] !== module;
```

### `require.ensure`

> `require.ensure()` 是 webpack 特有的，现在已经被 `import()` 取代。

<details>
<summary>英文</summary>

> `require.ensure()` is specific to webpack and superseded by `import()`.

</details>

```ts
require.ensure(dependencies: String[], callback: function(require), errorCallback: function(error), chunkName: String)
```

给定 `dependencies` 参数，将其对应的文件拆分到一个单独的 bundle 中，此 bundle 会被异步加载。当使用 CommonJS 模块语法时，这是动态加载依赖的唯一方法。意味着，可以在模块执行时才运行代码，只有在满足某些条件时才加载依赖项。

> 这个特性依赖于内置的 `Promise`。如果想在低版本浏览器使用 `require.ensure`，记得使用像 `es6-promise` 或者 `promise-polyfill` 这样 polyfill 库，来预先填充(shim) Promise 环境。

<details>
<summary>英文</summary>

Split out the given `dependencies` to a separate bundle that will be loaded asynchronously. When using CommonJS module syntax, this is the only way to dynamically load dependencies. Meaning, this code can be run within execution, only loading the `dependencies` if certain conditions are met.

> This feature relies on `Promise` internally. If you use `require.ensure` with older browsers, remember to shim `Promise` using a polyfill such as [es6-promise]() or [promise-polyfill]().

</details>

```js
var a = require('normal-dep');

if (module.hot) {
  require.ensure(['b'], function (require) {
    var c = require('c');

    // Do something special...
  });
}
```

按照上面指定的顺序，webpack 支持以下参数：

- `dependencies`：字符串构成的数组，声明 callback 回调函数中所需的所有模块。
- `callback`：只要加载好全部依赖，webpack 就会执行此函数。`require` 函数的实现，作为参数传入此函数。当程序运行需要依赖时，可以使用 `require()` 来加载依赖。函数体可以使用此参数，来进一步执行 `require()` 模块。(意思就是把 require 当做参数传入，可以在 callback 中继续 require)
- `errorCallback`：当 webpack 加载依赖失败时，会执行此函数。
- `chunkName`：由 `require.ensure()` 创建出的 chunk 的名字。通过将同一个 `chunkName` 传递给不同的 `require.ensure()` 调用，我们可以将它们的代码合并到一个单独的 chunk 中，从而只产生一个浏览器必须加载的 bundle。

> 虽然我们将 `require` 的实现，作为参数传递给回调函数，然而如果使用随意的名字，例如 `require.ensure([], function(request) { request('someModule'); })` 则无法被 webpack 静态解析器处理，所以还是请使用 `require`，例如 `require.ensure([], function(require) { require('someModule'); })`。

<details>
<summary>英文</summary>

The following parameters are supported in the order specified above:

- `dependencies`: An array of strings declaring all modules required for the code in the `callback` to execute.
- `callback`: A function that webpack will execute once the dependencies are loaded. An implementation of the `require` function is sent as a parameter to this function. The function body can use this to further `require()` modules it needs for execution.
- `errorCallback`: A function that is executed when webpack fails to load the dependencies.
- `chunkName`: A name given to the chunk created by this particular `require.ensure()`. By passing the same `chunkName` to various `require.ensure()` calls, we can combine their code into a single chunk, resulting in only one bundle that the browser must load.

> Although the implementation of `require` is passed as an argument to the `callback` function, using an arbitrary name e.g. `require.ensure([], function(request) { request('someModule'); })` isn't handled by webpack's static parser. Use `require` instead, e.g. `require.ensure([], function(require) { require('someModule'); })`.

</details>

## AMD

AMD(Asynchronous Module Definition) 是一种定义了写入模块接口和加载模块接口的 JavaScript 规范。webpack 支持以下的 AMD 方法：

<details>
<summary>英文</summary>

Asynchronous Module Definition (AMD) is a JavaScript specification that defines an interface for writing and loading modules. The following AMD methods are supported by webpack:

</details>

### `define`(with factory) /define（通过 factory 方法导出）

```ts
define([name: String], [dependencies: String[]], factoryMethod: function(...))
```

如果提供 `dependencies` 参数，将会调用 `factoryMethod` 方法，并（以相同的顺序）传入每个依赖项的导出。如果未提供 `dependencies` 参数，则调用 `factoryMethod` 方法时传入 require, exports 和 module（用于兼容）。如果此方法返回一个值，则返回值会作为此模块的导出。由编译器(compiler)来确保依赖项在最终输出 bundle 中可用。

> 注意：webpack 会忽略 `name` 参数。

<details>
<summary>英文</summary>

If `dependencies` are provided, `factoryMethod` will be called with the exports of each dependency (in the same order). If `dependencies` are not provided, `factoryMethod` is called with `require`, `exports` and `module` (for compatibility!). If this function returns a value, this value is exported by the module. The compiler ensures that each dependency is available.

> Note that webpack ignores the name argument.

</details>

```js
define(['jquery', 'my-module'], function ($, myModule) {
  // Do something with $ and myModule...
  // 使用 $ 和 myModule 做一些操作……

  // Export a function
  // 导出一个函数
  return function doSomething() {
    // ...
  };
});
```

此 define 导出方式不能在异步函数中调用。

<details>
<summary>英文</summary>

> This CANNOT be used in an asynchronous function.

</details>

### `define`(with value) / `define`（通过 value 导出）

```ts
define(value: !Function)
```

只会将提供的 `value` 导出。这里的 `value` 可以是除函数外的任何值。

<details>
<summary>英文</summary>

> This will simply export the provided `value`. The `value` here can be anything except a function.

</details>

```js
define({
  answer: 42,
});
```

此 define 导出方式不能在异步函数中调用。

<details>
<summary>英文</summary>

> This CANNOT be used in an async function.

</details>

### `require`(amd-version) / `require`（AMD 版本）

```ts
require(dependencies: String[], [callback: function(...)])
```

与 `require.ensure` 类似，给定 `dependencies` 参数，将其对应的文件拆分到一个单独的 bundle 中，此 bundle 会被异步加载。然后会调用 `callback` 回调函数，并传入 `dependencies` 数组中每一项的导出。

> 这个特性依赖于内置的 `Promise`。如果想在低版本浏览器使用 `require.ensure`，记得使用像 `es6-promise` 或者 `promise-polyfill` 这样 polyfill 库，来预先填充(shim) Promise 环境。

<details>
<summary>英文</summary>

Similar to `require.ensure`, this will split the given `dependencies` into a separate bundle that will be loaded asynchronously. The `callback` will be called with the exports of each dependency in the `dependencies` array.

> This feature relies on [Promise]() internally. If you use AMD with older browsers (e.g. Internet Explorer 11), remember to shim `Promise` using a polyfill such as [es6-promise](https://github.com/stefanpenner/es6-promise) or [promise-polyfill](https://github.com/taylorhakes/promise-polyfill).

</details>

```js
require(['b'], function (b) {
  var c = require('c');
});
```

这里没有提供命名 chunk 名称的选项。

<details>
<summary>英文</summary>

> There is no option to provide a chunk name.

</details>

## Labeled Modules 标签模块

webpack 内置的 `LabeledModulesPlugin` 插件，允许使用下面的方法导出和导入模块：

<details>
<summary>英文</summary>

The internal `LabeledModulesPlugin` enables you to use the following methods for exporting and requiring within your modules:

</details>

### `export` label / export 标签

导出给定的 `value`。export 标记可以出现在函数声明或变量声明之前。函数名或变量名是导出值的标识符。

<details>
<summary>英文</summary>

Export the given `value`. The label can occur before a function declaration or a variable declaration. The function name or variable name is the identifier under which the value is exported.

</details>

```js
export: var answer = 42;
export: function method(value) {
  // Do something...
};
```

以异步的方式使用，可能不会达到预期的效果。

<details>
<summary>英文</summary>

> Using it in an async function may not have the expected effect.

</details>

### `require` label / `require`标签

使当前作用域下，可访问所依赖模块的所有导出。require 标签可以放置在一个字符串之前。依赖模块必须使用 export 标签导出值。CommonJS 或 AMD 模块无法通过这种方式，使用标签模块的导出。

<details>
<summary>英文</summary>

Make all exports from the dependency available in the current scope. The `require` label can occur before a string. The dependency must export values with the `export` label. CommonJS or AMD modules cannot be consumed.

</details>

some-dependency.js

```js
export: var answer = 42;
export: function method(value) {
  // Do something...
};
```

在另一个文件使用 require 标签

```js
require: 'some-dependency';
console.log(answer);
method(...);
```

## Webpack

webpack 除了支持上述的语法之外，还可以使用一些 webpack 特定的方法：

<details>
<summary>英文</summary>

Aside from the module syntaxes described above, webpack also allows a few custom, webpack-specific methods:

</details>

### `require.context`

```js
require.context(
  (directory: String),
  (includeSubdirs: Boolean) /* optional, default true( 可选的，默认值是 true) */,
  (filter: RegExp) /* optional, default /^\.\/.*$/, any file */,
  (mode: String) /* optional, 'sync' | 'eager' | 'weak' | 'lazy' | 'lazy-once', default 'sync' */
);
```

使用 `directory` 路径、`includeSubdirs` 选项和 `filter` 来指定一系列完整的依赖关系，便于更细粒度的控制模块引入;以及使用 `mode`选项 指定加载的方式。后面可以很容易地进行解析：

<details>
<summary>英文</summary>

Specify a whole group of dependencies using a path to the `directory`, an option to `includeSubdirs`, a `filter` for more fine grained control of the modules included, and a `mode` to define the way how loading will work. Underlying modules can then be easily resolved later on:

</details>

```js
var context = require.context('components', true, /\.html$/);
var componentA = context.resolve('componentA');
```

如果将 mode 指定为“ lazy”，则将异步加载基础模块：

<details>
<summary>英文</summary>

If `mode` is specified as "lazy", the underlying modules will be loaded asynchronously:

</details>

```js
var context = require.context('locales', true, /\.json$/, 'lazy');
context('localeA').then((locale) => {
  // do something with locale
});
```

可用模式及其行为的完整列表在 [`import()`](https://github.com/lazyken/webpack4_zh/blob/main/API/Module%20Methods%E6%A8%A1%E5%9D%97%E6%96%B9%E6%B3%95.md/#import-1) 文档中进行了描述。

<details>
<summary>英文</summary>

The full list of available modes and its behavior is described in [import()](https://github.com/lazyken/webpack4_zh/blob/main/API/Module%20Methods%E6%A8%A1%E5%9D%97%E6%96%B9%E6%B3%95.md/#import-1) documentation.

</details>

### `require.include`

```ts
require.include(dependency: String)
```

引入一个依赖但不执行它，这可以用于优化依赖模块在输出 chunk 中的位置。

<details>
<summary>英文</summary>

Include a `dependency` without executing it. This can be used for optimizing the position of a module in the output chunks.

</details>

```js
require.include('a');
require.ensure(['a', 'b'], function (require) {
  /* ... */
});
require.ensure(['a', 'c'], function (require) {
  /* ... */
});-
```

这会产生以下输出:

- entry chunk: `file.js` and `a`
- anonymous chunk: `b`
- anonymous chunk: `c`

如果不使用 `require.include('a')`，输出的两个匿名 chunk (anonymous chunk)都有模块 a。

<details>
<summary>英文</summary>

This will result in the following output:

- entry chunk: `file.js` and `a`
- anonymous chunk: `b`
- anonymous chunk: `c`

Without `require.include('a')` it would be duplicated in both anonymous chunks.

</details>

### `require.resolveWeak`

与 `require.resolve` 类似，但是这不会将 module 引入到 bundle 中。这就是所谓的"弱(weak)"依赖。

<details>
<summary>英文</summary>

Similar to `require.resolve`, but this won't pull the `module` into the bundle. It's what is considered a "weak" dependency.

</details>

```js
if (__webpack_modules__[require.resolveWeak('module')]) {
  // Do something when module is available...
}
if (require.cache[require.resolveWeak('module')]) {
  // Do something when module was loaded before...
}

// You can perform dynamic resolves ("context")
// just as with other require/import methods.
// 你可以像执行其他 require/import 方法一样，执行动态解析（“上下文”）。
const page = 'Foo';
__webpack_modules__[require.resolveWeak(`./page/${page}`)];
```

> `require.resolveWeak` 是通用渲染（SSR + 代码分离）的基础，用于诸如 [`react-universal-component`](https://github.com/faceyspacey/react-universal-component) 之类的包中。它允许代码在「服务器端」和「客户端初始页面的加载上」同步渲染。它要求手动或以某种方式提供 chunk。它可以「在不需要指示模块应该被打包的情况下」引入模块。它与 `import()` 一起使用，当用户导航触发额外的导入时，它会被接管。

<details>
<summary>英文</summary>

> `require.resolveWeak` is the foundation of universal rendering (SSR + Code Splitting), as used in packages such as [react-universal-component](https://github.com/faceyspacey/react-universal-component). It allows code to render synchronously on both the server and initial page-loads on the client. It requires that chunks are manually served or somehow available. It's able to require modules without indicating they should be bundled into a chunk. It's used in conjunction with `import()` which takes over when user navigation triggers additional imports.

</details>

## Further Reading

- [CommonJS Wikipedia](https://en.wikipedia.org/wiki/CommonJS)
- [Asynchronous Module Definition](https://en.wikipedia.org/wiki/Asynchronous_module_definition)
