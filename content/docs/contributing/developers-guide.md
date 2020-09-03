---
title: developer's guide for pypyr
description: Handy developer & coding tips for pypyr contributors.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [contributing]
menu:
  docs:
    parent: contributing
seo_article_headline: Developer's guide for pypyr open-source contributors.
seo_description: Developer's guide for coding style, testing, code coverage and pull requests.
topics: [ custom code]
---
# developer's guide
## coding style
You've read [pep8](https://www.python.org/dev/peps/pep-0008/)? pypyr
uses flake8 as a quality gate during ci.

## testing without worrying about dependencies
Run tox to test the packaging cycle inside a tox virtual env, plus run all
tests:

```shell
# run tests & flake 8 linter
$ tox ops/build
# run tests, flake 8 linter, test packaging & validate README.rst
$ tox ops/build package
```

This of course assumes you have tox installed in your current active Python 
environment.

## If tox takes too long
For day-to-day dev, the entire testing & linting cycle via tox takes too long 
while you're still working on something. You very probably want to work in a 
virtual environment, but this is entirely up to you. 

```shell
$ python3 -m venv .env/dev
$ . .env/dev/bin/activate
$ pip install -e .[dev]
```

The `pip install -e .[dev]` command will install all the necessary dependencies 
for you.

Once you have done this, you have all the dependencies you need to dev and 
test locally and run the various tools directly without mediating through tox.

```shell
# run tests & flake 8 linter
$ pypyr ops/build
# run tests, flake 8 linter, test packaging & validate README.rst
$ pypyr ops/build package
```

Where you create your virtual environment is up to you, of course, 
but if you did want to keep in in the project directory, `./.env` is a good 
place to do it in since it's in `.gitignore` already.

## day-to-day testing
-   The test framework is [pytest](https://pytest.org).

-   Tests live under `./tests` (surprising, eh?). Mirror the directory
    structure of the code being tested.

-   Prefix a test definition with `test_` - so a unit test looks like

    ```python
    def test_this_should_totally_work():
    ```

-   To execute all tests, from root directory:

    ```shell
    $ pytest tests
    ```

-   To execute a specific test module:

    ```shell
    $ pytest tests/unit/arb_test_file.py
    ```

-   For a bit more info on running tests:

    ```shell
    $ pytest --verbose tests
    $ pytest --verbose tests/unit/arb_test_file.py
    ```

## coverage
pypyr has 100% test coverage. GitHub Actions CI enforces this if you try to 
merge with the main branch.

The standard `ops/build` pipeline runs the coverage check and outputs the 
results to terminal:

```shell
# run linting, tests + coverage with terminal output
$ pypyr ops/build
```

If the above results in less than 100%, hunt down missing lines like
this:

```shell
# display line numbers in a particular file where branch coverage missing.
# works only after report.
$ coverage report -m pypyr/mymodule.py
```

## PRs
When you pull request, code will have to pass the linting and coverage
requirements listed above. The CI enforces these, so might as well run
these locally first, eh?

So definitely do this locally before you PR:

```shell
# run tests, flake 8 linter, test packaging & validate README.rst
$ tox ops/build package
```

Try to keep the commit history tidy.

The PR description should describe the changes in it. Favor concise
bullets over paragraphs. Chances are pretty good each bullet will
coincide somewhat with each commit included in the PR. Do use previous
PRs as a guide.
