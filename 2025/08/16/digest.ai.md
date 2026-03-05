# Git Mailing List Digest — 2025/08/16

**The day in brief.** A moderately active Saturday with 28 emails across 10 threads saw the final polishing of the new `git repo info` command series, continued discussion of Rust platform compatibility, and a comprehensive submodule path encoding series. The standout technical contributions were the submodule gitdir path encoding work and the `git repo info` documentation refinements.

## Notable threads

### `git repo info` command reaches final form

The long-running effort to create a dedicated `git repo info` command for repository metadata queries reached its final documentation and test refinement stage today. The v11 series, now approved for merging, saw multiple patches addressing Junio Hamano's feedback on documentation formatting and test organization. Key improvements include standardized test output file naming, proper definition list syntax in documentation, and consolidated test repository initialization. The command will provide structured access to metadata like reference storage type (`references.format`), repository layout (`layout.bare`, `layout.shallow`), with both human-readable (`keyvalue`) and machine-parsable (`nul`) output formats. This represents the culmination of a GSoC project to migrate repository metadata functionality from `git rev-parse` to a more maintainable home.

### Submodule gitdir path encoding series

A comprehensive 9-patch series addressing submodule gitdir naming conflicts was introduced today, building on 2018 work by Brandon Williams. The series implements URL-style encoding (RFC 3986) for submodule gitdir paths to prevent filesystem conflicts, with special handling for case-insensitive systems and Windows reserved names. The implementation moves submodule gitdirs from `.git/modules/` to `.git/submodules/` by default while maintaining backward compatibility, includes a config override mechanism (`submodule.<name>.gitdirpath`), and adds extensive test coverage for mixed legacy/new-style paths. The work touches core submodule handling across multiple files and includes 207 lines of new tests, representing a significant improvement in submodule path safety.

### Rust platform compatibility discussion continues

The ongoing debate about Rust dependency platform requirements saw new input today regarding Windows version support. Matthias Aßhauer provided historical context about Git for Windows' extended support policies beyond Microsoft's official EOL dates, noting that using `x86_64-pc-windows-gnu` would inadvertently drop Windows 8.1 support while `*-win7-windows-gnu` would maintain compatibility. The discussion highlights the tension between core Git's potential alignment with vendor support lifecycles and Git for Windows' historical commitment to extended support. This remains an unresolved debate in the Rust transition, with platform support tradeoffs still being weighed.

## In brief

**CMake build fix for reftable tests** — Johannes Schindelin submitted a follow-up fix replacing obsolete `UNIT_TEST_PROGRAMS` variable parsing with wildcard pattern matching (`t-*.c`) to maintain CMake build compatibility after the reftable test migration.

**Documentation philosophy clarification** — D. Ben Knoble and Junio Hamano aligned on documentation improvement principles, emphasizing adapting documentation to current needs rather than replicating historical approaches.

**`git stash --only-unstaged` workaround** — Phillip Wood contributed an interim shell script solution demonstrating how to safely restore only unstaged changes from a `--keep-index` stash using three-way merging, potentially informing the eventual built-in solution.

**Test fix for installed Git binaries** — A minor test suite adjustment was made to account for additional git-gui commands that appear when tests are run with `GIT_TEST_INSTALLED`, maintaining test reliability across build configurations.

## On the radar

**Git alias security discussion** — While the thread about alias behavior limitations was officially closed, follow-up emails today continued probing Git's security model regarding repository trust boundaries and command shadowing, suggesting areas where documentation could be enhanced.