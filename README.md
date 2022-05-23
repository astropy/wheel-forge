## A pipeline for building wheels for Astropy coordinated packages

[![Build Status](https://dev.azure.com/astropy-project/wheel-forge/_apis/build/status/astropy.wheel-forge?branchName=master)](https://dev.azure.com/astropy-project/wheel-forge/_build/latest?definitionId=1&branchName=master)

### 📣 Please note

This repository is now archived and no longer actively used. Wheels are now built as part of the continuous integration of each package.

### TL;DR

If you are responsible for a coordinated package and have released a
new version, go to the [Azure build](https://dev.azure.com/astropy-project/wheel-forge/_build/) and click **Queue** then **Queue** again in the dialog (or ask someone who has permissions to do so). Once the builds are complete, run:

    python get_wheels.py

and upload the wheels for your package with twine.

### Overview

The aim of this repository is to provide an easy way to automatically
build wheels for Astropy coordinates packages. This makes use of the
[autowheel](https://github.com/astrofrog/autowheel) package which in
turn uses the [cibuildwheel](https://github.com/joerick/cibuildwheel)
package. The autowheel package checks PyPI to see which wheels are
missing for a list of packages defined in ``autowheel.yml`` (see below)
and if there are missing wheels, it will download the source from
PyPI then use cibuildwheel to actually build the wheels.

The autowheel package can also automatically figure out the oldest
compatible version Numpy that can be used to build wheels for each
Python version - this is important since wheels should be built on
the oldest compatible Numpy version possible to then work on any
more recent Numpy version.

### Configuration

The basic configuration regarding which packages should be built is contained in the [autowheel.yml](https://github.com/astropy/wheel-forge/blob/master/autowheel.yml) file. The basic syntax for this is documented in the
[README for autowheel](https://github.com/astrofrog/autowheel/blob/master/README.rst). Essentially this file should only need to be
updated if:

* A new Python version becomes available
* The minimum version for a package changes
* The minimum version of Numpy for a package changes and is more recent than the oldest version of Numpy compatible with a given Python version
* A package needs to be removed or added

Otherwise, the configuration file does not need to be updated solely
because a new version of a package is available (as described in the
autowheel docs).

### Background on building

The building happens on Azure pipelines and is controlled by the [azure-pipelines.yml](https://github.com/astropy/wheel-forge/blob/master/azure-pipelines.yml) and [azure-template.yml](https://github.com/astropy/wheel-forge/blob/master/azure-template.yml) file. These should not need to be changed unless there are changes in the way Azure Pipelines operates.
The builds can be found [here](https://dev.azure.com/astropy-project/wheel-forge/_build/). For now, triggering builds needs to be done manually - to do this, go to the previous link for the Azure build and click on 'Queue'.

### Getting wheels and uploading to PyPI

Once the wheels have been built, they are uploaded as artifacts in Azure Pipelines. The easiest way to get these is to use the [get_wheels.py](https://github.com/astropy/wheel-forge/blob/master/get_wheels.yml)
script provided:

    python get_wheels.py

This will create a ``wheelhouse`` directory containing all the wheels that have been built. At this point, you can then choose to upload
these to PyPI if you wish, using
[twine](https://pypi.org/project/twine/).  E.g.:

    twine upload wheelhouse/<pkgname>*.whl



### Further information

Note that in the master branch, only wheels that are missing from PyPI are built, while in pull requests, all wheels are built (to make sure
that everything is working correctly). This is controlled by the ``--build-existing/--no-build-existing`` flags inside [azure-template.yml](https://github.com/astropy/wheel-forge/blob/master/azure-template.yml).

Currently we don't build wheels for Python 2.7 because the way
cibuildwheel runs tests causes issues with pytest plugins. Since
Python 2.7 support will shortly be dropped from coordinated packages,
this will likely not be fixed.

When adding a package to [autowheel.yml](https://github.com/astropy/wheel-forge/blob/master/autowheel.yml), be sure to also add it in the matrix in
[azure-template.yml](https://github.com/astropy/wheel-forge/blob/master/azure-template.yml).
