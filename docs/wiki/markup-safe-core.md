# Markup Safe Core

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

The Markup class and escape function form the foundation of MarkupSafe, providing safe handling of HTML/XML text and protection against injection attacks. These components work together to ensure that text marked as safe is not double-escaped, while untrusted input is properly escaped.

## The escape Function

[`src/markupsafe/__init__.py:24-45`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L24-L45)

The `escape()` function converts potentially dangerous characters to HTML-safe sequences. It escapes `&`, `<`, `>`, `'`, and `"` in strings to prevent injection attacks when displaying user-controlled text in HTML contexts.

The function has a performance optimization: if the input is already a plain `str` type, it skips the `__html__` check and goes directly to escaping. This is the common case and avoids unnecessary attribute lookups. For other types, it checks for an `__html__` method first—if present, the method's return value is trusted and used directly without escaping. This allows objects to declare their output as already safe. If there's no `__html__` method, the object is converted to a string and then escaped.

The actual character replacement happens in `_escape_inner()`, which is imported from either a C speedup module or a pure Python fallback [`src/markupsafe/__init__.py:7-10`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L7-L10). See [Performance Speedups](performance-speedups.md) for details.

## escape_silent Function

[`src/markupsafe/__init__.py:48-61`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L48-L61)

`escape_silent()` is a convenience wrapper around `escape()` that treats `None` as an empty string instead of converting it to the string `'None'`. This is useful for optional values that might be `None`.

## soft_str Function

[`src/markupsafe/__init__.py:64-81`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L64-L81)

`soft_str()` converts an object to a string without losing the safety information of a `Markup` instance. If the input is already a string (including `Markup`, which is a string subclass), it returns it as-is. This prevents double-escaping: `escape(soft_str(value))` will not escape an already-escaped `Markup` string again.

## The Markup Class

[`src/markupsafe/__init__.py:84-329`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L84-L329)

`Markup` is a string subclass that marks its content as safe for HTML/XML insertion. Unlike a plain string, a `Markup` string should not be escaped again. The key design principle is that operations on `Markup` instances return new `Markup` instances, preserving the safety marking.

### Construction and Safety

[`src/markupsafe/__init__.py:122-134`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L122-L134)

The `__new__` method checks if the input object has an `__html__` method. If it does, that method is called and its return value becomes the string content, trusting that the object knows its content is safe. Otherwise, the input is converted to a string normally without escaping.

The `__html__()` method returns `self`, implementing the HTML serialization protocol. This allows frameworks to recognize `Markup` strings as safe.

### Concatenation and String Operations

String operations like `__add__`, `__mod__`, `__mul__`, and `join()` are overridden to escape their arguments before combining them with the `Markup` string [`src/markupsafe/__init__.py:136-171`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L136-L171). For example, adding a plain string to a `Markup` string escapes the plain string first:

```python
Markup("<em>Hello</em> ") + "<foo>"  # Returns Markup('<em>Hello</em> &lt;foo&gt;')
```

The `__mod__` (percent formatting) method uses a helper class `_MarkupEscapeHelper` [`src/markupsafe/__init__.py:357-378`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L357-L378) to intercept value access and escape values as they are formatted.

### Utility Methods

`split()`, `rsplit()`, and `splitlines()` [`src/markupsafe/__init__.py:173-186`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L173-L186) return lists of `Markup` instances to preserve the safety marking of parts.

`unescape()` [`src/markupsafe/__init__.py:188-197`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L188-L197) converts HTML entities back to their characters. `striptags()` [`src/markupsafe/__init__.py:199-228`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L199-L228) removes HTML/XML tags and unescapes entities, useful for extracting plain text from markup.

### Format Methods

`format()` and `format_map()` [`src/markupsafe/__init__.py:313-323`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L313-L323) use an `EscapeFormatter` to handle string formatting while escaping unsafe values. The formatter [`src/markupsafe/__init__.py:332-354`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L332-L354) recognizes objects with `__html_format__` or `__html__` methods and passes their output through without escaping, allowing custom safe formatting.

### Case and Character Manipulation

Methods like `capitalize()`, `lower()`, `upper()`, `replace()`, `ljust()`, `center()`, and others [`src/markupsafe/__init__.py:245-301`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/__init__.py#L245-L301) wrap their results in `Markup` to preserve the safety marking. Methods that take string arguments, like `replace()` with the `new` parameter, escape those arguments first.

## Core Escaping Logic

[`src/markupsafe/_native.py:1-7`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/src/markupsafe/_native.py#L1-L7)

The pure Python implementation performs sequential character replacements to escape HTML special characters:
- `&` → `&amp;`
- `<` → `&lt;`
- `>` → `&gt;`
- `'` → `&#39;`
- `"` → `&#34;`

The order matters: `&` must be replaced first, otherwise the `&` characters in the other escape sequences would be escaped again. The C speedup version (when available) performs these replacements more efficiently.

## Design Principles

The core design ensures:
1. **No double-escaping**: A `Markup` string is never escaped again because it returns `Markup` from all operations.
2. **Trust HTML-aware objects**: Objects implementing `__html__` declare their output is safe.
3. **Escape by default**: Regular strings and unknown objects are escaped when combined with `Markup`.
4. **Type preservation**: String operations on `Markup` return `Markup`, maintaining the safety contract throughout a chain of operations.
