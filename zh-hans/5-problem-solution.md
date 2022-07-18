# 问题解决
## 第三方库lapjv报错
lapjv是socube不可或缺的依赖包，是实现J-V算法的重要工具。更多问题可以去其官方仓库[src-d/lapjv](https://github.com/src-d/lapjv)。
### 1. numpy缺失

在安装某些版本的lapjv依赖项之前，你需要预先安装numpy（否则你会看到`ModuleNotFoundError`的错误。没有名为numpy的模块），这是因为这些版本，如v1.3.1，在setup.py中直接`import numpy`。

```python
import platform
from setuptools import setup, Extension
import numpy

CXX_ARGS = {
    "Darwin": ["-std=c++11", "-march=native", "-ftree-vectorize"],
    "Linux": ["-fopenmp", "-std=c++11", "-march=native", "-ftree-vectorize"],
    "Windows": ["/openmp", "/std:c++latest", "/arch:AVX2"]
}
```
### 2. RuntimeError: module comiled against API
安装lapjv的二进制版本（wheel），使用时可能存在以下`RuntimeError`。这是因为出版商用来编译lapjv的C库API版本与当前安装的numpy不同。
```
RuntimeError: module compiled against API version 0xf but this version of numpy is 0xe
```
有两种解决方法，如下:
- 源码安装lapjv，使其使用现在安装的numpy进行重新编译。注意使用的C++编译器需要支持setup.py中CXX_ARGS定义的C++版本，例如“-std=c++11”。最新版的lapjv需要使用"-std=c++17"。因此对没有基础的用户具有一定技术挑战。另外源码安装需要提供cpp库的依赖，对于windows来说，可以安装完整的visual studio 2019，或者直接下载[构建工具](https://visualstudio.microsoft.com/visual-cpp-build-tools/)。下载和安装完成后，重启电脑即可使用。
```
pip install lapjv --no-binary lapjv
```
- 安装对应版本的numpy，不过可能会和其他包的依赖numpy版本发生冲突。

## 第三方库pytables出错
该库是pandas库的to_hdf API的依赖库。该包的某些版本缺少所需的动态C库，如tables-3.7.0-cp38-cp38-win_amd64，你可以尝试安装[PyPi](https://pypi.org/project/tables/)中列出的其他版本来解决。其他问题，你可以查看它的[官方GitHub仓库](https://github.com/PyTables/PyTables)。
```
Traceback (most recent call last):
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\site-packages\pandas\compat\_optional.py", line 138, in import_optional_dependency
    module = importlib.import_module(name)
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\importlib\__init__.py", line 127, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 1014, in _gcd_import
  File "<frozen importlib._bootstrap>", line 991, in _find_and_load
  File "<frozen importlib._bootstrap>", line 975, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 671, in _load_unlocked
  File "<frozen importlib._bootstrap_external>", line 843, in exec_module
  File "<frozen importlib._bootstrap>", line 219, in _call_with_frames_removed
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\site-packages\tables\__init__.py", line 45, in <module>
    from .utilsextension import get_hdf5_version as _get_hdf5_version
ImportError: DLL load failed while importing utilsextension: 找不到指定的模块。

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\concurrent\futures\process.py", line 239, in _process_worker
    r = call_item.fn(*call_item.args, **call_item.kwargs)
  File "d:\life_matters\IDRB\深度组学\单细胞组学\SoCube\src\socube\utils\concurrence.py", line 74, in wrapper
    return func(*args, **kwargs)
  File "d:\life_matters\IDRB\深度组学\单细胞组学\SoCube\src\socube\utils\io.py", line 262, in writeHdf
    data.to_hdf(file, key=key, mode=mode, **kwargs)
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\site-packages\pandas\core\generic.py", line 2763, in to_hdf
    pytables.to_hdf(
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\site-packages\pandas\io\pytables.py", line 311, in to_hdf
    with HDFStore(
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\site-packages\pandas\io\pytables.py", line 572, in __init__
    tables = import_optional_dependency("tables")
  File "c:\Users\zhang\anaconda3\envs\socube_test\lib\site-packages\pandas\compat\_optional.py", line 141, in import_optional_dependency
    raise ImportError(msg)
ImportError: Missing optional dependency 'pytables'.  Use pip or conda to install pytables.
```

## docker镜像使用出错
### 1. nvidia-container-cli: initialization error
 docker在windows下默认依赖后端是WSL2。而较早版本的windows 10的WSL2不支持GPU，因此会收到下面的报错。
 ```
Running hook #0:: error running hook: exit status 1, stdout: , stderr: nvidia-container-cli: initialization error: driver error: failed to process request: unknown
 ```
 用户可以选择升级windows 10 21H2等新版本。推荐用户在拥有GPU的linux服务器使用docker。或者用户可以选择纯CPU运行，但速度会较慢。
 ```powershell
docker run `
        -v <Your input data directory absoluate path outside>:/workspace/datasets `
        gcszhn/socube:latest `
        -i datasets/<Your input data file base name>
 ```