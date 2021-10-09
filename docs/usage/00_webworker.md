(using_from_webworker)=
# [在 web worker 中使用 Pyodide](https://pyodide.org/en/stable/usage/webworker.html)

本文档描述如何使用 Pyodide 在 web worker 中异步执行 Python 脚本。

## 设置

设置你的项目来服务 `webworker.js`。你也应该提供 `pyodide.js`，以及它所有相关的 `.asm.js`，`.data`，`.json` 和 `.wasm` 文件，尽管如果 `pyodide.js` 指向的站点提供这些文件的当前版本，这不是严格要求的。提供所需文件的最简单方法是使用CDN，例如 `https://cdn.jsdelivr.net/pyodide`。这就是这里提出的解决方案。

更新 `webworker.js` 示例，使其具有 `pyodide.js` 的有效 URL，并将 {any}`indexURL <globalThis.loadPyodide>` 设置为支持文件的位置。

在你的应用程序代码中创建一个 `new Worker(...)`，并使用它的 `.onerror`（[worker onerror]）和 `.onmessage`（[worker onmessage]）方法（listener）附加侦听器。
从工作线程到主线程的通信是通过worker . postmessage()方法完成的(反之亦然)。

Update the `webworker.js` sample so that it has as valid URL for `pyodide.js`, and sets
{any}`indexURL <globalThis.loadPyodide>` to the location of the supporting files.

[worker onmessage]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers#Sending_messages_to_and_from_a_dedicated_worker
[worker onerror]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers#Handling_errors

## 详细例子

在这个示例过程中，我们将涉及三个方面：

- **web worker** 负责在它自己的单独线程中运行脚本。
- **worker API** 公开 consumer-to-provider 的通信接口。
- **consumer** 希望在主线程之外运行一些脚本，这样它们就不会阻塞主线程。

In this example process we will have three parties involved:

### Consumers

我们的目标是在另一个线程中运行一些 Python 代码，这个线程将不能访问主线程对象。因此，我们需要一个 API，它不仅接受我们想要运行的 Python `script` 作为输入，还接受它所依赖的 `context`（如果在主线程中运行 Python 脚本，我们通常可以访问的一些 Javascript 变量）。让我们首先描述我们想要的 API。

下面是一个消费者将通过 worker interface/API `py-worker.js` 与 web worker 进行交换的例子。它使用提供的 `context`  和一个名为 `asyncRun()` 的函数运行以下 Python `script`。

```js
import { asyncRun } from "./py-worker";

const script = `
    import statistics
    from js import A_rank
    statistics.stdev(A_rank)
`;

const context = {
  A_rank: [0.8, 0.4, 1.2, 3.7, 2.6, 5.8],
};

async function main() {
  try {
    const { results, error } = await asyncRun(script, context);
    if (results) {
      console.log("pyodideWorker return results: ", results);
    } else if (error) {
      console.log("pyodideWorker error: ", error);
    }
  } catch (e) {
    console.log(
      `Error in pyodideWorker at ${e.filename}, Line: ${e.lineno}, ${e.message}`
    );
  }
}

main();
```

在编写 API 之前，让我们先看看 worker 是如何操作的。我们的 web worker 如何使用给定的 `context` 运行 `script`。

### Web worker

让我们从定义开始。一个 [worker][worker api] 是：

> worker 是一个使用构造函数（例如 [Worker()][worker constructor] 创建的对象，该构造函数运行一个指定的 JavaScript 文件——这个文件包含在 worker 线程中运行的代码；worker 运行在与当前窗口不同的另一个全局上下文中。这个上下文可以由一个 DedicatedWorkerGlobalScope 对象（在 dedicated workers 的情况下——被单个脚本使用的工作者）或一个 SharedWorkerGlobalScope（在 shared workers 的情况下——在多个脚本之间共享的 workers）表示。

在我们的例子中，我们将使用单个 worker 来执行 Python 代码，而不会干扰客户端渲染（由主 JavaScript 线程完成）。worker 做两件事：

1. 监听来自主线程的新消息
2. 在它执行完 Python 脚本后进行响应

这些都是它应该完成的任务，但它还可以做其他事情。例如，要始终加载包 `numpy` 和 `pytz`，你需要插入一行 {any}`await pyodide.loadPackage(['numpy', 'pytz']); <pyodide.loadPackage>` 如下所示：

```js
// webworker.js

// Setup your project to serve `py-worker.js`. You should also serve
// `pyodide.js`, and all its associated `.asm.js`, `.data`, `.json`,
// and `.wasm` files as well:
importScripts("https://cdn.jsdelivr.net/pyodide/v0.18.1/full/pyodide.js");

async function loadPyodideAndPackages() {
  self.pyodide = await loadPyodide({
    indexURL: "https://cdn.jsdelivr.net/pyodide/v0.18.1/full/",
  });
  await self.pyodide.loadPackage(["numpy", "pytz"]);
}
let pyodideReadyPromise = loadPyodideAndPackages();

self.onmessage = async (event) => {
  // make sure loading is done
  await pyodideReadyPromise;
  // Don't bother yet with this line, suppose our API is built in such a way:
  const { python, ...context } = event.data;
  // The worker copies the context in its own "memory" (an object mapping name to values)
  for (const key of Object.keys(context)) {
    self[key] = context[key];
  }
  // Now is the easy part, the one that is similar to working in the main thread:
  try {
    await self.pyodide.loadPackagesFromImports(python);
    let results = await self.pyodide.runPythonAsync(python);
    self.postMessage({ results });
  } catch (error) {
    self.postMessage({ error: error.message });
  }
};
```

### The worker API

现在我们已经确定了双方需要什么以及他们如何操作，让我们使用这个简单的 API（`py-worker.js`）将它们连接起来。这部分是可选的，只是一个设计选择，你可以通过在主线程和 webworker 之间直接交换消息来实现类似的结果。您只需要像这个 API 那样使用正确的参数调用 `.postMessages()`。

```js
const pyodideWorker = new Worker("./build/webworker.js");

export function run(script, context, onSuccess, onError) {
  pyodideWorker.onerror = onError;
  pyodideWorker.onmessage = (e) => onSuccess(e.data);
  pyodideWorker.postMessage({
    ...context,
    python: script,
  });
}

// Transform the run (callback) form to a more modern async form.
// This is what allows to write:
//    const {results, error} = await asyncRun(script, context);
// Instead of:
//    run(script, context, successCallback, errorCallback);
export function asyncRun(script, context) {
  return new Promise(function (onSuccess, onError) {
    run(script, context, onSuccess, onError);
  });
}
```

[worker api]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API
[worker constructor]: https://developer.mozilla.org/en-US/docs/Web/API/Worker/Worker

## 警告

使用 web worker 是有利的，因为 Python 代码运行在一个独立于主 UI 的线程中，因此不会影响应用程序的响应性。然而，也有一些限制。目前，Pyodide 不支持在多个 web worker 之间或与主线程共享 Python 解释器和包。因为每个 web worker 都在自己的虚拟机中，所以你也不能在 web worker 和主线程之间共享全局变量。最后，虽然 web worker 是独立于主线程的，但是 web worker 本身是单线程的，所以一次只能执行一个 Python 脚本。
