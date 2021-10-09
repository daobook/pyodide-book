# 常见问题

## 如何在 Pyodide 中加载外部 Python 文件？

‎两种可能的解决方案是，‎

- 将这些文件包含在一个 Python 包中，用 `python setup.py bdist_wheel` 建立一个纯 Python 轮子，然后用 {ref}`micropip 加载<micropip-installing-from-arbitrary-urls>`。
- 将 Python 代码作为一个字符串获取，并在 Python 中进行评估。
  ```js
  pyodide.runPython(await (await fetch("https://some_url/...")).text());
  ```

在这两种情况下，文件需要用网络服务器来提供，不能从本地文件系统加载。

## 为什么我不能从本地文件系统加载文件？

由于安全原因，浏览器中的 JavaScript 不允许加载本地数据文件。你需要用网络浏览器来提供它们。在 Chrome 中支持[文件系统 API](https://wicg.github.io/file-system-access/)，但在 Firefox 或 Safari 中不支持。

## 我怎样才能改变 {any}`runPython <pyodide.runPython>` 和 {any}`runPythonAsync <pyodide.runPythonAsync>` 的行为？

你可以直接从 JavaScript 中调用 Python 函数。在许多情况下，把你自己的 Python 函数作为一个入口，并调用它而不是使用 `runPython` 是有意义的。{any}`runPython <pyodide.runPython>` 和 {any}`runPythonAsync <pyodide.runPythonAsync>` 的定义非常简单。

```javascript
function runPython(code) {
  pyodide.pyodide_py.eval_code(code, pyodide.globals);
}
```

```javascript
async function runPythonAsync(code) {
  let coroutine = pyodide.pyodide_py.eval_code_async(code, pyodide.globals);
  try {
    let result = await coroutine;
    return result;
  } finally {
    coroutine.destroy();
  }
}
```

要制作你自己的 {any}`runPython <pyodide.runPython>` 版本，你可以这样做：

```pyodide
pyodide.runPython(`
  import pyodide
  def my_eval_code(code, ns):
    extra_info = None
    result = pyodide.eval_code(code, ns)
    return ns["extra_info"], result]
`)

function myRunPython(code){
  return pyodide.globals.get("my_eval_code")(code, pyodide.globals);
}
```

然后 `pyodide.myRunPython("2+7")` 返回 `[None, 9]` 而 `pyodide.myRunPython("extra_info='hello' ; 2 + 2")` 返回 `['hello', 4]`。如果你想改变 {any}`pyodide.loadPackagesFromImports` 所加载的包，你可以使用 monkey patch {any}`pyodide.find_imports`，它以 `code` 为参数，返回导入的包的列表。

## 我如何在一个自定义的命名空间中执行代码？

{any}`pyodide.eval_code` 的第二个参数是一个执行代码的全局命名空间。这个命名空间是一个 Python 字典。

```javascript
let my_namespace = pyodide.globals.dict();
pyodide.runPython(`x = 1 + 1`, my_namespace);
pyodide.runPython(`y = x ** x`, my_namespace);
my_namespace.y; // ==> 4
```

## 如何检测代码是否在 Pyodide 中运行？

在运行时，你可以通过以下方式检测代码是否在 Pyodide 中运行。

```py
import sys

if "pyodide" in sys.modules:
   # running in Pyodide
```

更为普遍的是，你可以用以下方法检测用 Emscripten（包括 Pyodide）构建的 Python，

```py
import platform

if platform.system() == 'Emscripten':
    # running in Pyodide or other Emscripten based build
```

然而，由于 Pyodide 构建系统的工作方式，这在构建时（即在 `setup.py` 中）是不工作的。它首先用主机编译器（如 gcc）编译包，然后用 emsdk 重新运行编译命令。所以 `setup.py` 不会在 Pyodide 环境中运行。

要检测 Pyodide，在构建时使用，

```python
import os

if "PYODIDE" in os.environ:
    # building for Pyodide
```

我们曾经为此目的使用环境变量 `PYODIDE_BASE_URL`，但这种用法已被废弃。

## 如何从 Javascript 中创建自定义 Python 包？

将一个函数集合放入一个 JavaScript 对象中，并使用 {any}`pyodide.registerJsModule` Javascript：

```javascript
let my_module = {
  f: function (x) {
    return x * x + 1;
  },
  g: function (x) {
    console.log(`Calling g on argument ${x}`);
    return x;
  },
  submodule: {
    h: function (x) {
      return x * x - 1;
    },
    c: 2,
  },
};
pyodide.registerJsModule("my_js_module", my_module);
```

你可以像普通的 Python 包一样导入你的包：

```py
import my_js_module
from my_js_module.submodule import h, c
assert my_js_module.f(7) == 50
assert h(9) == 80
assert c == 2
```

## 我怎样才能从我的服务器发送一个 Python 对象到 Pyodide？

最好的方法是用 pickle 来做到这一点。如果服务器中使用的 Python 版本与客户端中使用的 Python 版本完全匹配，那么可以成功 pickle 的对象可以被发送到客户端并在 Pyodide 中解除 pickle。如果 Python 的版本不同，那么例如发送 AST 是不可能成功的，因为在大多数 Python 的次要版本中，Python 的 AST 有破坏性的变化。

同样，当对定义在 Python 包中的 Python 对象进行腌制（pickling）时，包的版本需要在服务器和 pyodide 之间完全匹配。

一般来说，pickles 在不同的架构之间是可移植的（这里是 amd64 和 wasm32）。极少数情况下，它们是不可移植的，例如目前 scikit-learn 中基于树的模型，可以被认为是上游库的错误。 

```{admonition} pickle 的安全问题
:class: warning

Unpickling 数据类似于 `eval`。在任何面向公众的服务器上，解开从客户端发送的任何数据是一个非常糟糕的主意。对于从客户端向服务器发送数据，请尝试其他的序列化格式，如 JSON。
```

## 我怎样才能把一个 Python 函数作为事件处理程序，然后再把它删除？

请注意，最直接的方法是不可行的：

```py
from js import document
def f(*args):
    document.querySelector("h1").innerHTML += "(>.<)"

document.body.addEventListener('click', f)
document.body.removeEventListener('click', f)
```

这泄漏了 `f`，并且没有删除事件监听器（相反，`removeEventListener` 将默默地什么都不做）。

为了正确地做到这一点，请使用 :func:`pyodide.create_proxy`，如下所示：

```py
from js import document
from pyodide import create_proxy
def f(*args):
    document.querySelector("h1").innerHTML += "(>.<)"

proxy_f = create_proxy(f)
document.body.addEventListener('click', proxy_f)
# Store proxy_f in Python then later:
document.body.removeEventListener('click', proxy_f)
proxy_f.destroy()
```

这也避免了内存泄漏。

## 我怎样才能从 Pytho n中使用带可选参数的 fetch 呢？

最明显的 Javascript 代码的翻译是行不通的。

```py
import json
resp = await js.fetch('/someurl', {
  "method": "POST",
  "body": json.dumps({ "some" : "json" }),
  "credentials": "same-origin",
  "headers": { "Content-Type": "application/json" }
})
```

这就泄露了字典，而且 `fetch` api 忽略了我们试图提供的选项。你可以正确地这样做，如下：

```py
import json
from pyodide import to_js
from js import Object
resp = await js.fetch('example.com/some_api',
  method= "POST",
  body= json.dumps({ "some" : "json" }),
  credentials= "same-origin",
  headers= Object.fromEntries(to_js({ "Content-Type": "application/json" })),
)
```

## 我怎样才能控制 stdin / stdout / stderr 的行为？

这与在本地 Python 中的工作原理基本相同：你可以分别覆盖 `sys.stdin`、`sys.stdout` 和 `sys.stderr`。如果你想暂时这样做，建议使用 [`contextlib.redirect_stdout`](https://docs.python.org/3/library/contextlib.html#contextlib.redirect_stdout) 和 [`contextlib.redirect_stderr`](https://docs.python.org/3/library/contextlib.html#contextlib.redirect_stderr)。没有 `contextlib.redirect_stdin`，但可以很容易地制作你自己的，如下：

```py
from contextlib import _RedirectStream
class redirect_stdin(_RedirectStream):
    _stream = "stdin"
```

例如，如果你这样做：

```
from io import StringIO
with redirect_stdin(StringIO("\n".join(["eval", "asyncio.ensure_future", "functools.reduce", "quit"]))):
  help()
```

它将打印：

```
Welcome to Python 3.9's help utility!
<...OMITTED LINES>
Help on built-in function eval in module builtins:
eval(source, globals=None, locals=None, /)
    Evaluate the given source in the context of globals and locals.
<...OMITTED LINES>
Help on function ensure_future in asyncio:
asyncio.ensure_future = ensure_future(coro_or_future, *, loop=None)
    Wrap a coroutine or an awaitable in a future.
<...OMITTED LINES>
Help on built-in function reduce in functools:
functools.reduce = reduce(...)
    reduce(function, sequence[, initial]) -> value
    Apply a function of two arguments cumulatively to the items of a sequence,
<...OMITTED LINES>
You are now leaving help and returning to the Python interpreter.
```