# Build and Packaging

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

MarkupSafe uses setuptools for building and modern Python packaging standards. The build configuration includes dependency management via PEP 735 dependency groups, C extension compilation for performance speedups, and multi-platform wheel building.

## Project Configuration

[`pyproject.toml:1-27`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L1-L27) defines the core project metadata. The package requires Python >=3.10 [`pyproject.toml:19`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L19) and is published to PyPI as "MarkupSafe". The project is classified as production-stable [`pyproject.toml:10`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L10) and includes type hints [`pyproject.toml:17`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L17).

## Dependency Management

[`pyproject.toml:28-57`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L28-L57) declares dependency groups using PEP 735. The default groups [`pyproject.toml:64`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L64) include `dev`, `pre-commit`, `tests`, and `typing`:

- **dev**: ruff, tox, tox-uv for development workflow
- **tests**: pytest and pytest-run-parallel (Python 3.13+) for test execution
- **typing**: mypy, pyright, and pytest for static type checking
- **docs**: Sphinx and Pallets theme for documentation building
- **docs-auto**: sphinx-autobuild for live documentation preview
- **pre-commit**: pre-commit hooks management
- **gha-update**: GitHub Actions version management (Python 3.12+)

The `uv.lock` file is included in distributions [`MANIFEST.in:2`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/MANIFEST.in#L2) and pinned dependency versions are managed via [uv](https://docs.astral.sh/uv/).

## Build System

[`pyproject.toml:59-61`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L59-L61) specifies setuptools as the build backend with version >=77. [`setup.py`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/setup.py) contains the build logic for C extensions:

- [`setup.py:12`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/setup.py#L12) defines the `markupsafe._speedups` C extension from `src/markupsafe/_speedups.c`
- [`setup.py:19-37`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/setup.py#L19-L37) implements `ve_build_ext`, a custom build command that gracefully handles C compilation failures
- [`setup.py:54-58`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/setup.py#L54-L58) checks the Python implementation; C extensions are only built for CPython, not PyPy, Jython, or GraalVM
- [`setup.py:60-82`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/setup.py#L60-L82) implements fallback logic: attempts to build with C extensions, falls back to pure Python if compilation fails (unless running under cibuildwheel), or skips extensions entirely on unsupported platforms

This design ensures the package installs successfully even when C compilation is not available, with [Performance Speedups](performance-speedups.md) enabled where possible.

## Wheel Building

[`pyproject.toml:206-222`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L206-L222) configures cibuildwheel for cross-platform binary distribution:

- [`pyproject.toml:207`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L207) enables CPython free-threading builds
- [`pyproject.toml:208`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L208) uses `build[uv]` as the build frontend for efficiency
- [`pyproject.toml:210-213`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L210-L213) overrides the frontend for musl-libc RISC-V (where uv is unavailable)
- [`pyproject.toml:215-222`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L215-L222) specifies target architectures: x86_64 and aarch64 on Linux, x86_64 and ARM64 on macOS, and auto detection plus ARM64 on Windows

Wheel building is orchestrated by [CI and Workflows](ci-workflows.md).

## Source Distribution Contents

[`MANIFEST.in`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/MANIFEST.in) controls what is included in source distributions:

- Documentation and test directories [[cite:MANIFEST.in:3,5]]
- Type stub files and py.typed marker [`MANIFEST.in:6-7`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/MANIFEST.in#L6-L7) for IDE support
- Changelog and lock file [`MANIFEST.in:1-2`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/MANIFEST.in#L1-L2)
- Exclusion of compiled bytecode [`MANIFEST.in:8`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/MANIFEST.in#L8)

## Testing and Quality Assurance

[`pyproject.toml:129-189`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L129-L189) defines tox environments for different tasks:

- `py3.10` through `py3.14`: run pytest on specified Python versions, with `*t` variants for free-threading
- `parallel`: checks for thread-safety issues using pytest-run-parallel [`pyproject.toml:152-159`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L152-L159)
- `style`: runs pre-commit hooks on all files [`pyproject.toml:161-165`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L161-L165)
- `typing`: runs mypy for type checking [`pyproject.toml:167-172`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L167-L172)
- `docs`: builds documentation with Sphinx [`pyproject.toml:174-177`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/pyproject.toml#L174-L177)
- `docs-auto`: live documentation server for development
- Update environments for keeping dependencies and CI configurations current

See [Testing Suite](testing-suite.md) for test configuration details and [CI and Workflows](ci-workflows.md) for automated quality checks.
