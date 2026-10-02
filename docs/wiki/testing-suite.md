# Testing Suite

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

The testing suite provides comprehensive coverage for escape operations, Markup behavior, exception handling, and memory leak detection. Tests run against both the native Python implementation and optional C speedups.

## Test Infrastructure

[`tests/conftest.py:18-39`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/conftest.py#L18-L39) The test configuration uses pytest fixtures to parametrize tests across implementation variants. The `_mod` fixture swaps between the native `_escape_inner` and the optional `_speedups` module at session scope, allowing the same test suite to verify both implementations work identically. Tests gracefully skip speedups-specific checks when the C extension is unavailable.

[`tests/conftest.py:20-21`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/conftest.py#L20-L21) The report header indicates whether Python 3.13's free-threaded mode is active, helping identify platform-specific behavior.

## Escape Operations

[`tests/test_escape.py:11-34`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_escape.py#L11-L34) The `test_escape` function parametrizes escape behavior across character encodings and special characters:
- Empty strings and ASCII text
- Multi-byte UTF-8 (2-byte Japanese characters, 4-byte emoji)
- Proper escaping of `&`, `>`, `<`, `'`, and `"` characters

Edge cases handle proxy objects and string subclasses: [`tests/test_escape.py:37-68`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_escape.py#L37-L68) The `Proxy` class tests objects that disguise their `__class__` to fool `isinstance` checks, while `ReferenceStr` tests `str` subclasses where `str(o)` does not return a plain `str`. Both must escape correctly without type confusion.

## Markup Behavior

[`tests/test_markupsafe.py`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py) The main test module covers Markup's core features:

- **String operations** [`tests/test_markupsafe.py:13-16`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L13-L16): Adding unsafe strings to Markup maintains proper escaping.
- **String interpolation** [`tests/test_markupsafe.py:19-33`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L19-L33): Both `%` formatting (positional and named) and numeric formatting escape untrusted data while preserving Markup values.
- **Type preservation** [`tests/test_markupsafe.py:36-39`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L36-L39): Operations on Markup return Markup, and `__html__()` returns itself.
- **HTML interoperability** [`tests/test_markupsafe.py:42-52`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L42-L52): Objects implementing `__html__()` are respected, and format operations on them work correctly.
- **Format methods** [`tests/test_markupsafe.py:107-141`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L107-L141): `format()` and `format_map()` escape values unless they are Markup or have `__html_format__()` methods.
- **String methods** [`tests/test_markupsafe.py:183-187`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L183-L187): `split()`, `rsplit()`, and `splitlines()` return lists of Markup.
- **HTML utilities** [`tests/test_markupsafe.py:77-104`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_markupsafe.py#L77-L104): `striptags()` removes HTML comments and tags; `unescape()` decodes HTML entities.

## Exception Handling

[`tests/test_exception_custom_html.py`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_exception_custom_html.py) The exception propagation test ensures that if an object's `__html__()` method raises an exception, that exception is propagated correctly to the caller. This addresses a past bug where exceptions were sometimes swallowed.

## Extension Module Initialization

[`tests/test_ext_init.py`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_ext_init.py) The multi-phase initialization test verifies that the C extension module properly supports being re-imported: after clearing `sys.modules`, a new import produces a different module object with a different `_escape_inner` function. This ensures the extension is not cached in a way that breaks module reloading.

## Memory Leak Detection

[`tests/test_leak.py:10-28`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/tests/test_leak.py#L10-L28) The `test_markup_leaks` test detects memory leaks by calling `escape()` in nested loops and tracking the number of objects tracked by the garbage collector. After each inner loop iteration, the object count is recorded. A stable count (or at most 2–3 different counts as implementations stabilize) indicates no leak. A leak would show different counts across each iteration. This test is marked `thread_unsafe` because it calls `gc.get_objects()`, which is unsafe in multi-threaded environments.

## Test Markers and Safety

Tests use `@pytest.mark.thread_unsafe` to indicate which tests cannot run safely in threaded contexts (those calling `gc.get_objects()` or tampering with `sys.modules`). The `skipif` marker conditionally skips tests when speedups are not available, allowing the suite to run on all platforms.
