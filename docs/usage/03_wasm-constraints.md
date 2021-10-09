# Pyodide Python 兼容性

## Python 标准库

除了下面几节中列出的模块外，大多数 Python 标准库都是功能性的。除了 [`src/tests/python_tests.txt`](https://github.com/pyodide/pyodide/blob/main/src/tests/python_tests.txt) 中跳过的测试或通过 [patches](https://github.com/pyodide/pyodide/tree/main/cpython/patches) 跳过的测试外，大部分 CPython 测试套件都通过了。

### 可选模块

以下 stdlib 模块是默认包含的，但是它们可以通过 `loadPyodide({..., fullStdLib = false })` 排除。然后可以根据需要使用 {any}`pyodide.loadPackage` 加载各个模块，

- `distutils`
- `test`：它是上述的一个例外，因为它在默认情况下被排除在外。

### 被移除的模块

以下模块已从标准库中删除，以减少下载大小，由于它们目前无法在 WebAssembly VM 中工作

- curses
- dbm
- ensurepip
- idlelib
- lib2to3
- tkinter
- turtle.py
- turtledemo
- venv

### 包含但不工作的模块

以下模块可以导入，但由于 WebAssembly VM 的限制，不能正常使用：

- multiprocessing
- threading
- sockets

以及任何需要这些的功能。