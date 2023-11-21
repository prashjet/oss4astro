# Package Structure and Writing Tests

This demo we will show how to (1) structure your package, and (2) write and run tests using `pytest` software.

## `pysequence` example repository

We will use example repository called [pysequence](https://github.com/prashjet/pysequence).  You can clone this repository to follow along with the demonstration. The repository structure currently looks like this:

```
├── pysequence
│   ├── README.md
│   ├── pysequence                      # top-level package
│   │   ├── __init__.py                 # initialize the pysequence package
│   │   ├── sequences.py                # a module of pysequence
│   │   ├── analysis                    # a subpackage for analysis 
│   │   │   ├── __init__.py             # initialize the subpackage
│   │   │   ├── compare_sequences.py    # a submodule of pysequence 
```

## `pysequence` package structure

The actual source code is located in the directory `pysequence/pysequence/`, i.e. with the package name repeated twice. This is a common convention, and it keeps your source code separate from other types of files, e.g. the `README` and tests (which we will create today). 

The file `sequences.py` contains functions defining mathematical sequences. The 

Definitions:

- a module is a file containing Python definitions and statements to be imported in other places (e.g. `pysequence.sequences`
- a package is a way of structuring 
- subpackages/submodules are nested directories/

### Where can I import from?

Three options:
- within the directory itself
- Intermediate: using the PYTHONPATH
- Final: once installed using pip

### different ways to `import`

With the basic setup.

See the section What can go in `__init__.py` for more advanced.

### `if __name__ == "__main__":`

What is this?

Demo what happens if it isn't there.

### Intra-package References

In the file `compare_sequences.py` we see an intra-package references, i.e. the line

```
from ..sequences import get_sequence
```
This means XYZ.

If we try running the file to execute the example code, as we did with `sequences.py`, we see the following error,

...

A workaround would be to replace the relative import statement with the following,

```
import sys
sys.path.append('../../')
from pysequence.sequences import get_sequence
```

But this isn't great. Recommend: this type of example code better belongs within tests. See below!

### What can go in `__init__.py`

So far, all of our `__init__.py` files are empty. What is it!?

When is it run?

What can go into it e.g. for less cumbursome imports.

## Testing
