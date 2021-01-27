# The Internals of Spark on Kubernetes Online Book

[![CI](https://github.com/jaceklaskowski/spark-kubernetes-book/workflows/ci/badge.svg)](https://github.com/jaceklaskowski/spark-kubernetes-book/actions)

The project contains the sources of [The Internals of Spark on Kubernetes](https://jaceklaskowski.github.io/spark-kubernetes-book) online book.

## Tools

The project is based on or uses the following tools:

* [Apache Spark](https://spark.apache.org/)

* [MkDocs](https://www.mkdocs.org/) which strives for being _a fast, simple and downright gorgeous static site generator that's geared towards building project documentation_

* [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme (with [Insiders](https://squidfunk.github.io/mkdocs-material/insiders/) features)

* [Markdown](https://commonmark.org/help/)

* [Visual Studio Code](https://code.visualstudio.com/) as a Markdown editor

* [Docker](https://www.docker.com/) to [run the Material for MkDocs](https://squidfunk.github.io/mkdocs-material/getting-started/#with-docker-recommended) (with extra plugins and extensions)

## Custom Docker Image

This book project uses a custom Docker image for previewing or publishing the content.

Learn more in [The Internals Online Books](https://books.japila.pl/#custom-docker-image) project.

## No Sphinx?! Why?

Read [Giving up on Read the Docs, reStructuredText and Sphinx](https://medium.com/@jaceklaskowski/giving-up-on-read-the-docs-restructuredtext-and-sphinx-674961804641).
