# Lazy Loading 懒加载/按需加载

> 本指南的继承自[代码分离](https://github.com/lazyken/webpack4_zh/blob/main/Guides%E6%8C%87%E5%8D%97/CodeSplitting%E4%BB%A3%E7%A0%81%E5%88%86%E7%A6%BB.md)。如果你尚未阅读该指南，请先行阅读。

<details>
<summary>英文</summary>

> This guide is a small follow-up to [Code Splitting](https://github.com/lazyken/webpack4_zh/blob/main/Guides%E6%8C%87%E5%8D%97/CodeSplitting%E4%BB%A3%E7%A0%81%E5%88%86%E7%A6%BB.md). If you have not yet read through that guide, please do so now.

</details>

懒加载或者按需加载，是一种很好的优化网页或应用的方式。这种方式实际上是先把你的代码在一些逻辑断点处分离开，然后在一些代码块中完成某些操作后，立即引用或即将引用另外一些新的代码块。这样加快了应用的初始加载速度，减轻了它的总体体积，因为某些代码块可能永远不会被加载。

<details>
<summary>英文</summary>

Lazy, or "on demand", loading is a great way to optimize your site or application. This practice essentially involves splitting your code at logical breakpoints, and then loading it once the user has done something that requires, or will require, a new block of code. This speeds up the initial load of the application and lightens its overall weight as some blocks may never even be loaded.

</details>

## Example 示例

我们在[代码分离](https://github.com/lazyken/webpack4_zh/blob/main/Guides%E6%8C%87%E5%8D%97/CodeSplitting%E4%BB%A3%E7%A0%81%E5%88%86%E7%A6%BB.md)中的例子基础上，进一步做些调整来说明这个概念。那里的代码确实会在脚本运行的时候产生一个分离的代码块 `lodash.bundle.js` ，在技术概念上“懒加载”它。问题是加载这个包并不需要用户的交互 -- 意思是每次加载页面的时候都会请求它。这样做并没有对我们有很多帮助，还会对性能产生负面影响。

我们试试不同的做法。我们增加一个交互，当用户点击按钮的时候用 console 打印一些文字。但是会等到第一次交互的时候再加载那个代码块（`print.js`）。为此，我们返回到代码分离的例子中，把 `lodash` 放到主代码块中，重新运行代码分离中的代码 `final Dynamic Imports example`。

<details>
<summary>英文</summary>

Let's take the example from [Code Splitting](https://github.com/lazyken/webpack4_zh/blob/main/Guides%E6%8C%87%E5%8D%97/CodeSplitting%E4%BB%A3%E7%A0%81%E5%88%86%E7%A6%BB.md) and tweak it a bit to demonstrate this concept even more. The code there does cause a separate chunk, `lodash.bundle.js`, to be generated and technically "lazy-loads" it as soon as the script is run. The trouble is that no user interaction is required to load the bundle -- meaning that every time the page is loaded, the request will fire. This doesn't help us too much and will impact performance negatively.

Let's try something different. We'll add an interaction to log some text to the console when the user clicks a button. However, we'll wait to load that code (`print.js`) until the interaction occurs for the first time. To do this we'll go back and rework the `final Dynamic Imports example` from Code Splitting and leave lodash in the main chunk.

</details>

project

```js
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
+ |- print.js
|- /node_modules
```

src/print.js

```js
console.log('The print.js module has loaded! See the network tab in dev tools...');

export default () => {
  console.log('Button Clicked: Here\'s "some text"!');
};
```

src/index.js

```js
+ import _ from 'lodash';
+
- async function getComponent() {
+ function component() {
    const element = document.createElement('div');
-   const _ = await import(/* webpackChunkName: "lodash" */ 'lodash');
+   const button = document.createElement('button');
+   const br = document.createElement('br');

+   button.innerHTML = 'Click me and look at the console!';
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.appendChild(br);
+   element.appendChild(button);
+
+   // Note that because a network request is involved, some indication
+   // of loading would need to be shown in a production-level site/app.
+   button.onclick = e => import(/* webpackChunkName: "print" */ './print').then(module => {
+     const print = module.default;
+
+     print();
+   });

    return element;
  }

- getComponent().then(component => {
-   document.body.appendChild(component);
- });
+ document.body.appendChild(component());
```

> 注意当调用 ES6 模块的 `import()` 方法（引入模块）时，必须指向模块的 `.default` 值，因为它才是 promise 被处理后返回的实际的 `module` 对象。

现在运行 webpack 来验证一下我们的懒加载功能：

<details>
<summary>英文</summary>

> Note that when using import() on ES6 modules you must reference the .default property as it's the actual module object that will be returned when the promise is resolved.

Now let's run webpack and check out our new lazy-loading functionality:

</details>

```text
...
          Asset       Size  Chunks                    Chunk Names
print.bundle.js  417 bytes       0  [emitted]         print
index.bundle.js     548 kB       1  [emitted]  [big]  index
     index.html  189 bytes          [emitted]
...
```

## Frameworks 框架

许多框架和类库对于如何用它们自己的方式来实现（懒加载）都有自己的建议。这里有一些例子：

<details>
<summary>英文</summary>

Many frameworks and libraries have their own recommendations on how this should be accomplished within their methodologies. Here are a few examples:

</details>

- React: Code Splitting and Lazy Loading
- Vue: Lazy Load in Vue using Webpack's code splitting
- Angular: Lazy Loading route configuration
- AngularJS: AngularJS + Webpack = lazyLoad by @var_bincom

## Further Reading 扩展阅读

[Lazy Loading ES2015 Modules in the Browser](https://dzone.com/articles/lazy-loading-es2015-modules-in-the-browser)
