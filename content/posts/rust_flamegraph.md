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

A common pitfall is trying to run the underlying flamegraph binary directly on your program.
For example, running inside target\debug like this:

```sh
cd target\debug
flamegraph flamegraphtest.exe > test.svg
Ignored 828 lines with invalid format
```

This fails because flamegraph expects stack samples in a specific format that cargo flamegraph takes care of generating.

You can check where the script is installed:

```powershell
Get-Command flamegraph
# Application     flamegraph.bat 0.0.0.0    C:\Strawberry\perl\bin\flamegraph.bat
```

### What the script does (step by step)

#### Initial Setup & Options

- Uses Getopt::Long to handle command-line options (--width, --height, --title, etc.).
- Default settings: width 1200px, frame height 16px, font Verdana, etc.
- Allows customization of colors, fonts, differential mode, inverted graphs, etc.

#### Reading Input

- Reads stack lines from STDIN or file.
- Can reverse stack order (--reverse) for icicle-style graphs.
- Parses each line into a stack array and count, skipping invalid lines.

#### Merging Stacks (flow function)

- Compares current stack with previous stack to merge repeated frames.
- Maintains a %Node hash storing start time (stime) and delta for each node.
- Essentially builds a tree of stack frames with associated sample counts.

#### Compute Graph Dimensions

- Calculates width_per_sample and frame height.
- Prunes tiny frames (minwidth) that would be too small to render.

#### Generate SVG
- Uses a small internal SVG package to construct the SVG.
- Adds background gradient, title, and labels.
- Iterates over %Node and draws rectangles for each stack frame.
- Computes color for each frame based on:
- Hot (default)
- Memory / IO / Java / JS / other color palettes
- Differential coloring if needed
- Optionally consistent colors via a palette file

#### Text & Interaction

- Adds function names inside rectangles (truncates long names).
- Embeds interactive JavaScript:
- Hover: shows details of stack frame
- Click: zoom into function
- Search: highlight functions by regex
-  Reset zoom

#### Palette Management

- If --cp is used, reads/writes a palette map to keep function colors consistent between runs.

### Error Handling

- If invalid lines are detected: Ignored N lines with invalid format
- If no valid input: prints a small error SVG.