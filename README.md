# ravenc · incremental build explorer

A prototype interface for exploring an incremental build as a graph instead of a log.

Pick a change, and the graph recolours itself: which build units were rebuilt, which
were reused, which were rebuilt but produced byte-identical output, and which only moved.

![three panes: change picker, unit graph, unit detail](https://clpi.github.io/incremental-build-explorer/)

## What it shows

- **Unit graph** — 200 units across the ravenc pipeline (core → runtime → syntax → hir →
  typeck → mir → borrowck → optimize → mono → regalloc → codegen → link → driver →
  suites → docs), with dependency edges and reuse/rebuild colouring per change.
- **Change picker** — 7 canned scenarios, from a one-line comment fix (1 unit rebuilt) to
  a public definition change (134 rebuilt) and a toolchain relocation (200 rebuilt,
  0 content changes, 0 time saved).
- **Unit detail** — inputs with hash before → after, outputs with hash before → after and
  relocated paths, cost, schedule slot, downstream impact, and the exact reuse reason
  (`inputs unchanged`, `upstream rebuilt, interface fingerprint clean`,
  `output identical after rebuild`, `interface fingerprint changed: …`).
- **Header** — rebuilt vs reused counts, serial rebuild cost, incremental wall clock at
  `-j8`, and the wall clock reuse saved.

## How the model works

`model.js` builds the graph deterministically (same graph, hashes and timings every load).
A unit is rebuilt only when one of its input hashes moves: its own sources, the interface
fingerprint of a dependency, or the resolved build flags. Wall clock is a greedy list
schedule of the rebuilt units on the real dependency graph with 8 workers.

**The data is synthetic.** Timings, hashes and reuse verdicts are generated to demonstrate
the view; they are not measurements of a real build. The page says so in the header.

## Running it

Single self-contained file, no build step, no dependencies: open `index.html`.
