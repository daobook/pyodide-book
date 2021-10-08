(downloading_deploying)=
# [下载和部署 Pyodide](https://pyodide.org/en/stable/usage/downloading-and-deploying.html#downloading-deploying)


## 下载 Pyodide
### CDN

Pyodide 包，包括 `pyodide.js` 文件，可从 JsDelivr CDN 获得，

| 渠道             | `indexURL`                                        | 注释                                                                                 | REPL                                               |
| ------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------- | -------------------------------------------------- |
| 最新发布 | `https://cdn.jsdelivr.net/pyodide/v0.18.1/full/` | 推荐，浏览器缓存                                                      | [链接](https://pyodide.org/en/stable/console.html) |
| Dev (`main` branch) | `https://cdn.jsdelivr.net/pyodide/dev/full/`     | 为 main 上的每次提交重新部署，没有浏览器缓存，应该只用于测试 | [链接](https://pyodide.org/en/latest/console.html) |

要访问特定文件，请将文件名附加到 `indexURL`。例如，在 `pyodide.js` 的情况下，`"${indexURL}pyodide.js"`。

```{warning}
之前的 CDN `pyodide-cdn2.iodide.io` 已弃用，不应使用。
```

### Github 发布

您还可以从 [Github 版本](https://github.com/pyodide/pyodide/releases)（`pyodide-build-*.tar.bz2` 文件）下载 Pyodide 包，自己为它们提供服务，如以下部分所述。

(serving_pyodide_packages)=
## 提供 Pyodide 包

提供 Pyodide 包

如果您构建了 Pyodide 发行版或下载了发行版 tarball，则需要使用适当的 headers 提供 Pyodide 文件。

### 在本地服务

使用 Python 3.7.5+，您可以通过启动在本地提供 Pyodide 文件

```
python -m http.server
```

来自 Pyodide 分发文件夹。

将您的 WebAssembly 感知浏览器指向 <http://localhost:8000/console.html> 并打开您的浏览器控制台以通过 Pyodide 查看 Python 的输出！

### 远程部署

任何能够托管静态文件并正确设置 WASM MIME 类型和 CORS headers 的解决方案都可以使用。例如，您可以使用 Github Pages 或类似的服务。

有关优化大小和加载时间的其他建议，请参阅有关[部署的 Emscripten 文档](https://emscripten.org/docs/compiling/Deploying-Pages.html)。
