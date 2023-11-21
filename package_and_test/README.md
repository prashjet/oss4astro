# Package Structure and Writing Tests

Today I will demonstrate how to (1) structure your package (based on info from the [official Python documentation](https://docs.python.org/3/tutorial/modules.html#packages)), and (2) write and run tests using [`pytest`](https://docs.pytest.org/en/7.4.x/) framework. For both sections, we'll use an example repository called [pysequence](https://github.com/prashjet/pysequence). You can clone this repository to follow along with the demonstration. 

## Package Structure

The repository structure should initally looks like this:

```
├── pysequence                          # the repository
│   ├── README.md
│   ├── pysequence                      # top-level package
│   │   ├── __init__.py                 # to initialize the pysequence package
│   │   ├── sequences.py                # a module within the package
│   │   ├── analysis                    # a subpackage
│   │   │   ├── __init__.py             # to initialize the subpackage
│   │   │   ├── compare_sequences.py    # a submodule i.e. a module within the subpackage 
```

At the top of the tree is the repository, called `pysequence`. This will contain the source code for the Python package itself, which lives in the directory `pysequence/pysequence/` i.e. with the package name repeated twice. The repitition of the package name is a common convention, but you can choose a different name for the repository and the package directories if you prefer. Aside from the package source code, the repository also contains many other files e.g. a `README`, and a `test` directory (which we will create today).

The file `sequences.py` contains functions defining mathematical sequences, e.g.

<p align="center">
  <img width="500" src="./imgs/fibonacci.png">
</p>

When using our package, we want to be able to import functions such as `fibonacci_numbers` to be able to use elsewhere.

Now we're ready for some definitions:
- **module**: a file containing Python definitions and statements to be imported in other places (e.g. the `sequences.py` file is a module)
- **package**: a way of structuring collections of modules so that they can be imported using *dotted module names* e.g. `pysequence.sequences` is the dotted module names for the `sequences.py` file
- **subpackages/submodules** are nested directories/files within the top-level package. It is possible to define a hierarchy of subpackages within subpackages etc...

The `__init__.py` is needed to tell Python that the directory it lives in is a package. Each sub-package needs its own `__init__.py` file. For the simplest case, let's leave `__init__.py` as an empty file.

### Where can I import from?

By default, you can only import a package when you are in the directory containing that package, e.g. only when you're in the main directory for the `pysequence` repository can you `import pysequence`.

This isn't what we want! One of the main reasons for creating a software package is to keep the package separate from scripts and notebooks which use the package. Without this, it quickly becomes difficult to re-use your software in different contexts.

There are two solutions to this. This first solution is to *install* your software package. We will eventually do this, but for the time being, we will use the temporary solution of adding the directory containing the package to your Python path. I will do this interactively inside an `ipython` session as follows,

```
import sys
sys.path.append('/Users/prash/Astro/Teaching/23_ws_OSS/my_example_repos/pysequence/')
```

A more permanent solution is to edit the `PYTHONPATH` variable in your `bashrc` file (see [here](https://bic-berkeley.github.io/psych-214-fall-2016/using_pythonpath.html) for instructions).

### different ways to `import`

Now that Python knows where to find our package, we can import modules and their functions. To access the `fibonacci_numbers` function, we have two basic options. Either importing the module, then appending the function name when we want to use it,

```
In [1]: import pysequence.sequences
In [2]: pysequence.sequences.fibonacci_numbers(5)
Out[2]: [1, 1, 2, 3]
```

or importing the function *from* the module,

```
In [3]: from pysequence.sequences import fibonacci_numbers
In [4]: fibonacci_numbers(5)
```

We can also create an alias via the *as* command, 

```
In [5]: from pysequence.sequences import fibonacci_numbers as fib
In [6]: fib(5)
```

In that case, we made an alias for the function name, but we could also have created an alias for the module,

```
In [7]: import pysequence.sequences as psq
In [8]: psq.fibonacci_numbers(5)
```

Submodules and functions within them can be imported similarly, e.g.

```
In [9]: import pysequence.sequences.analysis as psq_analysis
In [10]: psq_analysis.get_intersection_of_sequences
```

or 

```
In [11]: from pysequence.sequences.analysis import get_intersection_of_sequences
```

These are all of our import in the basic setup. For more advanced options, we need to edit our `__init__.py`.

### `if __name__ == "__main__":`

If you run the file `sequences.py` from the command line, i.e

```
python sequences.py
```

then the code snippet at the bottom is executed. This snippet is inside the check `if __name__ == "__main__"` check. This statement is only `True` when you run the file directly, not when.

This is why we need the `if __name__ == "__main__"` check; otherwise, the code snippet would run every time we imported the `sequences` module. 

### Intra-package References

In the submodule `compare_sequences.py` we can see an example of an intra-package reference, i.e. an import statement within the package.

```
from ..sequences import get_sequence
```

This is a *relative* import, where the double dots `..` mean to look in one package *up* from the current subpackage. A single dot could be used to import a different module within the *same* level of the package.

If we try running the file to execute the example code snippet, as we did with `sequences.py`, we see the following error,

```
ImportError: attempted relative import with no known parent package
```

i.e. you cannot run the 

There are a couple of (workarounds)[https://stackoverflow.com/questions/16981921/relative-imports-in-python-3] for this. Maybe the most straightforward is to replace the relative import statement with an absolute import, but then to be able to run the `compare_sequences.py` file as a script, you still have to 

```
import sys
import os

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
sys.path.append(os.path.dirname(SCRIPT_DIR))

from pysequence.sequences import get_sequence
```

But this is a bit of a pain. My recommendation would be to use code snippets such as these in the tests, instead of in the modules themselves

### What can go inside `__init__.py`

So far, all of our `__init__.py` files are empty. In addition to its special role in defining packages, this file is also a regular module, which is executed whenever the package is imported. In principle, it can contain any code that you want to run each time you import the package. There are lots of [opinions](https://www.reddit.com/r/Python/comments/1bbbwk/whats_your_opinion_on_what_to_include_in_init_py/) on whether or not you should take advantage of this in practice.

I think the following is a safe, minimal and useful way to do so. Edit the `__init__.py` of the top-level package, to import all of the top-level modules and subpackages i.e. we add this line to the top,

```
from . import sequences, analysis
```

This allows you to have access to all of the package contents just by importing the top-level package, e.g. we can now do

```
In [12]: import pysequence as psq
In [13]: psq.sequences.fibonacci_numbers(5)
```

To me, this feels like the most familiar way to interact with python packages.

## Testing

Why test?

### Types of test

- unit-tests
- end-to-end tests
- edge-cases

### Python `assert`


### Using `pytest`

Let's write our first test. The code example inside 

In the file `compare_sequences.py` we see an intra-package references, i.e. the line

### What have I gained by doing this...?

https://stackoverflow.com/questions/3697432/how-to-find-list-intersection


### Next time

Continuous integration (CI): automatically run these tests at specified stages of code development (e.g. each time you commit, or make a pull request, or make a versioned code release). Lots of popular CI tools are based on `pytest`. In the next lecture, I'll show you how to set this up!

### Testing best practice

