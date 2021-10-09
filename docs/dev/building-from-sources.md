(building_from_sources)=
# 从源代码构建

构建在 Linux 上是最简单的，在 Mac 上则相对简单明了。对于 Windows，我们目前推荐使用 Docker 镜像（如下所述）来构建 Pyodide。在 Windows 上构建的另一个选择是使用 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 来创建一个 Linux 构建环境。

## 使用 `make` 构建

确保先决条件 [emsdk](https://github.com/emscripten-core/emsdk) 已经安装。Pyodide 将构建一个自定义的、打了补丁的 emsdk 版本，所以不需要在之前自己构建它。

额外的构建先决条件是：

- 一个可以工作的本地编译器工具链，足以构建 [CPython](https://devguide.python.org/setup/#linux)。
- 一个本地的 Python 3.9 来运行构建脚本。
- CMake
- PyYAML
- FreeType 2 开发库来编译 Matplotlib。
- Cython 来编译 SciPy
- SWIG 来编译 NLopt
- gfortran（GNU Fortran 95 编译器）
- [f2c](http://www.netlib.org/f2c/)
- [ccache](https://ccache.samba.org)（可选），强烈推荐用于更快的重建。

在 Mac 上，你还将需要：

- [Homebrew](https://brew.sh/) for installing dependencies
- System libraries in the root directory (
  `sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /`
  should do it, see https://github.com/pyenv/pyenv/issues/1219#issuecomment-428305417)
- coreutils for md5sum and other essential Unix utilities (`brew install coreutils`)
- cmake (`brew install cmake`)
- Cython to compile SciPy (`brew install cython`)
- SWIG to compile NLopt (`brew install swig`)
- pkg-config (`brew install pkg-config`)
- openssl (`brew install openssl`)
- gfortran (`brew cask install gfortran`)
- f2c: Install wget (`brew install wget`), and then run the buildf2c script from
  the root directory (`sudo ./tools/buildf2c`)

在安装完构建先决条件后，从命令行运行：

```bash
make
```

## 使用 Docker

我们在 Docker Hub 上提供了一个基于 Debian 的 Docker 镜像，并且已经安装了依赖项，使 Pyodide 的构建更加容易。除此之外，我们还提供了一个预构建的镜像，可用于 Pyodide 的快速定制和部分构建。请注意，在 Mac 上使用非预建的 Docker 镜像构建会非常慢，如果可能的话，最好在主机上构建。

1. 安装 Docker
2. 从 Pyodide 的 git 签出，运行 `./run_docker` 或者 `./run_docker --pre-built`
3. 运行 `make` 来构建。

注意：你可以通过设置环境变量 `EMSDK_NUM_CORE`、`EMCC_CORES` 和 `PYODIDE_JOBS` 来控制分配给构建的资源（每个变量的默认值是 4）。

如果 `make` 的运行在随后的每次尝试中都会确定地停止，增加 docker 容器可用的最大 RAM 用量可能会有帮助[这与系统内部的物理 RAM 容量不同]。理想情况下，docker 容器至少要有 3GB 的内存才能顺利地构建 Pyodide。这些设置可以通过 Docker 首选项改变（见[这里](https://stackoverflow.com/questions/44533319/how-to-assign-more-memory-to-docker-container)）。

你可以在主机上编辑源码签出中的文件，然后在 Docker 环境中反复运行 `make` 来测试你的改变。

(partial-builds)=
## Partial builds

要在 Pyodide 中构建一个可用软件包的子集，请将环境变量 `PYODIDE_PACKAGES` 设置为一个逗号分隔的软件包列表。比如说，

```
PYODIDE_PACKAGES="toolz,attrs" make
```

所列软件包的依赖关系也将被自动构建。包的名称必须与 `packages/` 中的文件夹名称完全匹配；尤其是它们是区分大小写的。

要构建一个最小版本的 Pyodide，请设置 `PYODIDE_PACKAGES="micropip"`。软件包 `micropip` 和 `distutils` 总是被自动包含在内（但是一个空的 `PYODIDE_PACKAGES` 会被解释为未设置）。作为一种速记方法，我们可以说 `make minimal`。

## 环境变量

以下环境变量会对构建产生额外的影响：

- `PYODIDE_JOBS`：当适用于并行编译时，传递给 `emmake make` 命令的 `-j` 选项。默认值：3。
- `PYODIDE_BASE_URL`：部署 Pyodide 软件包的基本 URL。它必须以尾部的 `/` 结尾。默认值：`./` 从与 `pyodide.js` 所在的相同的基础 URL 路径加载 Pyodide 包。例如：
  `https://cdn.jsdelivr.net/pyodide/v0.18.1/full/`
- `EXTRA_CFLAGS`：添加额外的编译标志。
- `EXTRA_LDFLAGS`：添加额外的链接器（linker）标志。

设置 `EXTRA_CFLAGS="-D DEBUG_F"` 可以在 Pyodide 核心代码的错误分支中提供详细的诊断信息。这些错误信息经常是有帮助的，即使问题是一个致命的配置问题，Pyodide 甚至不能被初始化。这些错误分支也发生在正确工作的代码中，但它们相对不常见，所以在实践中产生的噪音量不会太大。简称 `make debug` 会自动设置这个标志。

在某些情况下，设置 `EXTRA_LDFLAGS="-s ASSERTIONS=1` 或 `ASSERTIONS=2` 也是有帮助的，但这大大降低了 Pyodide 的链接和运行速度，并在控制台产生大量的噪音。
