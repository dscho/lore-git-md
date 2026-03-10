Here's the daily digest for March 2, 2026:

## The day in brief

A busy day with several significant developments, including security hardening reaching consensus, performance regression fixes, and new feature proposals. The standout threads involve the resolution of a long-running security discussion around ANSI escape sequence handling and a major performance regression in the reftable backend being root-caused and fixed.

## Notable threads

**ANSI escape sequence security finalized** -- Junio Hamano has approved Johannes Schindelen's security patches for sanitizing ANSI escape sequences in Git's sideband channel (CVE-2024-32002, CVE-2024-52005). After extensive discussion and real-world testing in Git for Windows and Red Hat environments, the implementation will proceed to 'next' with default-on behavior for real-world testing. The series removes problematic version-dependent defaults while maintaining URL-scoped configuration capabilities.

**Histogram diff edge case fixed** -- Yee Cheng Chin's patch addressing redundant output in the histogram diff algorithm (XDF_HISTOGRAM_DIFF) has reached v2 with comprehensive test coverage. The fix handles cases where shifted change groups during compaction produce matching lines from both files, re-diffing affected sections using Myer's algorithm while preserving all original diff flags. All substantive technical concerns are resolved with the implementation ready for merging.

**Performance regression in reftable hooks** -- A 2.66x slowdown in `receive-pack` operations with the reftable backend was root-caused to unnecessary sideband muxer setup when no hooks exist. Jeff King identified the issue in Adrian Ratiu's hook API conversion, and Adrian is preparing a comprehensive fix that will skip async thread creation when no hooks are present. The regression specifically affected update hooks during operations with many refs.

**Configurable branch comparisons merged** -- Harald Nordgren's multi-year effort to make `git status` branch comparisons configurable has received final approval. The implementation adds `status.compareBranches` to specify which tracking branches to compare against (defaulting to just `@{upstream}` but now supporting `@{push}`). Junio has reviewed the thorough test suite and documentation, marking the culmination of 28 iterations over two months.

**Repository statistics feature approved** -- Justin Tobler's series adding repository statistics to `git repo` has been approved by Junio Hamano after addressing edge cases in the object tracking implementation. The feature identifies largest objects by type, commits with most parents, and trees with most entries, with OID annotations in the output. Performance remains stable (3-6s execution time) after final polish changes.

## In brief

**Submodule remote handling fix** -- Jacob Keller confirms approval for a bugfix addressing submodule fetching from non-"origin" remotes. The patch introduces a `get-default-remote` helper to properly handle custom remote names rather than assuming "origin".

**Hook configuration series ready** -- Adrian Ratiu's config-based hooks implementation is cleared to target 'master' after prerequisite work landed in 'next'. The series adds out-of-repo execution support and disabled hook visibility.

**Oidmap cleanup API refactored** -- A 5-patch series converts Git's oidmap cleanup from boolean flags to explicit callbacks, making memory management more type-safe. The changes touch rev-list, list-objects-filter, odb, and sequencer subsystems.

**Fetch jobs documentation clarified** -- Johannes Schindelen confirms there was no recorded discussion about leaving `fetch.parallel=0` behavior undocumented, clearing the way to document its fallback to `online_cpus()`.

**Send-email client cert support** -- A feature adding client certificate authentication to `git send-email` is approved, supporting both PEM and PKCS12 formats while maintaining existing SMTP patterns.

## On the radar

**Partial clone blob size limits** -- Discussion continues about a proposed `fetch.blobSizeLimit` config, with performance concerns raised about size-based filters versus type-based alternatives. Junio questions whether existing `remote.*.partialCloneFilter` could serve the same purpose.

**Path formatting design** -- Tian Yuchen proposes an alternative to global `--path-format` flags using per-field modifiers in `git repo-info`, prompting discussion about whether plumbing commands need such flexibility.

**Global state reduction GSoC** -- Phillip Wood's feedback about configuration handling has been incorporated into the GSoC project description, with Junio endorsing the more accurate technical guidance.