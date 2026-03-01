# Git Mailing List Digest — 2025/01/27

**The day in brief.** A busy Monday with 88 emails across 19 threads, dominated by two major technical efforts: Christian Couder's promisor-remote/LOP series reaching v4 with significant protocol and documentation refinements, and Patrick Steinhardt's 19-patch series decoupling the reftable library from Git's core infrastructure. Notable progress also on Derrick Stolee's name-hashing optimization and several platform-specific fixes.

## Notable threads

### Promisor-remote/LOP protocol v4

Christian Couder's Large Object Promisor (LOP) series reached its fourth iteration with substantial refinements to both protocol implementation and documentation. The core capability allows servers to advertise promisor remotes to clients via protocol v2, controlled by new `promisor.advertise` (server) and `promisor.acceptFromServer` (client) configs. Junio Hamano's review focused on security boundaries and test isolation, particularly around MIDX interactions that required temporary test disabling. The design document now better positions LOPs as a Git-LFS alternative while acknowledging areas for future work like lazy fetch security. The series appears technically stable but faces process questions about shared foundation patches with Usman Akinyemi's parallel work.

### Reftable library independence

Patrick Steinhardt's 19-patch series systematically removes Git dependencies from the reftable library, replacing Git-specific helpers with direct POSIX calls or reftable-specific implementations. Changes span I/O operations (`read_in_full` → direct `read`), error handling (`BUG()` → error returns), platform compatibility (new `compat/posix.h`), and build system adjustments. The work enables reftable to be used independently by projects like libgit2. Junio raised long-term maintenance concerns about parallel implementations in sync, suggesting this architectural tradeoff between independence and maintainability warrants further discussion.

### Name-hashing optimization

Derrick Stolee's performance optimization series introduced configurable name hashing for pack-objects delta compression via `--name-hash-version`. Version 2's directory-aware algorithm shows dramatic improvements in pathological cases (37GB → 7GB in one private repo). The series includes comprehensive testing infrastructure and safety checks, with only minor documentation nits remaining. Junio confirmed he'll apply the typo-fixed version, marking this mature optimization ready for integration.

### Windows reftable compatibility

Ongoing discussion about Windows file handling in the reftable backend focused on Patrick Steinhardt's approach to file-in-use errors. His solution immediately fails rather than retrying when external processes (like JGit) hold locks, arguing retries would rarely succeed. The thread revealed architectural tensions about whether to modify platform-layer `unlink()` or adapt reftable code, with Windows maintainer Johannes Sixt proposing alternative approaches. Christian Reich confirmed successful testing of Steinhardt's patch in custom Git for Windows builds.

### Bare repository fetch regression

Bence Ferdinandy's fix for bare repository fetch behavior (regressed in 2.48.0) received final approval from both Patrick Steinhardt and Junio Hamano. The solution properly distinguishes mirror vs non-mirror remote behavior, ensuring HEAD references are overwritten only for true mirrors while creating proper remote-tracking HEADs otherwise. Thorough test coverage in t5505 and t5510 validated the fix, which now awaits integration.

## In brief

**OS version capability design** Christian Couder argued against merging with "agent" string, citing web browser User-Agent pitfalls; Junio remained unconvinced, favoring protocol simplicity. **Shallow clone tag behavior** Analysis revealed prefix ordering affects protocol-dependent fetching; fix works but underlying cause remains unclear. **zlib-ng compatibility** Justin Tobler reviewed constness fixes, suggesting better documentation for non-obvious casts. **CI modernization** Patrick Steinhardt's series completed review with all feedback addressed, ready for integration. **MSVC warning fixes** Sören Krecker's type safety work drew architectural questions about whether `size_t` conversions are appropriate for line numbers. **Refspec reorganization** Meet Soni's series centralizing refspec logic in refspec.c received detailed API quality feedback from Junio.

## On the radar

The promisor-remote/LOP series' shared foundation patches with Usman Akinyemi's work may need coordination to avoid future rebase conflicts. Junio's concern about long-term maintenance of reftable's parallel implementations remains an open architectural question worth watching as the series progresses.