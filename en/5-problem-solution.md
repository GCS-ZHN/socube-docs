# Problem Solution

## 5.1 Error about lapjv 

lapjv is an indispensable dependency package for Socube and is an important tool for implementing the J-V algorithm. For more questions, you can go to its official repository [src-d/lapjv](https://github.com/src-d/lapjv).

### 1. numpy missing

You need to pre-install numpy before installing certain versions of lapjv dependencies (otherwise you will see `ModuleNotFoundError` errors. There is no module named numpy), this is because these versions, such as v1.3.1, `import numpy` directly in setup.py. setup.py is the executable file of the python installer, so it will cause the installation to fail.

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

The following `RuntimeError` may exist when installing and using the binary version (wheel) of lapjv. This is because the version of the C library API used by the publisher to compile lapjv is different from the currently installed numpy.
```
RuntimeError: module compiled against API version 0xf but this version of numpy is 0xe
```
There are two solutions to this problem, as follows:
- Installing lapjv from the source so that it recompiles using the current installed numpy. Note that the C++ compiler used needs to support the version of C++ defined by CXX_ARGS in setup.py, e.g. "-std=c++11". The latest version of lapjv needs to use "-std=c++17". Therefore it is technically challenging for users without basic knowledge. Also the installation from source needs to provide a dependency on the CPP library, for windows you can install the full visual studio 2019 or download [build tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) directly. After the download and installation is complete, restart your computer and it will be ready to use.
```bash
pip install lapjv --no-binary lapjv
```
- Install the corresponding version of numpy, but there may be conflicts with other packages that depend on the numpy version.

## 5.2 Error in PyTables

The library is a dependent library for the `to_hdf` API of pandas. Some versions of this package lack the required dynamic C libraries, such as tables-3.7.0-cp38-cp38-win_amd64, you can try installing other versions listed in [PyPi](https://pypi.org/project/tables/) to fix it. For other issues, you can visit its [official GitHub repository](https://github.com/PyTables/PyTables).
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

## 5.3 Error using docker image

### 1. nvidia-container-cli: initialization error

Docker relies on the WSL2 backend by default under Windows. The earlier version of WSL2 in Windows 10 does not support GPU, so you will receive the following error.
```
Running hook #0:: error running hook: exit status 1, stdout: , stderr: nvidia-container-cli: initialization error: driver error: failed to process request: unknown
```
Users can choose to upgrade to a newer version such as Windows 10 21H2. It is recommended to use docker on a linux server with a GPU, or you can choose to run it purely on CPU, but it will be slower.
```powershell
docker run `
        -v <Your input data directory absoluate path outside>:/workspace/datasets `
        gcszhn/socube:latest `
        -i datasets/<Your input data file base name>
```