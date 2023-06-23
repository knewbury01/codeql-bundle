# CodeQL bundle

A CLI application for building a custom CodeQL bundle.

## Introduction

A CodeQL bundle is an archive containing a CodeQL CLI and compatible [standard library packs](https://github.com/github/codeql) that provides a single deployable artifact for using CodeQL locally or in a CI/CD environment.

A custom CodeQL bundle contains additional CodeQL query packs, library packs, or customization packs.
CodeQL customization packs are special CodeQL library packs that provide unofficial support to extend the CodeQL standard library packs with *sources*, *sinks*, *data-flow steps*, *barriers*, *sanitizer*, and models that describe frameworks
aimed to increase the coverage and/or precision of the CodeQL queries.

Customizations are used to add, or extend, support for open-source frameworks or proprietary frameworks.
For more details on CodeQL customization packs see the section [CodeQL customization packs](#codeql-customization-packs).

## Installation

The CodeQL bundle application can be installed using `pip` with the command:

```bash
python 3.11 -m pip install https://github.com/rvermeulen/codeql-bundle/releases/download/v0.1.5/codeql_bundle-0.1.5-py3-none-any.whl
```

## Usage

Before you can use the CodeQL bundle application you must download a bundle you want to customize from the CodeQL Action [releases](https://github.com/github/codeql-action/releases) page.

The CodeQL bundle application requires a [CodeQL workspace](https://codeql.github.com/docs/codeql-cli/about-codeql-workspaces/) to locate the packs you want to include in a custom bundle.
You can see the packs available in your workspace by running `codeql pack ls -- <dir>` where `<dir>` is the root directory of your CodeQL workspace.

With both a CodeQL bundle and a CodeQL workspace you can create a bundle with the command:

```bash
codeql-bundle --bundle <path-to-bundle> --output codeql-custom-bundle.tar.gz --workspace <path-to-workspace> --log INFO
```

## CodeQL customization packs

The CodeQL bundle CLI application provides a development experience for customization packs that mimics the development experience for official CodeQL packs.

A customization pack is a library pack with the following properties:

1. Has a module `Customizations.qll` located in the subdirectory `<scope>/<package_name>` where any character `-` in the `scope` or `package_name` is replaced with `_`.
   For example, a library pack with the name `foo/cpp-customizations` must have a `Customizations.qll` at `foo/cpp_customizations/Customizations.qll`.
1. Has a dependency on the standard library pack it wants to customize. For example, `codeql/cpp-all`.

If both properties hold, then the CodeQL bundle application will import the customizations pack `Customizations.qll` module in the standard library pack and recompile all
the CodeQL query packs in CodeQL bundle that have a dependency on that standard library pack to ensure the customizations are available to the queries.

### Steps to create a customization pack

This example targets the C/C++ language, but you can use this for any supported language.

1. Create a new pack `codeql pack init foo/cpp-customizations`.
2. Turn the pack into a library pack `sed -i '' -e 's/library: false/library: true/' cpp-customizations/qlpack.yml`.
3. Add a dependency on `codeql/cpp-all` with `codeql pack add --dir=cpp-customizations codeql/cpp-all`
4. Implement the customizations module with `mkdir -p cpp-customizations/foo/cpp_customizations && echo "import cpp" > cpp-customizations/foo/cpp_customizations/Customizations.qll`
