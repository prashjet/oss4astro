# Package Structure and Writing Tests

This demo we will show how to (1) structure your package, and (2) write and run tests using `pytest` software.

## `pysequence` package

We will use example repository called [pysequence](https://github.com/prashjet/pysequence).  You can clone this repository to follow along with the demonstration. The repository structure should initally looks like this:

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

When using our package, we want to be able to import functions such as `fibonacci_numbers` to be able to use elsewhere. This 

- **module**: a file containing Python definitions and statements to be imported in other places (e.g. the `sequences.py` file is a module)
- **package**: a way of structuring collections of modules so that they can be imported using *dotted module names* e.g. `pysequence.sequences` is the dotted module names for the `sequences.py` file
- **subpackages/submodules** are nested directories/files within the top-level package. It is possible to define a hierarchy of subpackages within subpackages etc...

The `__init__.py` is needed to tell Python that the directory it lives in is a package. Each sub-package needs its own `__init__.py` file. In the simplest case, you can leave `__init__.py` as an empty file. Later we will look into when you might want to add 

### Where can I import from?

By default, you can import a package when you are in the directory containing that package, e.g. only when you're in the main directory for the `pysequence` repository can you `import pysequence`.

This isn't what we want! One of the main reasons for creating a software package is to keep the package separate from scripts and notebooks which use the package. Without this, it quickly becomes difficult to re-use your software in different contexts.

There are two solutions to this. This first solution is to *install* your software package. We will eventually do this for your package! For the time being, we will use the temporary solution of adding the directory containing the package to your Python path. I will do this interactively within an `ipython` session as follows,

```
import sys
sys.path.append('/Users/prash/Astro/Teaching/23_ws_OSS/my_example_repos/pysequence/')
```

A more permanent solution is to edit the `PYTHONPATH` variable in your `bashrc` file (see [here](https://bic-berkeley.github.io/psych-214-fall-2016/using_pythonpath.html) for instructions).

### different ways to `import`

Now that Python knows where to find our package, we can import modules and their contents functions. To access the `fibonacci_numbers` function, we have two basic options. Either importing the 

```
In [1]: import pysequence.sequences
In [2]: pysequence.sequences.fibonacci_numbers(5)
Out[2]: [1, 1, 2, 3]
```

or importing the function from the module,

```
In [3]: from pysequence.sequences import fibonacci_numbers
In [4]: fibonacci_numbers(5)
```

We can also use an alias using the `as` command, 

```
In [5]: from pysequence.sequences import fibonacci_numbers as fib
In [6]: fib(5)
```

There we made an alias for the function, but we could also have created an alias for the 

```
In [7]: import pysequence.sequences as psq
In [8]: psq.fibonacci_numbers(5)
```

Submodules and functions within them can be imported similarly, e.g.

```
In [7]: import pysequence.sequences.analysis as psq_analysis
In [8]: psq_analysis.get_intersection_of_sequences
```

or 

```
In [7]: from pysequence.sequences.analysis import get_intersection_of_sequences
```

These are all of our options with the basic setup of `__init__.py`. See the section about [`__init__.py`](what-can-go-in-`__init__.py`) for more advanced options.

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

