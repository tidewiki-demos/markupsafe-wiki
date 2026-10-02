# Development Setup

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

This page covers how to set up your local development environment for working on MarkupSafe. The setup uses dev containers for consistency, VS Code configuration for editor settings, and pre-commit hooks to maintain code quality.

## Dev Container

The project includes a [Dev Container](https://containers.dev/) configuration to standardize the development environment across machines. This ensures all developers work with the same Python version and dependencies.

### Configuration

The dev container is defined in [`.devcontainer/devcontainer.json`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/devcontainer.json). It uses the official Python dev container image from Microsoft [`.devcontainer/devcontainer.json:3`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/devcontainer.json#L3) and configures VS Code to:

- Use the virtual environment at `.venv` as the Python interpreter [`.devcontainer/devcontainer.json:7`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/devcontainer.json#L7)
- Automatically activate the environment in the terminal [`.devcontainer/devcontainer.json:8`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/devcontainer.json#L8)
- Launch Python with the `-X dev` flag to enable CPython's development mode [`.devcontainer/devcontainer.json:9-12`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/devcontainer.json#L9-L12)

### Initialization

When the container is created, it runs [`.devcontainer/on-create-command.sh`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/on-create-command.sh). This script:

1. Installs `uv` if not already present [`.devcontainer/on-create-command.sh:5-9`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/on-create-command.sh#L5-L9) — a fast Python package installer and resolver
2. Creates a virtual environment and installs all project dependencies via `uv sync` [`.devcontainer/on-create-command.sh:13`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/on-create-command.sh#L13)
3. Installs pre-commit hooks [`.devcontainer/on-create-command.sh:17`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.devcontainer/on-create-command.sh#L17)

To use the dev container in VS Code, install the Dev Containers extension and reopen the project folder in a container.

## Editor Settings

Code style is defined in [`.editorconfig`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.editorconfig), which is supported by most editors. Key settings:

- **Indentation**: 4 spaces for Python files, 2 spaces for web assets (CSS, HTML, JavaScript, JSON, YAML) [[cite:.editorconfig:4-5,12-13]]
- **Line length**: 88 characters maximum [`.editorconfig:10`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.editorconfig#L10) — this aligns with Ruff's default and the Black formatter
- **Line endings**: Unix-style LF [`.editorconfig:8`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.editorconfig#L8)
- **Encoding**: UTF-8 [`.editorconfig:9`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.editorconfig#L9)
- **Whitespace**: trailing whitespace removed, final newline added [`.editorconfig:6-7`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.editorconfig#L6-L7)

Ensure your editor is configured to respect `.editorconfig`. Most modern editors have built-in support or plugins available.

## Pre-commit Hooks

The project uses pre-commit to run checks before each commit. This catches issues early and enforces consistency.

### Setup

Pre-commit hooks are automatically installed during dev container initialization. To install them manually:

```bash
pre-commit install --install-hooks
```

### Hooks

The configuration in [`.pre-commit-config.yaml`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.pre-commit-config.yaml) runs:

- **Ruff** [`.pre-commit-config.yaml:2-6`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.pre-commit-config.yaml#L2-L6): Linter and formatter for Python code. The hooks check for issues (`ruff`) and apply formatting (`ruff-format`).
- **uv-lock** [`.pre-commit-config.yaml:7-10`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.pre-commit-config.yaml#L7-L10): Ensures the lock file is up-to-date when dependencies change.
- **Standard pre-commit hooks** [`.pre-commit-config.yaml:11-18`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/.pre-commit-config.yaml#L11-L18): Checks for merge conflicts, debug statements, byte-order markers, trailing whitespace, and missing newlines at end of file.

To run all hooks on staged files before committing, the hooks execute automatically on `git commit`. To run them manually on all files:

```bash
pre-commit run --all-files
```

## Local Setup Without Dev Container

If you prefer to set up your environment locally:

1. **Install Python**: Version 3.8 or later (check the CI configuration in [CI and Workflows](ci-workflows.md) for tested versions)
2. **Install uv**: Follow the [uv installation guide](https://docs.astral.sh/uv/getting-started/installation/)
3. **Create and activate the virtual environment**:
   ```bash
   uv sync
   ```
4. **Install pre-commit hooks**:
   ```bash
   pre-commit install --install-hooks
   ```

Your editor should automatically recognize the `.editorconfig` file and apply its settings.

## Next Steps

- Run the [Testing Suite](testing-suite.md) to verify your setup
- Review [Build and Packaging](build-packaging.md) if you plan to package or release MarkupSafe
- Check [CI and Workflows](ci-workflows.md) to understand the automated checks
