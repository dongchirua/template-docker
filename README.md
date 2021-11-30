# Template materials with Docker

Example repository to create a custom workring environment in Python, R, SQL.

## Try it on Binder

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/dongchirua/template-docker/master)

## Structure of the repo

This repository is based on the [binder-examples/minimal-dockerfile](https://github.com/binder-examples/minimal-dockerfile) example.

[`repo2docker`](https://repo2docker.readthedocs.io) is the underlying tool that is used to build an environment from a repository.

`repo2docker` can be configured with several types of files. In the case of this repo:

- `binder/Dockerfile`: specify the way to build up working space using [`docker`](https://mybinder.readthedocs.io/en/latest/tutorials/dockerfile.html)
- `binder/environment.yml`: specify dependencies that will be installed using `conda`
- `binder/postBuild`: specify extra dependencies such as JupyterLab extensions

Once created, the environment can be reused without building it again.

For more information, see the [extensive rep2docker documentation](https://repo2docker.readthedocs.io).

## Materials

Materials can be added anywhere to this repository, either at the top level or in subdirectory.

When building the environment, the materials (and any other file) will be copied to the Docker image.

In this example, there is already a test notebook available under `example-notebook.ipynb`.
