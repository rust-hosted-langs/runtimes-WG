# Runtimes in Rust working group

[![Gitter](https://badges.gitter.im/rust-hosted-langs/runtimes-WG.svg)](https://gitter.im/rust-hosted-langs/runtimes-WG?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## Purposes
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

## Contributing

We abide by the [Rust Code of Conduct](https://www.rust-lang.org/en-US/conduct.html)
and ask that you do as well.

Presently we are a a group whose shared interests are implementing programming
languages and doing so in Rust. If that describes you, feel free to join us
in the Gitter channel linked above for general language and implementation
conversation.

If you have implemented a language in Rust, or are in the process of doing so,
we'd love to hear about it!

### Adding your language to our directory

See the [survey ticket](https://github.com/rust-hosted-langs/runtimes-WG/issues/1)
for an list of sample questions to consider. Fork this repository, create a new file
`lib/<lang-name>.md` and submit a pull request.


## Initial actions
- Fundamentally, memory management and concurrency are where the most critical
  tradeoffs are made in runtime design. This is where implementation and APIs
  in Rust most carefully requires unsafe code and safe abstraction design.
  There have been plenty of forays into implementing languages in Rust already,
  we should learn from each other.
- [Survey of Rust prior art](https://github.com/rust-hosted-langs/runtimes-WG/issues/1)
- [Survey of non-Rust languages and runtimes](https://github.com/rust-hosted-langs/runtimes-WG/issues/5)

## Long term goals

These goals are broad desires and probably don't reflect actual outcomes.

- Mature implementations of a variety of memory management runtimes wrapped in safe APIs
  - well documented
  - documenting where safe abstraction is not possible without unreasonable trade-offs
  - The ideal result is a common set of traits that could be implemented by
    allocators, GCs and managed data structures such that a change in a GC
    strategy could be as simple as swapping in a different crate and recompiling.
- A language and runtime that is to Rust as CPython is to C
  - Deliberately creating a scripting/glue language is strategic.
    Languages such as Python, Ruby and PHP proliferate quickly into every
    domain and where they go, so goes their host language.
    This goal forms an important long-term part of the Rust ecosystem.
  - familiar, low friction, broad appeal, a new gateway to using Rust for many
  - an accessible implementation
  - low friction to implementing new and wrapping existing libraries for it in Rust
  - doesn’t compromise the memory safety of the host language
  - This may be one or more existing projects that can be given momentum!
