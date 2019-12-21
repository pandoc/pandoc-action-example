# Using `pandoc` with [GitHub Actions](https://github.com/features/actions)

<!-- badges: start -->
[![Simple Usage](https://github.com/maxheld83/pandoc-example/workflows/Simple%20Usage/badge.svg)](https://github.com/maxheld83/pandoc-example/actions)
[![Long Usage](https://github.com/maxheld83/pandoc-example/workflows/Long%20Usage/badge.svg)](https://github.com/maxheld83/pandoc-example/actions)
[![Advanced Usage](https://github.com/maxheld83/pandoc-example/workflows/Advanced%20Usage/badge.svg)](https://github.com/maxheld83/pandoc-example/actions)
<!-- badges: end -->

You can use [pandoc](https://pandoc.org/), the **universal markup converter**, on GitHub Actions to convert documents.

GitHub Actions is an Infrastructure as a Service (IaaS) from GitHub, that allows you to automatically run code on GitHub's servers on every push (or a bunch of other GitHub events).
For example, you can use GitHub Actions to convert some `file.md` to `file.pdf` (via LaTeX) and upload the results to a web host.


## Using `docker://pandoc` Images Directly

You can now *directly* [reference container actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/configuring-a-workflow#referencing-a-container-on-docker-hub) on GitHub Actions.
You do not *need* a separate GitHub Action.

If you need LaTeX (because you want to convert through to PDF), you should use the `docker://pandoc/latex` image.
Otherwise, the smaller `docker://pandoc/core` will suffice.

It is a good idea to be explicit about the pandoc version you require, such as `docker://pandoc/core:2.9`.
This way, any future breaking changes in pandoc will not affect your workflow.
You can find out whatever the latest released docker image is on [docker hub](https://hub.docker.com/r/pandoc/core/tags).
You should avoid specifying *no* tag or the `latest` tag -- these will float to the latest image and will expose your workflow to potentially breaking changes.


## Simple Usage

You can use pandoc inside GitHub actions exactly as you would use it on the command line.
The string passed to `args` gets appended to the [`pandoc` command](https://pandoc.org/MANUAL.html).

The below example is equivalent to running `pandoc --help`.

You can see it in action [here](http://github.com/maxheld83/pandoc-example).

```
name: Simple Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - uses: docker://pandoc/core:2.9
        with:
          args: "--help" # gets appended to pandoc command
```


## Long Pandoc Calls

Remember that as per the [GitHub Actions workflow syntax](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepswithargs), "an `array of strings` is not supported by the `jobs.<job_id>.steps.with.args` parameter.
Pandoc commands can sometimes get quite long and unwieldy, but you must pass them as a *single* string.
If you want to break up the string over several lines, you can use YAML's [block chomping indicator](http://www.yaml.org/spec/1.2/spec.html#id2794534):

```
name: Long Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - run: echo "foo" > input.txt  # create an example file
      - uses: docker://pandoc/core:2.9
        with:
          args: |>  # allows you to break string into multiple lines
            --standalone
            --output=index.html
            input.txt
```

You can see it in action [here](http://github.com/maxheld83/pandoc-example).


## Advanced Usage

You can also:

- create an output directory to compile into; makes it easier to deploy outputs.
- upload the output directory to [GitHub's artifact storage](https://help.github.com/en/articles/managing-a-workflow-run#downloading-logs-and-artifacts); you can quickly download the results from your GitHub Actions tab in your repo.

You can see it in action (haha!) [here](http://github.com/maxheld83/pandoc-example).

```
name: Advanced Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
      - run: mkdir output
      - uses: docker://pandoc/latex:2.9
        with: # needs a README in your repo root!
          args: "--standalone --output=output/README.pdf README.md"
      - uses: actions/upload-artifact@master
        with:
          name: output
          path: output
```
