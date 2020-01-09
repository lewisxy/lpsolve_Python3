# Compile lpsolve module for Python 3

## Introduction

`lpsolve` is a library for solving Mixed Integer Linear Programming (MILP) implemented in C.
It does have a Python extension but haven't been updated for a long time and only support 
Python 2. As the Python C API changed a lot from 2 to 3, support for Python 3 is not as simple as
just recompile it for Python 3. This guide will walk through the process for compiling and 
installing `lpsolve` module for Python 3.

## Steps
1. Download and extract the latest version of [`lpsolve`][source] and [Python module][pysource] 
source code. The current version is 5.5.2.5 and these two directories can be merged into `lp_solve_5.5`
(`extra` directory inside `lp_solve_5.5`). 

2. Compile the library following the readme. The script for building libraries are in `lp_solve_5.5/lpsolve55`. You should follow the `readme.txt` in this directory. This should produce `liblpsolve55.a`
and `liblpsolve55.so`(`liblpsolve55.dll` on Windows) in `lp_solve_5.5/lpsolve55/bin/<your os>`.

3. Copy (do not delete) these files to where installed libraries should go. 
On Linux/MacOS (it should be `/usr/local/lib`)

4. In `lp_solve_5.5/extra/Python`, you will need to modify the `setup.py` to build it for Python3.
The script is a bit messy and you need to perform the following changes:
 - Change `print ...` to `print(...)`
 - Delete all the `if ... else`, everything after all `import ...` and above `setup(...)`. Don't delete
 - `print(...)` you just changed.
 - Add the followings between `import ...` and `print(...)`
   - Add `LPSOLVE55 = '../../lpsolve55/bin/<your os>'`
   - Add `WIN32 = 'NOWIN32'` unless you are compiling for `win32` (not tested)
   - If you want to build with Numpy, add `NUMPY = 'NUMPY'` otherwise `NUMPY = 'NONUMPY'`
   - If you want to build with Numpy, you also need to find your Numpy include directory.
 It should be located in `site-packages` directory of you Python installation directory, 
 and the overall path looks something like this 
 `'<??>/Python/<3.x>/lib/python/site-packages/numpy/core/include'`. If you go to this
 directory, inside `numpy` directory, there should be a lot of `.h` files.
 Add `NUMPYPATH='<??>/Python/<3.x>/lib/python/site-packages/numpy/core/include'`
 - In `setup(...)` set `include_dirs=['../..', NUMPYPATH]`

5. If you are on MacOS, change `#include <malloc.h>` in `hash.c` to `#include <stdlib.h>`.

6. Follow the changed to `pythonmod.c` mentioned [here][ref] except for the followings: 
You should change all `PyString_<...>` to `PyBytes_<...>`. It is advised to use search and 
replace functionality of the text editor. **DO NOT** change it to `PyUnicode_<...>`
The code will compile, but you will run into issues when you use it in Python 3. This is because 
of the change of Python C API, if you want to learn more about the change, see [this][porting]. 
The short answer is that String in C (lpsolve source code) is equivalent to `bytes` type but 
not `str` type in Python3.

7. Now you should be able to compile the library. Use `python3 setup.py install` to compile and
install the package. (Note: if your system default `python` is Python 3, you can just run 
`python3 setup.py install`, it is important not to run `setup.py` from Python 2).

8. If you see no errors, the package should be installed. To test whether it works, opens a
Python 3 shell in your terminal, type `from lpsolve55 import *`. It should not print anything.
If it prints errors, you probably need to change some code again :(.

9. Follow the [documentation][docs] to try a few examples. You need to change all string literals
`'...'` to bytes literals b`'...'` (just add a `b` in the front of the literal) in order to make
it work. All other things should be the same as the module in Python 2.

## Common Issues
1. If your module could not found library `liblpsolve55.so`, it means your library
is not in the path. Try to set the environmental variable `LD_LIBRARY_PATH` to the location of your
library.

2. If your module could not initalize because of `unknown symbols xxx`. Double check the changes in 
`pythonmod.c`, especially for the replacements.

3. On Linux, if `gcc` throws error, try to compile with `clang` (`cc=clang` in `ccc`)

If you have other questions, please open issue.


[pysource]: https://sourceforge.net/projects/lpsolve/files/lpsolve/5.5.2.5/lp_solve_5.5.2.5_Python_source.tar.gz/download
[source]: https://sourceforge.net/projects/lpsolve/files/lpsolve/5.5.2.5/lp_solve_5.5.2.5_source.tar.gz/download
[ref]: https://stackoverflow.com/a/50592697
[porting]: http://python3porting.com/cextensions.html#strings-and-unicode
[docs]: http://lpsolve.sourceforge.net/5.5/Python.htm
