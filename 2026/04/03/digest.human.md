Here's the daily digest for April 3, 2026:

## The day in brief

A moderately busy day with 59 emails across 21 threads, featuring significant progress on multiple fronts. Key developments include Patrick Steinhardt's in-memory ODB backend series nearing completion, Rust becoming enabled by default in the build system, and ongoing refinements to graph visualization and test infrastructure. The `the_repository` removal effort saw productive iteration while several smaller features and fixes moved toward completion.

## Notable threads

### Graph visualization improvements take shape

Pablo Sabater's RFC series to improve `git log --graph` output for root commits and excluded parents saw substantive discussion about the visual design. Junio C Hamano clarified that the solution must handle both true root commits and commits with parents outside the displayed range ("seems_root" concept). Tian Yuchen proposed using distinct symbols (■) for root commits, but Junio noted this approach was previously rejected due to complexity with `--left-right` and `--boundary` outputs. The thread also surfaced a test case portability issue (case-sensitive filenames) that Pablo committed to fixing in v2.

### In-memory ODB backend advances

Patrick Steinhardt's 16-patch series introducing an in-memory object database source received its first maintainer feedback after completion. Junio acknowledged the technical implementation while clarifying the intended use case for truly transient objects. The series systematically implements all required ODB callbacks for a backend that stores objects purely in memory, building on Patrick's ongoing ODB abstraction work. Junio noted potential coordination with Justin Tobler's parallel work on ODB read interfaces but raised no substantive technical concerns.

### Rust enabled by default

brian m. carlson's 4-patch series to enable Rust support by default in Git 2.54 (after missing the planned 2.53 target) was approved by Junio. The changes include documentation updates, CI fixes for Alpine Linux compatibility, Linux linker flag adjustments for libdl, and finally flipping the default build flag. The series maintains backward compatibility by keeping two CI jobs that build without Rust, addressing platform-specific requirements while implementing a previously agreed-upon change.

### `the_repository` removal continues

Shreyansh Paliwal's refactoring series to remove `the_repository` usage from the refs subsystem progressed with a narrowed focus. After discovering edge cases in repository validity checking, the author split off the problematic branch helper conversions for future work. The remaining patches (converting ref lock timeouts, hash algorithm usage, and reftable backend) received positive feedback, with Patrick Steinhardt noting he has related work coming for remaining `is_bare_repository()` calls. Tian Yuchen suggested clarifying documentation about a remaining static variable, but the changes appear ready to proceed.

## In brief

**Promisor-remote test improvements** -- Christian Couder finalized his promisor-remote refactoring series by addressing Junio's feedback on test URI handling, renaming variables for clarity and expanding special character encoding.

**Submodule fetch error configuration** -- A new series allows configuring whether submodule fetch failures should be fatal ("fail", default) or just produce warnings ("warn"), useful when working with upstream branches referencing unpublished submodule commits.

**Worktree path config conditions** -- Chen Linxuan's v3 series adds `includeIf` conditions for matching worktree paths (`worktree` and `worktree/i`), improving config inclusion in multi-worktree setups where `gitdir` patterns are awkward.

**Config file locking timeout** -- Jörg Thalheim fixed concurrent config file access issues by making Git retry lock acquisition for 100ms (matching reference lock behavior), preventing silent failures in operations like `git worktree add -b`.

**Git for Windows 2.54.0-rc0** -- Johannes Schindelin announced the release candidate, notable for removing `git svn` from the distribution due to maintenance challenges while updating bundled components.

**Windows build improvements** -- Schindelin also contributed patches for UCRT64 environment preparation and CMake `writev()` support, continuing Git for Windows' modernization efforts.

## On the radar

**Rustification platform concerns** -- While Rust is now enabled by default, Randall S. Becker's concerns about NonStop platform support remain unresolved long-term.

**Test modernization for bare repos** -- Johannes Schindelin's series to prepare tests for potential `safe.bareRepository=explicit` behavior awaits documentation updates and minor test refinements.

**ODB interface alignment** -- Patrick Steinhardt's in-memory ODB work may need coordination with Justin Tobler's parallel changes to ODB read interfaces as both series progress.