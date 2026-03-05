# Git Mailing List Digest - 2025/08/30

**The day in brief.** A moderately active day with 16 emails across 8 threads, dominated by technical discussions around midx-write bugfixes and error handling improvements. The most notable developments include Derrick Stolee's v2 series fixing critical MIDX segfaults and Jeff King's proposal for smoother deprecation of `git whatchanged`. Several long-running threads saw resolution, including Patrick Steinhardt's ODB source system integration and the test reorganization effort.

## Notable threads

**MIDX segfault fixes and error handling improvements** -- Derrick Stolee submitted a comprehensive v2 series (6 patches) addressing memory safety issues in midx-write.c that could cause segfaults during `multi-pack-index repack` operations. The series fixes a regression introduced in Git 2.47.0 where uninitialized packfile memory could be accessed, particularly visible under ASAN with 100+ packfiles. Key changes include restoring safer packfile initialization behavior, improving error handling patterns, and adding stress tests. The series has addressed all previous feedback and appears ready for merging, with the segfault fix being particularly urgent for users experiencing crashes during background maintenance.

**Smoother deprecation path for git whatchanged** -- Jeff King proposed allowing aliases to override deprecated commands like `git whatchanged`, responding to concerns from kernel developers about workflow disruption. The suggested change to `git.c` would let users create a `whatchanged` alias that works across Git versions during the deprecation period. While acknowledging limitations (like containers lacking the alias configuration), this represents a pragmatic compromise between deprecation policies and real-world toolchain constraints. The discussion highlights ongoing tensions between maintaining backward compatibility and cleaning up legacy interfaces.

**ODB source system integration finalized** -- Derrick Stolee confirmed Patrick Steinhardt's MIDX --object-dir functionality maintains proper handling of alternate object databases without requiring full repository context. This review removes the last potential blocker for the ODB source system abstraction effort, validating that the changes correctly pass an object source structure rather than constructing a full repository. With all key reviewers now satisfied, the series appears ready for integration as previously indicated by Junio Hamano.

## In brief

**Test reorganization completion** -- Usman Akinyemi acknowledged the successful merge of the v6 series standardizing help command testing, thanking contributors for resolving all technical issues.

**Sparse-checkout rebase strategy** -- Derrick Stolee confirmed he'll proceed with rebasing his sparse-checkout series independently after Junio's decision, while leaving a small window for potential last-minute coordination.

**Curl tracing documentation fix** -- Jeff King and Kristoffer Haugsbakk discussed better practices for extracting manpage text to avoid formatting artifacts in documentation quotes.

**Contributor Summit registration** -- Shejialuo sought clarification about remote participation requirements for the upcoming hybrid event, indicating some remaining ambiguity in the registration workflow.

## On the radar

**Interactive add input validation** -- A new bugfix targets malformed numerical input in `git add -i`, though the patch currently lacks test coverage for the validation changes.