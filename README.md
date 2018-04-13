# Runtimes in Rust working group

[![Gitter](https://badges.gitter.im/rust-hosted-langs/runtimes-WG.svg)](https://gitter.im/rust-hosted-langs/runtimes-WG?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## WG summary
- Discuss runtime implementations
- Build a library of articles and papers relevant to implementing runtimes
  in Rust
- A survey of the landscape: low-level descriptions of existing languages
  hosted in Rust
- Explore common safe-Rust abstractions for a reasonable variety of memory
  management approaches
- Track other core ecosystem components and features, such as futures,
  tokio, await/async, ensuring good integration
- To make Rust an attractive option for building a language in

## Initial agenda
- Fundamentally, memory management and concurrency are where the most critical
  tradeoffs are made in runtime design. This is where implementation and APIs
  in Rust will most carefully require unsafe code and safe abstraction design.
- Survey of Rust prior art looking for
  - memory management approaches
  - concurrency models
  - code ergonomics
  - API safety
  - Possible Rust-lang obstacles
- Survey of non-Rust language runtimes
  - CPython, micropython, pypy/rpython
  - Ruby MRI, Rubinius
  - Lua 5, LuaJIT
  - Go
  - Pony
  - JVM
  - Erlang, Elixir, BEAM

### Prior art in Rust

(inexhaustive lists)

#### Languages, runtimes, libraries

- https://gitlab.com/yorickpeterse/inko
- https://github.com/PistonDevelopers/dyon
- https://github.com/gluon-lang/gluon
- https://github.com/murarth/ketos
- https://github.com/jonathandturner/rhai
- https://github.com/jorendorff/cell-gc
- https://github.com/Manishearth/rust-gc/
- https://github.com/cipriancraciun/vonuvoli-scheme

#### Garbage collection posts and papers

- https://manishearth.github.io/blog/2015/09/01/designing-a-gc-in-rust/
- https://manishearth.github.io/blog/2016/08/18/gc-support-in-rust-api-design/
- http://blog.pnkfx.org/blog/2015/10/27/gc-and-rust-part-0-how-does-gc-work/
- http://blog.pnkfx.org/blog/2015/11/10/gc-and-rust-part-1-specing-the-problem/
- http://blog.pnkfx.org/blog/2016/01/01/gc-and-rust-part-2-roots-of-the-problem/
- [Rust as a language for high performance GC implementation](http://users.cecs.anu.edu.au/~steveb/downloads/pdf/rust-ismm-2016.pdf)

### Possible Rust-lang obstacles
- How to handle drop-unsafety in garbage collection.
  - when object collection is non-determinstically ordered, `impl Drop` is inherntly unsafe
  - see rust-gc for one workaround, see cell-gc for alternative advice

## Long term goals

These goals are painted with broad strokes and need to be turned into specifications.

- A language and runtime that is to Rust as CPython is to C
  - Deliberately creating a scripting/glue language is strategic.
    Languages such as Python, Ruby and PHP proliferate quickly into every
    domain and where they go, so goes their host language.
    This goal forms an important long-term part of the Rust ecosystem.
  - familiar, low friction, broad appeal, a new gateway to using Rust for many
  - an accessible implementation
  - low friction to implementing new and wrapping existing libraries for it in Rust
  - doesn’t compromise the memory safety of the host language
- Mature implementations of a variety of memory management runtimes wrapped in safe APIs
  - well documented
  - documenting where safe abstraction is not possible without unreasonable trade-offs
  - The ideal result is a common set of traits that could be implemented by
    allocators, GCs and managed data structures such that a change in a GC
    strategy could be as simple as swapping in a different crate and recompiling.
