# Documentation

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

MarkupSafe's user guides and API reference are built with Sphinx and published to Read the Docs. The documentation covers escaping behavior, the Markup class for safe text handling, string formatting with HTML-aware methods, and how to implement custom HTML representations.

## Build Configuration

Documentation is built on Read the Docs using Ubuntu 24.04 with Python 3.13. [`.readthedocs.yaml:1-10`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.readthedocs.yaml#L1-L10) The build process uses `uv` to install dependencies from the `docs` group and runs Sphinx in strict mode (`-W` flag) to treat warnings as errors. Output is generated in `dirhtml` format for better organization of multi-page documentation.

## Sphinx Configuration

The Sphinx configuration in [`docs/conf.py`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/docs/conf.py) sets up:

- **Theme and styling**: Uses the "jinja" theme from Pallets Sphinx Themes with custom sidebars and static assets (logo and favicon).
- **Extensions**: Autodoc for API documentation from docstrings, intersphinx for linking to Python's standard library documentation, and support for issue/PR links via the `extlinks` extension.
- **Documentation source**: Autodoc extracts API docs directly from source code, preserving source order and including type hints in descriptions.

## Core Documentation Topics

### Escaping and Safe Text

[`docs/escaping.rst`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/docs/escaping.rst) documents the core escaping functions and the Markup class:

- `escape()` function for converting strings to safe HTML
- `escape_silent()` for optional values
- `soft_str()` for converting objects to strings
- `Markup` class methods: `escape`, `unescape`, and `striptags`

The documentation is auto-generated from the [Markup Safe Core](markup-safe-core.md) implementation.

### String Formatting

[`docs/formatting.rst`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/docs/formatting.rst) explains how the `Markup` class supports format strings with HTML-aware behavior:

1. Objects with `__html_format__()` methods are called first, allowing custom formatting logic
2. Objects with `__html__()` methods are called as a fallback
3. Other objects use Python's default format behavior and are escaped

The guide includes a practical example of a `User` class that renders differently based on format specifiers (plain HTML vs. linked HTML). Printf-style formatting (`%` operator) also escapes values automatically.

### HTML Representations

[`docs/html.rst`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/docs/html.rst) covers the `__html__()` protocol that both MarkupSafe and many web frameworks recognize:

- Objects implementing `__html__()` return safe HTML that bypasses escaping
- This is useful for classes like `Image` that generate their own HTML structure
- Developers must manually escape user-provided data within `__html__()` to prevent injection vulnerabilities

The documentation emphasizes that `__html__()` gives full control but requires careful handling of untrusted input.

## Project Links and Context

The documentation includes links to the project repository, issue tracker, PyPI releases, and the Pallets Discord community in the sidebar. The copyright is held by Pallets (2010 onwards), and version information is automatically extracted at build time.
