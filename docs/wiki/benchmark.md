# Benchmark

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Performance benchmarking script for measuring escape operation efficiency across different implementations.

## Overview

The benchmark script [`bench.py`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py) measures the performance of the `escape` function from [Markup Safe Core](markup-safe-core.md) under various conditions. It compares the native Python implementation against the optimized [Performance Speedups](performance-speedups.md) version, testing different input patterns to identify where each implementation performs best.

## How It Works

The script uses `pyperf` to run timing measurements. For each benchmark case, it:

1. Sets up a test environment by importing `markupsafe` and the escape implementation
2. Swaps in the target `_escape_inner` function (either `native` or `speedups`)
3. Runs the `escape()` operation multiple times to gather reliable timing data

[`bench.py:11-30`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py#L11-L30) shows that measurements run for both `"native"` and `"speedups"` variants of each test case.

## Test Cases

The benchmark covers five scenarios:

- **short escape** [`bench.py:5`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py#L5): Escaping a short HTML string
- **long escape** [`bench.py:6`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py#L6): Escaping a repeated HTML string (1000 copies)
- **short plain** [`bench.py:7`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py#L7): Escaping short plain text (no HTML entities)
- **long plain** [`bench.py:8`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py#L8): Escaping long plain text (1000 copies)
- **long suffix** [`bench.py:9`](https://github.com/tidewiki-demos/markupsafe/blob/b8ed616eb769578eac1bfb0eb42316be5a39e16b/bench.py#L9): Escaping HTML with a large plain text suffix (100,000 characters)

These cases test different characteristics: small vs. large inputs, presence vs. absence of characters requiring escaping, and mixed content patterns.

## Running the Benchmark

To run the benchmark, you need `pyperf` installed (see [Development Setup](development-setup.md)). Execute:

```bash
python bench.py
```

The script will output timing results for each test case and variant, showing wall-clock time and statistical information from `pyperf`.

## Integration

The benchmark is part of the project's performance analysis toolkit and complements the [Testing Suite](testing-suite.md). Results help validate that [Performance Speedups](performance-speedups.md) provide real improvements without regressions on edge cases.
