---
date: '2017-06-08 11:49:35'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: Common tools may be used when creating a python project
---

#### Documentation

[Sphinx](https://github.com/sphinx-doc/sphinx) is a tool that makes it easy to create intelligent and beautiful documentation. It was originally created for the Python documentation, and it has excellent facilities for the documentation of software projects in a range of languages. The following features should be highlighted:

- **Output formats:** HTML (including Windows HTML Help), LaTeX (for printable PDF versions), ePub, Texinfo, manual pages, plain text
- **Extensive cross-references:** semantic markup and automatic links for functions, classes, citations, glossary terms and similar pieces of information
- **Hierarchical structure:** easy definition of a document tree, with automatic links to siblings, parents and children
- **Automatic indices:** general index as well as a language-specific module indices
- **Code handling:** automatic highlighting using the Pygments highlighter
- **Extensions:** automatic testing of code snippets, inclusion of docstrings from Python modules (API docs), and more
- **Contributed extensions:** more than 50 extensions contributed by users in a second repository; most of them installable from PyPI

Sphinx uses reStructuredText as its markup language, and many of its strengths come from the power and straightforwardness of reStructuredText and its parsing and translating suite, the Docutils.

#### Testing

- [Coerage.py](https://bitbucket.org/ned/coveragepy) is a tool for measuring code coverage of Python programs. It monitors your program, noting which parts of the code have been executed, then analyzes the source to identify code that could have been executed but was not. Coverage measurement is typically used to gauge the effectiveness of tests. It can show which parts of your code are being exercised by tests, and which are not.

- [travis-ci](https://travis-ci.org/) is a hosted, distributed[2] continuous integration service used to build and test software projects hosted at GitHub

- [pytest](https://docs.pytest.org/en/latest/) The pytest framework makes it easy to write small tests, yet scales to support complex functional testing for applications and libraries.

#### Code styling enforcement tool

[flake8](http://flake8.pycqa.org/en/latest/) Your Tool For Style Guide Enforcement