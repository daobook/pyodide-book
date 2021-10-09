# 关于 Pyodide

带有科学堆栈的 Python，编译为 WebAssembly。

Pyodide 可用于您希望在 Web 浏览器中运行 Python 的任何上下文。

Pyodide 通过 WebAssembly 将 Python 3.9 运行时以及包括 NumPy、Pandas、Matplotlib、SciPy 和 scikit-learn 在内的 Python 科学堆栈带入浏览器。目前有超过 75 个[软件包](https://github.com/pyodide/pyodide/tree/main/packages)可用。此外，还可以从 PyPI 安装纯 Python 轮子。

Pyodide 提供了 Javascript 和 Python 之间对象的透明转换。在浏览器中使用时，Python 可以完全访问 Web API。

## 历史

Pyodide 是由 Mozilla 的 [Michael Droettboom](https://github.com/mdboom) 于 2018 年创建的，作为 [Iodide 项目](https://github.com/iodide-project/iodide) 的一部分。Iodide 是一个实验性的基于网络的笔记本环境，用于读写科学计算和交流。

## 贡献

请参阅{ref}`贡献指南<how_to_contribute>`，了解关于提交问题、进行修改和提交拉动请求的提示。Pyodide 是一个独立和社区驱动的开源项目。决策过程在 {ref}`project-governance` 中有所概述。

## 引用

If you use Pyodide for a scientific publication, we would appreciate citations.
Please find us [on Zenodo](https://zenodo.org/record/5135072) and use the citation
for the version you are using. You can replace the full author
list from there with "The Pyodide development team" like in the example below:

```
@software{michael_droettboom_2021_5135072,
  author       = {The Pyodide development team},
  title        = {pyodide/pyodide},
  month        = jul,
  year         = 2021,
  publisher    = {Zenodo},
  version      = {0.18.0a1},
  doi          = {10.5281/zenodo.5135072},
  url          = {https://doi.org/10.5281/zenodo.5135072}
}
```

## Communication

- Mailing list: [mail.python.org/mailman3/lists/pyodide.python.org/](https://mail.python.org/mailman3/lists/pyodide.python.org/)
- Gitter: [gitter.im/pyodide/community](https://gitter.im/pyodide/community)
- Twitter: [twitter.com/pyodide](https://twitter.com/pyodide)
- Stack Overflow: [stackoverflow.com/questions/tagged/pyodide](https://stackoverflow.com/questions/tagged/pyodide)

## License

Pyodide uses the [Mozilla Public License Version
2.0](https://choosealicense.com/licenses/mpl-2.0/).

## Infrastructure support

We would like to thank,

- [Mozilla](https://www.mozilla.org/en-US/) and
  [CircleCl](https://circleci.com/) for Continuous Integration resources
- [JsDelivr](https://www.jsdelivr.com/) for providing a CDN for Pyodide
  packages
- [ReadTheDocs](https://readthedocs.org/) for hosting the documentation.