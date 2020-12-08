> https://v4.webpack.js.org/guides/caching/

# Caching 缓存

> 本指南继续沿用[起步]()、[管理输出]()和[代码分离]()中的代码示例。

以上，我们使用 webpack 来打包我们的模块化后的应用程序，webpack 会生成一个可部署的 `/dist` 目录，然后把打包后的内容放置在此目录中。只要 `/dist` 目录中的内容部署到服务器上，客户端（通常是浏览器）就能够访问此服务器的网站及其资源。而最后一步获取资源是比较耗费时间的，这就是为什么浏览器使用一种名为 [缓存]() 的技术。可以通过命中缓存，以降低网络流量，使网站加载速度更快。然而，(如果我们在部署新版本时不更改资源的文件名，浏览器可能会认为它没有被更新，就会使用它的缓存版本。)由于缓存的存在，当你需要获取新的代码时，就会显得很棘手。

此指南的重点在于通过必要的配置，以确保 webpack 编译生成的文件能够被客户端缓存，而在文件内容变化后，能够请求到新的文件。

<details>
<summary>英文</summary>

> The examples in this guide stem from getting started, output management and code splitting.

So we're using webpack to bundle our modular application which yields a deployable `/dist` directory. Once the contents of `/dist` have been deployed to a server, clients (typically browsers) will hit that server to grab the site and its assets. The last step can be time consuming, which is why browsers use a technique called [caching](). This allows sites to load faster with less unnecessary network traffic. However, it can also cause headaches when you need new code to be picked up.

This guide focuses on the configuration needed to ensure files produced by webpack compilation can remain cached unless their content has changed.

</details>

## Output Filenames 输出文件的文件名

我们可以使用 `output.filename` 替换设置输出文件的文件名(可以确保浏览器获取到修改后的文件)。webpack 提供了一种使用方括号括起来的字符串来替代文件名的模板方法，称之为**substitutions**。[contenthash]替换将基于资源的内容添加唯一的 hash。 当资源的内容更改时，[contenthash]也将更改。  
让我们使用 `起步` 中的示例，以及 `管理输出` 中的 `plugins` 来作为项目的基础，所以我们不必手动处理维护 `index.html` 文件：

<details>
<summary>英文</summary>

We can use the `output.filename` substitutions setting to define the names of our output files. webpack provides a method of templating the filenames using bracketed strings called **substitutions**. The `[contenthash]` substitution will add a unique hash based on the content of an asset. When the asset's content changes, `[contenthash]` will change as well.  
Let's get our project set up using the example from `getting started` with the `plugins` from `output management`, so we don't have to deal with maintaining our `index.html` file manually:

</details>

project

```text
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
|- /node_modules
```

webpack.config.js

```js
  const path = require('path');
  const { CleanWebpackPlugin } = require('clean-webpack-plugin');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
-       title: 'Output Management',
+       title: 'Caching',
      }),
    ],
    output: {
-     filename: 'bundle.js',
+     filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist'),
    },
  };
```

使用此配置，然后运行我们的构建脚本 `npm run build`，应该产生以下输出：

<details>
<summary>英文</summary>

Running our build script, `npm run build`, with this configuration should produce the following output:

</details>

```text
...
                       Asset       Size  Chunks                    Chunk Names
main.7e2c49a622975ebd9b7e.js     544 kB       0  [emitted]  [big]  main
                  index.html  197 bytes          [emitted]
...
```

可以看到，bundle 的名称是它内容（通过 hash）的映射。如果我们不做修改，然后再次运行构建，我们以为文件名会保持不变。然而，如果我们真的运行，可能会发现情况并非如此：（译注：这里的意思是，如果不做修改，文件名可能会变，也可能不会。）

<details>
<summary>英文</summary>

As you can see the bundle's name now reflects its content (via the hash). If we run another build without making any changes, we'd expect that filename to stay the same. However, if we were to run it again, we may find that this is not the case:

</details>

```text
...
                       Asset       Size  Chunks                    Chunk Names
main.205199ab45963f6a62ec.js     544 kB       0  [emitted]  [big]  main
                  index.html  197 bytes          [emitted]
...
```

这是因为 webpack 在入口 chunk 中，包含了某些样板(boilerplate)，特别是 runtime 和 manifest。（译注：样板(boilerplate)指 webpack 运行时的引导代码）

> 输出可能会因当前的 webpack 版本而稍有差异。新版本不一定有和旧版本相同的 hash 问题，但我们以下推荐的步骤，仍然是可靠的。

<details>
<summary>英文</summary>

This is because webpack includes certain boilerplate, specifically the runtime and manifest, in the entry chunk.

> Output may differ depending on your current webpack version. Newer versions may not have all the same issues with hashing as some older versions, but we still recommend the following steps to be safe.

</details>

## Extracting Boilerplate 提取模板

就像我们之前从[代码分离]()了解到的，[SplitChunksPlugin]() 可以用于将模块分离到单独的文件中。webpack 提供了一个优化特性，可以使用 `optimize.runtimechunk` 选项将运行时代码分割成单独的块。将其设置为 `single` 可以为所有 chunks 创建一个运行时 bundle

<details>
<summary>英文</summary>

As we learned in [code splitting](), the [`SplitChunksPlugin`]() can be used to split modules out into separate bundles. webpack provides an optimization feature to split runtime code into a separate chunk using the [`optimization.runtimeChunk`]() option. Set it to `single` to create a single runtime bundle for all chunks:

</details>

webpack.config.js

```js
  const path = require('path');
  const { CleanWebpackPlugin } = require('clean-webpack-plugin');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Caching',
      }),
    ],
    output: {
      filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist'),
    },
+   optimization: {
+     runtimeChunk: 'single',
+   },
  };
```

让我们再次构建，然后查看提取出来的 `runtime` bundle：

<details>
<summary>英文</summary>

Let's run another build to see the extracted `runtime` bundle:

</details>

```text
Hash: 82c9c385607b2150fab2
Version: webpack 4.12.0
Time: 3027ms
                          Asset       Size  Chunks             Chunk Names
runtime.cc17ae2a94ec771e9221.js   1.42 KiB       0  [emitted]  runtime
   main.e81de2cf758ada72f306.js   69.5 KiB       1  [emitted]  main
                     index.html  275 bytes          [emitted]
[1] (webpack)/buildin/module.js 497 bytes {1} [built]
[2] (webpack)/buildin/global.js 489 bytes {1} [built]
[3] ./src/index.js 309 bytes {1} [built]
    + 1 hidden module
```

将第三方库(library)（例如 `lodash` 或 `react`）提取到单独的 `vendor` chunk 文件中，是比较推荐的做法，这是因为，它们很少像本地的源代码那样频繁修改。以上步骤将允许客户端向服务器发出更少的请求以保持最新。 (因此通过实现以上步骤，利用客户端的长效缓存机制，可以通过命中缓存来消除请求，减少向服务器获取资源，同时还能保证客户端代码和服务器端代码版本一致)。这可以通过使用 `Example 2 of SplitChunksPlugin` 中演示的 `SplitChunksPlugin` 的 `cacheGroups` 选项来完成。 让我们添加`optimization.splitChunks`选项并使用 `cacheGroups` 选项以及它的参数来构建：

<details>
<summary>英文</summary>

It's also good practice to extract third-party libraries, such as `lodash` or `react`, to a separate `vendor` chunk as they are less likely to change than our local source code. This step will allow clients to request even less from the server to stay up to date. This can be done by using the [`cacheGroups`]() option of the [`SplitChunksPlugin`])() demonstrated in [`Example 2 of SplitChunksPlugin`](). Lets add `optimization.splitChunks` with `cacheGroups` with next params and build:

</details>

webpack.config.js

```js
  const path = require('path');
  const { CleanWebpackPlugin } = require('clean-webpack-plugin');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Caching',
      }),
    ],
    output: {
      filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist'),
    },
    optimization: {
      runtimeChunk: 'single',
+     splitChunks: {
+       cacheGroups: {
+         vendor: {
+           test: /[\\/]node_modules[\\/]/,
+           name: 'vendors',
+           chunks: 'all',
+         },
+       },
+     },
    },
  };
```

让我们再次构建，然后查看新的 `vendor` bundle：

<details>
<summary>英文</summary>

Let's run another build to see our new `vendor` bundle:

</details>

```text
...
                          Asset       Size  Chunks             Chunk Names
runtime.cc17ae2a94ec771e9221.js   1.42 KiB       0  [emitted]  runtime
vendors.a42c3ca0d742766d7a28.js   69.4 KiB       1  [emitted]  vendors
   main.abf44fedb7d11d4312d7.js  240 bytes       2  [emitted]  main
                     index.html  353 bytes          [emitted]
...
```

我们可以看到，现在 `main` bundle 不包含来自 `node_module` 目录的 `vendor` 代码，而且它缩小到了 `240 bytes`。

<details>
<summary>英文</summary>

We can now see that our `main` bundle does not contain `vendor` code from `node_modules` directory and is down in size to `240 bytes`!

</details>

## Module Identifiers 模块标识符

让我们向项目中再添加一个模块 `print.js`：

<details>
<summary>英文</summary>

Let's add another module, `print.js`, to our project:

</details>

project

```text
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
+ |- print.js
|- /node_modules
```

print.js

```js
+ export default function print(text) {
+   console.log(text);
+ };
```

src/index.js

```js
  import _ from 'lodash';
+ import Print from './print';

  function component() {
    const element = document.createElement('div');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.onclick = Print.bind(null, 'Hello webpack!');

    return element;
  }

  document.body.appendChild(component());
```

再次运行构建，然后我们期望的是，只有 `main` bundle 的 hash 发生变化，然而……

<details>
<summary>英文</summary>

Running another build, we would expect only our `main` bundle's hash to change, however...

</details>

```text
...
                           Asset       Size  Chunks                    Chunk Names
  runtime.1400d5af64fc1b7b3a45.js    5.85 kB      0  [emitted]         runtime
  vendor.a7561fb0e9a071baadb9.js     541 kB       1  [emitted]  [big]  vendor
    main.b746e3eb72875af2caa9.js    1.22 kB       2  [emitted]         main
                      index.html  352 bytes          [emitted]
...
```

……我们可以看到这三个文件的 hash 都变化了。这是因为每个 `module.id` 会基于默认的解析顺序(resolve order)进行增量。也就是说，当解析顺序发生变化，ID 也会随之改变。因此，简要概括：

- `main` bundle 会随着自身的新增内容的修改，而发生变化。
- `vendor` bundle 会随着自身的 `module.id` 的修改，而发生变化。
- `manifest` bundle 会因为当前包含一个新模块的引用，而发生变化。

第一个和最后一个都是符合预期的行为，而 `vendor` 的 hash 发生变化是我们要修复的。让我使用 `optimization.moduleIds` 选项并设置为 `"hashed"`。

<details>
<summary>英文</summary>

...we can see that all three have. This is because each `module.id` is incremented based on resolving order by default. Meaning when the order of resolving is changed, the IDs will be changed as well. So, to recap:

- The `main` bundle changed because of its new content.
- The `vendor` bundle changed because its `module.id` was changed.
- And, the `runtime` bundle changed because it now contains a reference to a new module.

The first and last are expected, it's the `vendor` hash we want to fix. Let's use `optimization.moduleIds` with `'hashed'` option:

</details>

webpack.config.js

```js
  const path = require('path');
  const { CleanWebpackPlugin } = require('clean-webpack-plugin');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Caching',
      }),
    ],
    output: {
      filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist'),
    },
    optimization: {
+     moduleIds: 'hashed',
      runtimeChunk: 'single',
      splitChunks: {
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
          },
        },
      },
    },
  };
```

现在，不管再添加任何新的本地依赖，对于每次构建，`vendor` hash 都应该保持一致:

<details>
<summary>英文</summary>

Now, despite any new local dependencies, our `vendor` hash should stay consistent between builds:

</details>

```text
...
                          Asset       Size  Chunks             Chunk Names
   main.216e852f60c8829c2289.js  340 bytes       0  [emitted]  main
vendors.55e79e5927a639d21a1b.js   69.5 KiB       1  [emitted]  vendors
runtime.725a1a51ede5ae0cfde0.js   1.42 KiB       2  [emitted]  runtime
                     index.html  353 bytes          [emitted]
Entrypoint main = runtime.725a1a51ede5ae0cfde0.js vendors.55e79e5927a639d21a1b.js main.216e852f60c8829c2289.js
...
```

然后，修改我们的 `src/index.js`，临时移除额外的依赖：

<details>
<summary>英文</summary>

And let's modify our `src/index.js` to temporarily remove that extra dependency:

</details>

src/index.js

```js
  import _ from 'lodash';
- import Print from './print';
+ // import Print from './print';

  function component() {
    const element = document.createElement('div');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
-   element.onclick = Print.bind(null, 'Hello webpack!');
+   // element.onclick = Print.bind(null, 'Hello webpack!');

    return element;
  }

  document.body.appendChild(component());
```

最后，再次运行我们的构建:

<details>
<summary>英文</summary>

And finally run our build again:

</details>

```text
...
                          Asset       Size  Chunks             Chunk Names
   main.ad717f2466ce655fff5c.js  274 bytes       0  [emitted]  main
vendors.55e79e5927a639d21a1b.js   69.5 KiB       1  [emitted]  vendors
runtime.725a1a51ede5ae0cfde0.js   1.42 KiB       2  [emitted]  runtime
                     index.html  353 bytes          [emitted]
Entrypoint main = runtime.725a1a51ede5ae0cfde0.js vendors.55e79e5927a639d21a1b.js main.ad717f2466ce655fff5c.js
...
```

我们可以看到，这两次构建中，vendor bundle 的文件名称，都是 `55e79e5927a639d21a1b`。

<details>
<summary>英文</summary>

We can see that both builds yielded `55e79e5927a639d21a1b` in the `vendor` bundle's filename.

</details>

## Conclusion 结论

缓存可能很复杂，但是它为应用或网站用户带来的好处使其值得付出努力。请参阅下面的扩展阅读部分以了解更多信息。

<details>
<summary>英文</summary>

Caching can be complicated, but the benefit to application or site users makes it worth the effort. See the _Further Reading_ section below to learn more.

</details>

## Further Reading 扩展阅读

- [Issue 652](https://github.com/webpack/webpack.js.org/issues/652)
