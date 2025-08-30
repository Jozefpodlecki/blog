+++
title = "Profiling Rust with cargo-flamegraph"
date = "2025-08-30"

[taxonomies]
tags=["rust", "flamegraph"]

[extra]
comment = true
+++

# Introduction

When optimizing Rust applications, one of the most effective tools you can use is a flamegraph. Flamegraphs help visualize where your program spends most of its time and where performance bottlenecks may be lurking.

This guide walks through setting up and running cargo flamegraph in both release and debug modes.

First, install the flamegraph cargo subcommand:

```sh
cargo install flamegraph
```

### Running Flamegraph

To generate a flamegraph in release mode (optimized build):

```sh
cargo flamegraph
```

On Windows, cargo flamegraph uses the `blondie` profiler, which requires admin rights to hook into ETW.
If you don’t run your terminal as Administrator, you’ll encounter:

```
Error: could not find dtrace and could not profile using blondie: NotAnAdmin
```

### Preserving Debug Symbols

By default, cargo flamegraph runs in release mode. To ensure debug symbols are preserved (so your stack traces are readable), add the following to your Cargo.toml:

```toml
[profile.release]
debug = true
```

Sometimes you want to analyze your program without optimizations, to better understand control flow or debugging issues. For this, you can run flamegraph in debug mode:

```sh
cargo flamegraph --dev
```

### What did not work

