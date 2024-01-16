# Automatically Building and Testing your project in GitHub

Today we will use *GitHub Actions* to automatically build our Python project, run the test suite, and display the output. By the end of today you should know:

1. What are GitHub Actions?
2. How to setup a GitHub Action for a Python project.
3. How to add a coverage badge to your project.

Automatically building and testing your project can streamline code development for yourself and other contributors. These topics fall under the wider umbrellas of *Continuous Integration* and *Continuous Delivery* (CI/CD).

## Example repo

As before, we'll use an example repository called [pysequence](https://github.com/prashjet/pysequence). You can clone this repository to follow along with the demonstration.

Several of the features we have developed throughout the course have now been merged into the main branch of the `pysequence` repository, and will be necessary to run today's demonstration. Specifically, we will need:

- a `test` directory compatible with `pytest`
- a valid `pyproject.toml` file 

See our previous notes about [testing](https://github.com/prashjet/oss4astro/tree/main/package_and_test) and [release](https://github.com/prashjet/oss4astro/tree/main/releasing_a_package) for more details.

Let's first remind ourselves of the test we have written in this repository by running `pytest` from the main directory,

```
pytest .
```

## What are GitHub Actions?

The definition from the GitHub documentation page on [understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions):

>You can configure a GitHub Actions *workflow* to be triggered when an *event* occurs in your repository, such as a pull request being opened or an issue being created. Your *workflow* contains one or more *jobs* which can run in sequential order or in parallel. Each job will run inside its own *virtual machine runner*, or inside a container, and has one or more *steps* that either run a script that you define or run an action, which is a reusable extension that can simplify your workflow.

To understand these terms, let's walk through the example in `.github/workflows/workflow_01.yml`

### Workflow

The directory `.github/workflows` contains workflows. This should live in the top-level directory of your repository.

Each file in `.github/workflows` corresponds to a separate workflow. The files are in [YAML](https://yaml.org/) format.

Let's look into `workflow_01.yml`. Currently, this is the only workflow which is active.

### Event

>You can configure a GitHub Actions *workflow* to be triggered when an *event* occurs in your repository...

The event is specified in the `on` section of the workflow file. This event

```
on:
  push:
    branches: [ "main" ]
```

corresponds to the event "whenever something is pushed to the main branch".

Other types of events which can trigger a workflow include:

- pull-requests being made
- issues being raised
- project being released 

... and many more. See [GitHub docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) for a full list of possible events.

### Jobs

> Your *workflow* contains one or more *jobs* which can run in sequential order or in parallel.

The `workflow_01.yml` file specified one job which is called `build``

```
jobs:
  build:

    ...
```

### Virtual machine runner

> Each job will run inside its own *virtual machine runner*, or inside a container, 

In `workflow_01.yml` we specify that the virtual machine runner for the job `build` is a machine running the latest version of the Ubuntu operating system,

```
jobs:
  build:

    runs-on: ubuntu-latest
    ...
```

### Steps

> Each job ... has one or more *steps* that either run a script that you define or run an action, which is a reusable extension that can simplify your workflow.

Our job has four `steps`:

```
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-python@v3
      with:
        python-version: '3.10'
    - name: Install
      run: |
        python -m pip install --upgrade pip
        python -m pip install pytest coverage
        pip install .
    - name: Test with pytest
      run: pytest
```

The first two steps are specified with the `uses` command. These steps use predefined *actions*, i.e.

> a reusable extension that can simplify your workflow.

The [first action](https://github.com/actions/checkout) checks out the repository, the [second one](https://github.com/actions/setup-python) sets up Python (where we specify to use version 3.10). For both actions, the suffix '@v3' means that we use version 3 of that action.

The remaining two steps don't run predefined actions, but instead a set of commands defined by us. These steps are given names, and are specified using the `run` command under which we list a sequence of command-line commands. For example, the `Install` step consists of three commands which:

- upgrades `pip` (which is already installed when we used the `setup`) 
- installs `pytest`, ``
- run `pip install .` to install the `pysequence` package. This is as if you ran `pip install .` from the top-level directory of your repository, e.g. installing the dependencies listed in your `pyproject.toml` file.

The `Test with pytest` step then runs the tests. 

## Demonstrate the workflow

Let's see the workflow in action. To do this, we have to trigger the specified *event*, i.e. a push to the main branch. Let's do this twice: first let's push a benign change, then push a change that will invalidate one of the tests.

## Some more advanced features

Let's look into the `workflow_02.yml` to see some more advanced, and useful, features:

### Multiple events

This workflow is triggered by two events: pushes to the main branch or by the creation of pull requests into main

```
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```

### Run simultaneous on multiple OS and Python versions

This workflow has a new field `strategy`:

```
strategy:
    matrix:
        os: [ubuntu-latest, macos-latest]
        python-version: ["3.9", "3.10", "3.11"]
```

which specifies a matrix (i.e. a grid) of different operating systems and python versions to use for running the job. This will run 6 jobs in total i.e. (2 operating systems) X (3 python versions).

The matrix elements are used in the appropriate places in the rest of the job, i.e. `runs-on: ${{ matrix.os }}`, and `python-version: ${{ matrix.python-version }}`.

## Demonstrate the workflow

Let's demonstrate this workflow by submitting a pull request to the main branch. This will run six tests at once!

## Coverage 

Coverage is the percentage of code, counted by lines, which is touched by your tests. It's good to aim for high coverage, and to display this achievement proudly in your GitHub repository, using a *badge* such as,

<p align="center">
  <img width="100" src="./imgs/badge.png">
</p>

A green coverage badge is quality assurance for anyone looking at your repository. Having said that... be aware that 100% coverage does *not* mean that code is 100% safe, since there may be contexts which are not covered by tests.

We will use GitHub Actions and the third party [CodeCov](https://about.codecov.io/) tool to generate and display our coverage badge. But first, let's look at the underlying Python tool which measures coverage, namely the [`coverage`](https://coverage.readthedocs.io/en/7.4.0/) package.

### The Python `coverage` tool 

First let's install `coverage`,

```
pip install coverage
```

then run our test via the coverage command

```
coverage run -m pytest
```

and then generate a coverage report,

```
coverage report 
```

which outputs the following, 

```
Name                                       Stmts   Miss  Cover
--------------------------------------------------------------
pysequence/__init__.py                         2      0   100%
pysequence/analysis/__init__.py                1      0   100%
pysequence/analysis/compare_sequences.py      18      6    67%
pysequence/sequences.py                       36     12    67%
pysequence/test_compare_sequences.py           5      0   100%
--------------------------------------------------------------
TOTAL                                         62     18    71%
```

which breaks down, for python each file in your project, the number/percentage of lines missed/covered by a test. 

For a nicer presentation you can create an annotated HTML page detailing missed lines via 

```
coverage html
```

then open `htmlcov/index.html` in your browser to see a detailed coverage report.

### [CodeCov](https://about.codecov.io/)



### Alternatives to CodeCov

CodeCov is a third-party tool which is free to use for open-source projects. Other third-party tools such as [Coveralls](https://coveralls.io/) provide similar help.

There are some potential downsides to using third-party tools e.g. sometimes their servers go down, or they may change pricing plans etc. Given that `coverage` does most of the heavy lifting in computing and creating the coverage report, it should be simple to show coverage badge without using any third party tool. For the time being, however, existing solutions to do this look a little convoluted. For example, this [GitHub Action](https://github.com/tj-actions/coverage-badge-py) creates the badge file, commits and pushes this to your repository, which requires a secret GitHub access token. It's do-able, but (at least while there are free 

## Other GitHub actions



