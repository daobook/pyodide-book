# 使用 Pyodide

Pyodide 可用于您希望在 Web 浏览器或后端 JavaScript 环境中运行 Python 的任何上下文。

## 网络浏览器

要在网页上使用 Pyodide，您需要加载 `pyodide.js` 并使用 {any}`loadPyodide` 指定包的 index URL 来初始化 Pyodide：

```html-pyodide
<!DOCTYPE html>
<html>
  <head>
      <script src="https://cdn.jsdelivr.net/pyodide/v0.18.1/full/pyodide.js"></script>
  </head>
  <body>
    <script type="text/javascript">
      async function main(){
        let pyodide = await loadPyodide({
          indexURL : "https://cdn.jsdelivr.net/pyodide/v0.18.1/full/"
        });
        console.log(pyodide.runPython("1 + 2"));
      }
      main();
    </script>
  </body>
</html>
```

有关现有功能的更深入讨论，请参阅 {ref}`quickstart` 以及 [](loading_packages) 和 [](type-translations)。

您还可以使用 [Pyodide NPM 包](https://www.npmjs.com/package/pyodide) 将 Pyodide 集成到您的应用程序中。

```{note}
为避免混淆，请注意：
 - `cdn.jsdelivr.net/pyodide/` 分发用 Pyodide 和 `pyodide.js` 构建的 Python 包
 - `cdn.jsdelivr.net/npm/pyodide@0.18.1/` 是 Pyodide NPM 包的镜像，其中不包含任何 WASM 文件
```

## Web Workers

默认情况下，WebAssembly 在主浏览器线程中运行，它可以使 UI 对长时间运行的计算无响应。

为了避免这种情况，一种解决方案是 [在 WebWorker 中运行 Pyodide](using_from_webworker)。

## Node.js

从 0.18.0 版本开始，Pyodide 可以在 Node.js 中实验性地运行。

安装 [Pyodide npm 包](https://www.npmjs.com/package/pyodide)，

```
npm install pyodide
```

从 [Github 版本](https://github.com/pyodide/pyodide/releases)（**pyodide-build-\*.tar.bz2** 文件）下载并提取 Pyodide 包。发行版的版本需要与此包的版本完全匹配。

然后你可以在 Node.js 中加载 Pyodide，如下所示，

```js
let pyodide_pkg = await import("pyodide/pyodide.js");

let pyodide = await pyodide_pkg.loadPyodide({
  indexURL: "<pyodide artifacts folder>",
});

await pyodide.runPythonAsync("1+1");
```

```{note}
要启动支持顶级 await 的 Node.js REPL，请使用 `node --experimental-repl-await`。
```

```{warning}
在 Node.js 中运行时，当前不会缓存从 PyPi 下载的包。每次运行 `micropip.install` 时都会重新下载软件包。

出于同样的原因，目前明确不支持从 CDN 安装 Pyodide 包。
```

```{tableofcontents}
```