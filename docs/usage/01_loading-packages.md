(loading_packages)=
# [加载包](https://pyodide.org/en/stable/usage/loading-packages.html)

导入 Pyodide 后，只有 Python 标准库可用。要使用其他包，你需要使用以下两种方式加载它们：

- {any}`pyodide.loadPackage` 使用 Pyodide 构建的包，或
- {any}`micropip.install` 对于在 PyPi 或其他 URL 上可用的纯 Python 包。

```{note}
{mod}`micropip` 也可以用来加载 Pyodide 中构建的包（在这种情况下，它依赖于 {any}`pyodide.loadPackage`）。
```

如果你用 {any}`pyodide.loadPackagesFromImports` Pyodide 将自动下载代码段导入的所有包。这对于创建 repl 特别有用，因为用户可能会导入意外的包。目前，{any}`loadPackagesFromImports <pyodide.loadPackagesFromImports>` 不会从 PyPi 下载包，它只会下载包含在 Pyodide 发行版中的包。

## 使用 {any}`pyodide.loadPackage` 加载包

Pyodide 官方库中包含的包可以使用 {any}`pyodide.loadPackage` 加载：

```js
pyodide.loadPackage("numpy");
```

也可以从自定义 URL 加载包：

```js
pyodide.loadPackage("https://foo/bar/numpy.js");
```

URL 中的文件名必须是 `<package-name>.js`，并且必须有一个名为 `<package-name>.data` 的附带文件在同一目录下。

当您从官方存储库请求一个包时，该包的所有依赖项也会被加载。当从自定义 URL 加载包时，依赖关系解析尚未实现。

一般来说，一个包是不允许加载两次的。但是，可以通过在加载依赖包之前加载具有相同包名的自定义 URL 来覆盖依赖。

通过向 `loadPackage` 传递一个列表，也可以同时加载多个包。

```js
pyodide.loadPackage(["cycler", "pytz"]);
```

{any}`pyodide.loadPackage` 返回一个 `Promise`，当所有的包加载完成时解析：

```javascript
let pyodide;
async function main() {
  pyodide = await loadPyodide({ indexURL: "<some-url>" });
  await pyodide.loadPackage("matplotlib");
  // matplotlib is now available
}
main();
```

(micropip)=

## Micropip

### 从 PyPI 安装包

Pyodide 支持使用 {mod}`micropip` 从 PyPI 安装纯 Python 轮子。{func}`micropip.install` 返回一个 Python [Future](https://docs.python.org/3/library/asyncio-future.html)，所以你可以等待 future，或者在包加载完成后使用 Python future API 工作：

```pyodide
pyodide.runPythonAsync(`
  import micropip
  await micropip.install('snowballstemmer')
  import snowballstemmer
  stemmer = snowballstemmer.stemmer('english')
  print(stemmer.stemWords('go goes going gone'.split()))
`);
```

Micropip 通过检查从 PyPi JSON API 中预先记录的（pre-recorded）散列摘要下载的轮子的散列来实现文件完整性验证。

(micropip-installing-from-arbitrary-urls)=

### 从任意 URL 安装轮子

纯 Python 轮子也可以从任何 URL 安装 {mod}`micropip`，

```py
import micropip
micropip.install(
    'https://example.com/files/snowballstemmer-2.0.0-py2.py3-none-any.whl'
)
```

Micropip 根据一个文件是否以 ".whl" 结尾来判断它是否是一个 URL。whl”。URL 中的轮子名必须遵循 [PEP 427 命名约定](https://www.python.org/dev/peps/pep-0427/#file-format)，如果轮子是使用标准 Python 工具（`pip wheel`，`setup.py bdist_wheel`）制作的，则必须遵循 PEP 427 命名约定。

所有必需的依赖项必须预先安装 {mod}`micropip` 或者 {any}`pyodide.loadPackage`。

如果文件在远程服务器上，服务器必须设置跨源资源共享（CORS）头以允许访问。否则，您可以将 CORS 代理添加到 URL 中。但是请注意，使用第三方 CORS 代理具有安全隐患，特别是由于我们无法检查文件完整性，这与从 PyPi 安装不同。

## 例子

```html
<html>
  <head>
    <meta charset="utf-8" />
  </head>
  <body>
    <script
      type="text/javascript"
      src="https://cdn.jsdelivr.net/pyodide/v0.18.1/full/pyodide.js"
    ></script>
    <script type="text/javascript">
      async function main() {
        let pyodide = await loadPyodide({
          indexURL: "https://cdn.jsdelivr.net/pyodide/v0.18.1/full/",
        });
        await pyodide.loadPackage("micropip");
        await pyodide.runPythonAsync(`
        import micropip
        await micropip.install('snowballstemmer')
        import snowballstemmer
        stemmer = snowballstemmer.stemmer('english')
        print(stemmer.stemWords('go goes going gone'.split()))
      `);
      }
      main();
    </script>
  </body>
</html>
```