# CI and Workflows

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

This repository uses GitHub Actions to automatically run tests, perform code quality checks, build packages, and publish releases. The workflows are triggered by pull requests, pushes to main branches, and git tags.

## Test Workflows

### Tests Job

[`.github/workflows/tests.yaml:1-37`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/tests.yaml#L1-L37) The `tests.yaml` workflow runs the test suite across multiple Python versions and platforms. It is triggered on pull requests and pushes to `main` and `stable` branches, but skips runs when only documentation or README files change.

The matrix includes:
- Python 3.14, 3.13, 3.12, 3.11, and 3.10 on Linux
- Free-threaded Python builds (3.14t, 3.13t) on Linux
- Python 3.14 on Windows and macOS
- PyPy 3.11

Each configuration runs tox via uv [`.github/workflows/tests.yaml:36`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/tests.yaml#L36). For free-threaded Python builds, an additional parallel test run is executed [`.github/workflows/tests.yaml:37-38`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/tests.yaml#L37-L38).

See [Testing Suite](testing-suite.md) for details on how tests are configured and executed.

### Typing Job

[`.github/workflows/tests.yaml:39-55`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/tests.yaml#L39-L55) A separate `typing` job runs mypy for static type checking. It caches the mypy cache between runs to speed up repeated checks.

## Code Quality Checks

[`.github/workflows/pre-commit.yaml`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/pre-commit.yaml) The `pre-commit.yaml` workflow runs pre-commit checks on pull requests and pushes to `main` and `stable` branches. It uses uv to install and run the pre-commit framework with all hooks defined in `.pre-commit-config.yaml`. Results are reported via the pre-commit.ci service [`.github/workflows/pre-commit.yaml:24-25`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/pre-commit.yaml#L24-L25).

## Issue Management

[`.github/workflows/lock.yaml`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/lock.yaml) The `lock.yaml` workflow runs daily on a schedule and automatically locks closed issues, pull requests, and discussions that have been inactive for 14 days. This helps reduce noise on old discussions while preserving the ability to open new issues with fresh context.

## Build and Release Workflows

### Build Jobs

[`.github/workflows/publish.yaml:16-37`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L16-L37) The `sdist` job builds a source distribution using `uv build --sdist`. It only uploads the artifact when triggered by a tag push (not during manual workflow runs for new Python versions) [`.github/workflows/publish.yaml:35-37`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L35-L37).

[`.github/workflows/publish.yaml:38-64`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L38-L64) The `wheels` job builds binary wheels across Linux, Windows, and macOS using cibuildwheel. For Linux, it sets up QEMU to build wheels for ARM64 and RISC-V architectures [`.github/workflows/publish.yaml:51-55`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L51-L55). When manually triggered with a specific Python version, it only builds wheels for that version [`.github/workflows/publish.yaml:59-60`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L59-L60).

Both jobs set the `SOURCE_DATE_EPOCH` environment variable from the last commit timestamp for reproducible builds [[cite:.github/workflows/publish.yaml:29,56]].

See [Build and Packaging](build-packaging.md) for more details on the build process.

### Release and Publishing

[`.github/workflows/publish.yaml:65-91`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L65-L91) The `create-release` job downloads all build artifacts and creates or updates a GitHub release. Tag-triggered runs create a draft release [`.github/workflows/publish.yaml:77-83`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L77-L83), while manual workflow runs update the existing release with additional files [`.github/workflows/publish.yaml:85-91`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L85-L91).

[`.github/workflows/publish.yaml:92-110`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L92-L110) The `publish-pypi` job uploads the built distributions to PyPI. It requires approval via an `environment` block before attempting upload [`.github/workflows/publish.yaml:96-98`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L96-L98), allowing time to review files in the draft release. The job uses OIDC-based authentication (Trusted Publisher model) rather than static credentials [`.github/workflows/publish.yaml:101`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.github/workflows/publish.yaml#L101).

## Workflow Triggers

- **Pre-commit**: Runs on all pull requests and pushes to `main` and `stable`
- **Tests**: Runs on all pull requests and pushes to `main` and `stable` (skipping documentation-only changes)
- **Lock**: Runs daily on a schedule
- **Publish**: Automatically triggered when a tag is pushed; can also be manually triggered to build wheels for a new Python version on an existing tag

The publish workflow's manual trigger accepts a git tag and Python version identifier, allowing new wheels to be built and uploaded without creating a new release.
