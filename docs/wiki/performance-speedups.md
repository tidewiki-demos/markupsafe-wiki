# Performance Speedups

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

This page describes the C extension module that provides optimized implementations of core escaping operations. The module accelerates HTML and attribute escaping by implementing character-by-character escape logic in C rather than Python.

## Overview

The `markupsafe._speedups` module is a C extension that exposes a single function, `_escape_inner`, which performs fast HTML entity escaping on Unicode strings. This function is called by the core escaping logic in [Markup Safe Core](markup-safe-core.md) when the extension is available, providing a significant performance improvement over pure Python implementations.

## How it Works

The escape operation is implemented in two phases:

1. **Delta calculation**: Scans the input string to count how many extra bytes are needed for escaped characters. This allows pre-allocation of the output buffer with the exact size needed.

2. **Escape and copy**: Iterates through the input again, copying unescaped characters in batches (via `memcpy`) and writing HTML entity references for special characters.

Special characters are escaped as follows:
- `"` (quote) → `&#34;`
- `'` (apostrophe) → `&#39;`
- `&` (ampersand) → `&amp;`
- `<` (less-than) → `&lt;`
- `>` (greater-than) → `&gt;`

If no special characters are found, the function returns the input string unchanged (with an incremented reference count).

## Unicode Support

The extension handles three Unicode internal representations [`src/markupsafe/_speedups.c:161-171`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_speedups.c#L161-L171):

- **1-byte kind** (`escape_unicode_kind1`): Used for ASCII and Latin-1 strings
- **2-byte kind** (`escape_unicode_kind2`): Used for strings with characters up to U+FFFF
- **4-byte kind** (`escape_unicode_kind4`): Used for strings with characters beyond U+FFFF

Each kind function operates on the appropriate pointer type (`Py_UCS1`, `Py_UCS2`, `Py_UCS4`) and creates an output Unicode object with matching internal representation and character width.

## Implementation Details

The core logic is implemented using C macros [`src/markupsafe/_speedups.c:3-72`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_speedups.c#L3-L72):

- `GET_DELTA`: Traverses the input, accumulating the extra bytes needed for escapes
- `DO_ESCAPE`: Performs the actual escaping by tracking runs of unescaped characters and writing escape sequences for special ones

The `escape_unicode` dispatcher function [`src/markupsafe/_speedups.c:151-171`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_speedups.c#L151-L171) checks the input type, calls `PyUnicode_READY` (for compatibility with Python versions before 3.12), determines the appropriate Unicode kind, and delegates to the specialized handler.

## Module Initialization

The module is registered as `markupsafe._speedups` [`src/markupsafe/_speedups.c:188-199`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_speedups.c#L188-L199) and exports the `_escape_inner` method [`src/markupsafe/_speedups.c:173-176`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_speedups.c#L173-L176). Modern Python versions (3.12+) are configured to support per-interpreter GIL settings, allowing better interoperability with experimental GIL-free features.

The type stub in `_speedups.pyi` [`src/markupsafe/_speedups.pyi:1`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_speedups.pyi#L1) documents the function signature for type checkers.

## Building and Testing

The extension is built as part of the standard package build process described in [Build and Packaging](build-packaging.md). When the C compiler is unavailable, the package falls back to pure Python escaping; the module is optional for correctness but important for production performance.

Performance characteristics can be measured using the [Benchmark](benchmark.md) suite. The CI pipeline described in [CI and Workflows](ci-workflows.md) runs the [Testing Suite](testing-suite.md) against both the C extension and pure Python paths to ensure compatibility.
