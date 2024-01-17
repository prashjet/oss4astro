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
- a `pyproject.toml` file to install the project

See our previous notes about [testing](https://github.com/prashjet/oss4astro/tree/main/package_and_test) and [release](https://github.com/prashjet/oss4astro/tree/main/releasing_a_package) for more details.

Let's first refamiliarise ourselves of this repository by running the tests,

```
pytest .
```

## What are GitHub Actions?

The definition from the GitHub documentation page on [understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions):

>You can configure a GitHub Actions *workflow* to be triggered when an *event* occurs in your repository, such as a pull request being opened or an issue being created. Your *workflow* contains one or more *jobs* which can run in sequential order or in parallel. Each job will run inside its own *virtual machine runner*, or inside a container, and has one or more *steps* that either run a script that you define or run an action, which is a reusable extension that can simplify your workflow.

To understand these terms, let's walk through the example in `.github/workflows/workflow_01.yml`

### Workflow

The directory `.github/workflows` contains workflows.
This should live in the top-level directory of your repository.
Each file in `.github/workflows` corresponds to a separate workflow.
The files are in [YAML](https://yaml.org/) format.
Let's look into `workflow_01.yml` - currently this is the only workflow which is active.

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

- pull-requests being made,
- issues being raised,
- project being released,

... and many more. See [GitHub docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) for a full list of possible events.

### Jobs

> Your *workflow* contains one or more *jobs* which can run in sequential order or in parallel.

The `workflow_01.yml` file specifies one job which is called `build``

```
jobs:
  build:
    ...
```

You can name your job however you like.

### Virtual machine runner

> Each job will run inside its own *virtual machine runner*, or inside a container, 

In `workflow_01.yml` we specify that the virtual machine runner for the job `build` is a machine running on the latest version of the Ubuntu operating system,

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
        python -m pip install pytest
        pip install .
    - name: Test with pytest
      run: pytest
```

The first two steps are specified with the `uses` command. These steps execute predefined *actions*, i.e.

> a reusable extension that can simplify your workflow.

The [first action](https://github.com/actions/checkout) checks out the repository, the [second one](https://github.com/actions/setup-python) sets up Python (where we specify the Python version to be 3.10). For both, the suffix '@v3' means that we use version 3 of that action.

The remaining two steps don't run predefined actions, but instead a set of commands defined by us. These steps are given names, and are specified using `run`, under which we list a sequence of command-line instructions. For example, the `Install` step consists of three commands which:

- upgrades `pip` (which is installed from the `setup-python` action),
- installs `pytest`,
- run `pip install .` to install the `pysequence` package. This works as if you ran `pip install .` from the top-level directory of your repository, e.g. installing the dependencies listed in your `pyproject.toml` file.

The `Test with pytest` step then runs the tests. 

## Demonstrate the workflow

Let's see the workflow in action. To do this, we have to trigger the specified *event*, i.e. a push to the main branch. Let's do this twice: first let's push a benign change, then push a change that will invalidate one of the tests.

## Some more advanced features

Let's look into the `workflow_02.yml` to see some more advanced - and useful - features:

### Multiple events

So far, I have hacked `workflow_02.yml` to be inactive by setting the trigger even to be "the repository is deleted". Let's comment those lines out,

```
# on:
#   delete
```

and un-comment the real event. This workflow is now triggered by two possible events: pushes to the main branch or by the creation of pull requests into main brach,

```
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```

This is a useful 

### Run simultaneous on multiple OS and Python versions

This workflow also has a new field `strategy`:

```
strategy:
    matrix:
        os: [ubuntu-latest, macos-latest]
        python-version: ["3.9", "3.10", "3.11"]
```

This specifies a matrix (i.e. a grid) of different operating systems and python versions to use for running the job. This will run 6 jobs in total i.e. (2 operating systems) X (3 python versions).

The matrix elements are used in the appropriate places in the rest of the job, i.e. `runs-on: ${{ matrix.os }}`, and `python-version: ${{ matrix.python-version }}`.

### Demonstrate the workflow

Let's demonstrate this workflow by submitting a pull request to the main branch. This will run six tests at once!

## Coverage

Coverage is the percentage of code, counted by lines, which is touched by your tests. It's good to aim for high coverage, and to display this achievement proudly in your GitHub repository, using a *badge* such as,

<p align="center">
  <img width="200" src="./img/badge.png">
</p>

A green coverage badge is quality assurance for anyone looking at your repository. Having said that, be aware that 100% coverage does *not* mean that code is 100% safe, since there may be contexts which are not covered by tests.

We will use GitHub Actions and the third party [CodeCov](https://about.codecov.io/) tool to generate and display our coverage badge. But first, let's look at the underlying Python tool which measures coverage: the [`coverage`](https://coverage.readthedocs.io/en/7.4.0/) package.

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

For a nicer presentation you can create an annotated HTML page which shows which lines have been missed:

```
coverage html
```

Open `htmlcov/index.html` in your browser to see the detailed coverage report.

### [CodeCov](https://about.codecov.io/)

CodeCov is a third-party application.
We will use a GitHub action to upload our coverage report to CodeCov in order to produce our coverage badge.

First step: sign up for a CodeCov account using your GitHub profile. It's important to sign up using your GitHub profile for permissions to be set correctly.

Next: edit the workflow file. Let's look at `workflow_03.yml` for an example. This is a copy of `workflow_01.yml` with three differences:

- we install `coverage` along with `pytest` when installing dependencies
- run the tests using `coverage run -m pytest`
- we add an extra step which uploads the coverage report to `CodeCov`

Let's trigger this workflow and see what happens.

Once the job is successfully completed, we can then log into `CodeCov` to look at the coverage report.

Here we can also navigate to `pysequence --> Settings --> Bages & Graphs` to find the URL correspoding to our coverage badge.  It gives three options for the badge format: Markdown, HTML, or RST. We'll be adding the badge to our project `README.md` which is in Markdown, so copy and paste the Markdown URL to the top of your `README.md`.

See [here](https://docs.codecov.com/docs/commit-status) for more info on how to configure your CodeCov action. In short, you can put a `codecov.yml` file in your repository to specify options. One example: you can set a target coverage threshold (e.g. 70%). If coverage drops below this threshold (e.g. with a pull-request that adds lots of new code with no new tests) then the job will fail and, hopefully, this will motivate you to make more tests.

### Alternatives to CodeCov

CodeCov is a third-party tool which is free to use for open-source projects. Other third-party tools such as [Coveralls](https://coveralls.io/) provide similar services.

There are potential downsides to using third-party tools e.g. sometimes their servers go down, or they may change pricing plans etc. Given that `coverage` does most of the heavy lifting in computing and creating the coverage report, it should be simple to show coverage badge without using any third party tool. For the time being, however, existing solutions to do this are slightly convoluted. For example, this [GitHub Action](https://github.com/tj-actions/coverage-badge-py) creates the badge file, commits and pushes this to your repository, which requires a secret GitHub access token. Try this out if you want to avoid third-party apps.

## Other GitHub actions

There is a whole zoo of GitHub actions available for all kinds of tasks, not just testing. For example, there are actions which can automatically upload your project to PyPI, build and upload your documentation to ReadTheDocs, or even directly submit articles to scientific journals.

Read about ways to explore GitHub actions [here](https://docs.github.com/en/actions/learn-github-actions/finding-and-customizing-actions)