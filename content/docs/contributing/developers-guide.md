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
draft: false
seo_article_headline: Developer's guide for pypyr open-source contributors.
seo_description: Developer's guide for coding style, testing, code coverage and pull requests.
topics: [ custom code]
---
# developer's guide
## coding style
You've read [pep8](https://www.python.org/dev/peps/pep-0008/)? pypyr
uses flake8 as a quality gate during ci.

## testing without worrying about dependencies
Run tox to test the packaging cycle inside a virtual env, plus run all
tests:

```bash
# just run tests
$ tox -e dev -- tests
# run tests, validate README.rst, run flake8 linter
$ tox -e stage -- tests
```

## If tox takes too long
The test framework is pytest. If you only want to run tests:

```bash
$ pip install -e .[dev,test]
```

## day-to-day testing
-   Tests live under */tests* (surprising, eh?). Mirror the directory
    structure of the code being tested.

-   Prefix a test definition with *test\_* - so a unit test looks like

    ```python
    def test_this_should_totally_work():
    ```

-   To execute tests, from root directory:

    ```bash
    $ pytest tests
    ```

-   For a bit more info on running tests:

    ```bash
    $ pytest --verbose [path]
    ```

-   To execute a specific test module:

    ```bash
    $ pytest tests/unit/arb_test_file.py
    ```

## coverage
pypyr has 100% test coverage. Shippable CI enforces this on all
branches.

```bash
# run coverage tests with terminal output
$ tox -e ci -- --cov=pypyr --cov-report term tests
```

If the above results in less than 100%, hunt down missing lines like
this:

```bash
# display line numbers in a particular file where branch coverage missing.
# works only after report.
$ coverage report -m pypyr/mymodule.py
```

## PRs
When you pull request, code will have to pass the linting and coverage
requirements listed above. The CI enforces these, so might as well run
these locally first, eh?

Try to keep the commit history tidy.

The PR description should describe the changes in it. Favor concise
bullets over paragraphs. Chances are pretty good each bullet will
coincide somewhat with each commit included in the PR. Do use previous
PRs as a guide.
