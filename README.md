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

You can now *directly* [reference container actions](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/using-pre-written-building-blocks-in-your-workflow#referencing-a-container-on-docker-hub) on GitHub Actions.
You do not *need* a separate GitHub Action.

If you need LaTeX (because you want to convert through to PDF), you should use the `docker://pandoc/latex` image.
Otherwise, the smaller `docker://pandoc/core` will suffice.

It is a good idea to be explicit about the pandoc version you require, such as `docker://pandoc/core:3.8`.
This way, any future breaking changes in pandoc will not affect your workflow.
You can find out whatever the latest released docker image is on [docker hub](https://hub.docker.com/r/pandoc/core/tags).
You should avoid specifying *no* tag or the `latest` tag -- these will float to the latest image and will expose your workflow to potentially breaking changes.


## Simple Usage

You can use pandoc inside GitHub actions exactly as you would use it on the command line.
The string passed to `args` gets appended to the [`pandoc` command](https://pandoc.org/MANUAL.html).

The below example is equivalent to running `pandoc --help`.

You can see it in action [here](https://github.com/pandoc/pandoc-action-example).

```yaml
name: Simple Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-24.04
    steps:
      - uses: docker://pandoc/core:3.8
          args: "--help" # gets appended to pandoc command
```


## Long Pandoc Calls

Remember that as per the [GitHub Actions workflow syntax](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepswithargs), "an `array of strings` is not supported by the `jobs.<job_id>.steps.with.args` parameter.
Pandoc commands can sometimes get quite long and unwieldy, but you must pass them as a *single* string.
If you want to break up the string over several lines, you can use YAML's [block chomping indicator](https://yaml.org/spec/1.2.2/#8112-block-chomping-indicator):

```yaml
name: Long Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-24.04
    steps:
      - run: echo "foo" > input.txt  # create an example file
      - uses: docker://pandoc/core:3.8
        with:
          args: >-  # allows you to break string into multiple lines
            --standalone
            --output=index.html
            input.txt
```

You can see it in action [here](https://github.com/pandoc/pandoc-action-example).


## Advanced Usage

You can also:

- create an output directory to compile into; makes it easier to deploy outputs.
- upload the output directory to [GitHub's artifact storage](https://help.github.com/en/articles/managing-a-workflow-run#downloading-logs-and-artifacts); you can quickly download the results from your GitHub Actions tab in your repo.

Remember that wildcard substitution (say, `pandoc *.md`) or other shell features frequently used with pandoc do not work inside GitHub Actions yaml files `args:` fields.
Only [GitHub Actions context and expression syntax](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions) can be used here.
If you want to make use of such shell features, you have to run that in a separate step in a `run` field and store the result in the GitHub actions context.
The below workflow includes an example of how to do this to concatenate several input files.

You can see it in action (haha!) [here](https://github.com/pandoc/pandoc-action-example).

```yaml
name: Advanced Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4

      - name: create file list
        id: files_list
        run: |
          echo "Lorem ipsum" > lorem_1.md  # create two example files
          echo "dolor sit amet" > lorem_2.md
          mkdir output  # create output dir
          # this will also include README.md
          echo "files=$(printf '"%s" ' *.md)" > $GITHUB_OUTPUT

      - uses: docker://pandoc/latex:3.8
        with:
          args: --output=output/result.pdf ${{ steps.files_list.outputs.files }}

      - uses: actions/upload-artifact@v4
        with:
          name: output
          path: output
```

## pandoc-extra and Github Actions

Because Github Actions rewrites the value of `$HOME` in runners the template directory cannot be found leading to errors such as:

```
Could not find data file templates/eisvogel.latex
```

A work around to this is to specify the exact location on the filesystem of the template:

```
- uses: docker://pandoc/extra:3.8
  with:
    args: >-  # break string of arguments into multiple lines
      README.md --output=output/README.pdf
      --standalone
      --template /.pandoc/templates/eisvogel.latex
      --listings
      -V block-headings
```

## Alternatives

If you need to have Pandoc installed and available globally, for example because it is being used in a subprocess by a library or application, you can use one of the two following alternative actions:

- [pandoc/actions/setup](https://github.com/pandoc/actions/tree/main/setup) (Linux and MacOS runners support)
- [r-lib/setup-pandoc](https://github.com/r-lib/actions/tree/v2-branch/setup-pandoc) (Linux, MacOS and Windows runners support)
