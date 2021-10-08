(quickstart)=
# 入门

## 在线试用

直接在浏览器中尝试 [REPL](https://pyodide.org/en/latest/console.html) 中的 Pyodide（无需安装）。

## 设置

要将 Pyodide 包含在您的项目中，您可以使用以下 CDN URL：

```text
https://cdn.jsdelivr.net/pyodide/v0.18.1/full/pyodide.js
```

您还可以从 [Github 发行版](https://github.com/pyodide/pyodide/releases) 下载发行版或自己构建 Pyodide。有关更多详细信息，请参阅 [](downloading_deploying)。

`pyodide.js` 文件定义了一个名为 {any}`loadPyodide <globalThis.loadPyodide>` 的单异步函数（single async function），它设置 Python 环境并返回 {js:mod}`Pyodide 顶级命名空间 <pyodide>`。

```pyodide
async function main() {
  let pyodide = await loadPyodide({ indexURL : "https://cdn.jsdelivr.net/pyodide/v0.18.1/full/" });
  // Pyodide is now ready to use...
  console.log(pyodide.runPython(`
    import sys
    sys.version
  `));
};
main();
```

## 运行 Python 代码

Python 代码使用 {any}`pyodide.runPython` 函数运行。它将一串 Python 代码作为输入。如果代码以表达式结尾，则返回表达式的结果，并转换为 Javascript 对象（请参阅 {ref}`type-translations`）。例如，以下代码将 `version` 字符串作为 Javascript 字符串返回：

```pyodide
pyodide.runPython(`
  import sys
  sys.version
`);
```

导入 Pyodide 后，只有标准库中的包可用。有关加载其他包的信息，请参阅 {ref}`loading_packages`。

## 完整示例

创建并保存一个包含以下内容的测试 `index.html` 页面：

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
      <script src="https://cdn.jsdelivr.net/pyodide/v0.18.1/full/pyodide.js"></script>
  </head>
  <body>
    Pyodide 测试页 <br>
    打开浏览器控制台查看 Pyodide 输出
    <script type="text/javascript">
      async function main(){
        let pyodide = await loadPyodide({
          indexURL : "https://cdn.jsdelivr.net/pyodide/v0.18.1/full/"
        });
        console.log(pyodide.runPython(`
            import sys
            sys.version
        `));
        console.log(pyodide.runPython("print(1 + 2)"));
      }
      main();
    </script>
  </body>
</html>
```

## 可选示例

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <script src="https://cdn.jsdelivr.net/pyodide/v0.18.1/full/pyodide.js"></script>
  </head>

  <body>
    <p>
      您可以执行任何 Python 代码。只需在下面的框中输入内容，然后单击按钮。
    </p>
    <input id="code" value="sum([1, 2, 3, 4, 5])" />
    <button onclick="evaluatePython()">运行</button>
    <br />
    <br />
    <div>输出：</div>
    <textarea id="output" style="width: 100%;" rows="6" disabled></textarea>

    <script>
      const output = document.getElementById("output");
      const code = document.getElementById("code");

      function addToOutput(s) {
        output.value += ">>>" + code.value + "\n" + s + "\n";
      }

      output.value = "正在初始化...\n";
      // init Pyodide
      async function main() {
        let pyodide = await loadPyodide({
          indexURL: "https://cdn.jsdelivr.net/pyodide/v0.18.1/full/",
        });
        output.value += "准备好！\n";
        return pyodide;
      }
      let pyodideReadyPromise = main();

      async function evaluatePython() {
        let pyodide = await pyodideReadyPromise;
        try {
          let output = pyodide.runPython(code.value);
          addToOutput(output);
        } catch (err) {
          addToOutput(err);
        }
      }
    </script>
  </body>
</html>
```

## 从 Javascript 访问 Python 域

您还可以使用 {any}`pyodide.globals` 对象从 Javascript 访问 Python 中定义的所有函数和变量。

例如，如果您在 Python 中运行代码 `x = numpy.ones([3,3])`，您可以在浏览器的开发人员控制台中从 Javascript 以 `pyodide.globals.get("x")` 的形式访问变量 `x`。函数和导入也是如此。有关更多详细信息，请参阅 {ref}`type-translations`。

您可以在浏览器控制台中自行尝试：

```pyodide
pyodide.runPython(`
  import numpy
  x=numpy.ones((3, 4))
`);
pyodide.globals.get('x').toJs();
// >>> [ Float64Array(4), Float64Array(4), Float64Array(4) ]

// create the same 3x4 ndarray from js
x = pyodide.globals.get('numpy').ones(new Int32Array([3, 4])).toJs();
// x >>> [ Float64Array(4), Float64Array(4), Float64Array(4) ]
```

由于您可以完全访问 Python 全局域，您还可以将新值甚至 Javascript 函数重新分配给变量，并从 Javascript 创建新的：

```pyodide
// re-assign a new value to an existing variable
pyodide.globals.set("x", 'x will be now string');

// create a new js function that will be available from Python
// this will show a browser alert if the function is called from Python
pyodide.globals.set("alert", alert);

// this new function will also be available in Python and will return the squared value.
pyodide.globals.set("square", x => x*x);

// You can test your new Python function in the console by running
pyodide.runPython("square(3)");
```

随意使用浏览器控制台和上面的示例来处理代码。

## 从 Python 访问 Javascript 域

可以使用 `js` 模块从 Python 访问 Javascript 范围（请参阅将 {ref}`type-translations_using-js-obj-from-py`）。该模块代表全局对象 `window`，允许我们直接操作 DOM 并从 Python 访问全局变量和函数。

```python
import js

div = js.document.createElement("div")
div.innerHTML = "<h1>This element was created from Python</h1>"
js.document.body.prepend(div)
```

请参阅 [](serving_pyodide_packages) 以在本地分发 Pyodide 文件。
