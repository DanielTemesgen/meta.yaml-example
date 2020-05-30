# meta.yaml-example
A way of dealing with the joy of maintaining user and dev environments with conda.

## Justification

Packaging in Python can be difficult in in the best of times.
There aren't fixed standards, conda goes a long way but falls down with respect to pip compatibility.

### Problem 1: No easy way to split Run and Dev environments

One consistent annoyance, is the lack of separation between the following two environments:
1. The run environment
  * what your users need to install to use your package.
  * e.g. numPy, matplotlib
1. The developer environment
  * what the developers of the package need to install to develop the software.
  * this will usually be a superset of the run environment, with extra packages for testing, documentation deployment etc.
  * e.g. numpPy, matplotlib **and** pytest, sphinx etc.
  
If you're packaging a project with conda then you'll most likely be using a `meta.yaml` file.
Where you'll be declaring your dependencies in three separate sections:
1. Run
1. Build
1. Test

```
{% set version = "1.1.0" %}

package:
  name: imagesize
  version: {{ version }}

source:
  url: https://pypi.io/packages/source/i/imagesize/imagesize-{{ version }}.tar.gz
  sha256: f3832918bc3c66617f92e35f5d70729187676313caa60c187eb0f28b8fe5e3b5

build:
  noarch: python
  number: 0
  script: python -m pip install --no-deps --ignore-installed .

requirements:
  host:
    - python
    - pip
  run:
    - python

test:
  imports:
    - imagesize

about:
  home: https://github.com/shibukawa/imagesize_py
  license: MIT
  summary: 'Getting image size from png/jpeg/jpeg2000/gif file'
  description: |
    This module analyzes jpeg/jpeg2000/png/gif image header and
    return image size.
  dev_url: https://github.com/shibukawa/imagesize_py
  doc_url: https://pypi.python.org/pypi/imagesize
  doc_source_url: https://github.com/shibukawa/imagesize_py/blob/master/README.rst

```

Often, the Build and Test sections will be identical.

So it can be somewhat frustrating having to copy these dependencies multiple times.

## Problem 2: Cannot make the developer environment directly from a `meta.yaml`
The conda-build process will create the developer environment but it can be difficult to get it setup locally from this file.
This may require your dev environment file to be declared in two different locations. <br>
The `meta.yaml` and a `requirments.txt`, keeping those two files in sync is an unnecessary overhead..

**There must be a better way**

## Possible Solution

### How it works
Thanks to this helpful [GitHub issue](https://github.com/conda/conda/issues/6788) it became apparent that `meta.yaml` files accept Jinja templating and this can be used to feed in requirements at build time. <br>
I've extended this idea to take care of Problem 1 and 2 above.

Put simply we can define three separate files **once**:
1. conda_run_requirements.txt
  * The conda packages needed to run your package.
1. conda_dev_requirements.txt
  * The **additional** conda packages needed to develop your project.
1. pip_dev_requirements.txt
  * The **additional** pip-only packages that are required to develop your project.
  * Whilst these cannot be used in conjunction with the `meta.yaml` file users can use this to get the full dev requirements if they want to install packages for another part of their CI which can use pip e.g. documentation build.
  
An example of this this can be found in the meta.yaml file for this page. <br>
The Run section has:
* conda_run_requirements
The Build section has:
* conda_run_requirements
* conda_dev_requirements
The Test section has:
* conda_run_requirements
* conda_dev_requirements

### How to use this
The `meta.yaml` file reads in the files on build time.

If a user wants to develop the package they need only to `conda install` the `conda_run_requirements.txt` and `conda_dev_requirements.txt`, then pip install `pip_dev_requirements.txt`.

Requirements are only defined once and can be used locally **and** on package build.




