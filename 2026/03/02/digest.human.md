Here is the Git mailing list digest for March 2, 2026:

**The day in brief.** A busy day with 145 emails across 42 threads, dominated by several major technical discussions. Key highlights include resolution of a security thread on ANSI escape sequence handling, performance regressions in the hooks subsystem, and the introduction of a new `git history split` command. Multiple patch series reached final approval stages after extensive review.

**Notable threads**

**ANSI escape sequence security reaches resolution**  
The long-running thread about securing Git's sideband channel against ANSI escape sequence injection (CVE-2024-32002, CVE-2024-52005) saw significant progress. Maintainer Junio Hamano shifted position to accept Johannes Schindelen's default-on design for real-world testing in `next`, removing a major point of contention. The series now removes problematic version-dependent defaults and delays secure-by-default behavior until Git 3.0 while maintaining backward compatibility. Production validation in Git for Windows and Red Hat environments gives confidence in the approach.

**Performance regression in hooks subsystem**  
Patrick Steinhardt identified a 2.66x slowdown in `receive-pack` operations with the reftable backend, bisected to commit fc148b146a ("receive-pack: convert update hooks to new API"). Jeff King (Peff) root-caused the issue to unnecessary sideband muxer setup/teardown when no hooks exist. Adrian Ratiu is preparing a comprehensive fix that will skip async thread creation when no hooks are present, addressing the issue across all three affected hook types. The regression highlights the careful balance needed when modernizing core infrastructure.

**New `history split` command introduced**  
Patrick Steinhardt proposed an 8-patch series adding a new `git history split` subcommand to the experimental history editing family. The command interactively splits commits while maintaining tree consistency, addressing a common rebase workflow. The implementation builds on extensive refactoring of the add-patch machinery to support in-memory index operations. With 755 lines of new tests covering edge cases, this represents a significant addition to Git's history editing capabilities when finalized.

**In brief**  

**Histogram diff bugfix** -- Yee Cheng Chin addresses an edge case where shifted change groups produce redundant diff output, with comprehensive test coverage for both triggering scenarios.

**Configurable branch comparisons** -- Harald Nordgren's multi-year effort to make `git status` branch comparisons configurable receives final test review from Junio before merging.

**Submodule remote handling** -- Bugfix series for submodule fetching from non-"origin" remotes completes review cycle with Jacob Keller's reconfirmed approval.

**Repository statistics** -- Justin Tobler's repository metrics series gets final approval after addressing edge cases in largest object tracking.

**Config-based hooks** -- Adrian Ratiu's series transitions to target 'master' after prerequisite work lands in 'next', completing the configurable hooks implementation.

**Oidmap cleanup API** -- Refactoring series converting oidmap cleanup from boolean flag to callback API completes subsystem conversions with sequencer updates.

**On the radar**  

**Partial clone config** -- Discussion continues about `fetch.blobSizeLimit` proposal, with performance implications of size-based filters under scrutiny.

**Path formatting design** -- Debate continues on optimal API design for path handling in `git repo-info`, weighing global flags against per-field modifiers.

**Windows device file handling** -- Regression in gitdir validation for Windows `nul` device files remains unresolved pending platform expertise.