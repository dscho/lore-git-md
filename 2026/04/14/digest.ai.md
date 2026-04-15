Here's the Git mailing list digest for April 14, 2026:

**The day in brief.** A busy Tuesday with 93 emails across 16 threads, featuring significant progress on test infrastructure modernization, ongoing refactoring efforts, and multiple release announcements. Key developments include Patrick Steinhardt's in-memory ODB series receiving final review approval and Taylor Blau's incremental MIDX repacking work advancing toward merge readiness.

**Notable threads**

**Test infrastructure modernization**  
Patrick Steinhardt's series to enable `set -e` in Git's test suite continues to uncover edge cases, with Jeff King identifying a macOS-specific Bash 3.2.57 issue where `set -e` incorrectly propagates through `command` invocations. The discussion reveals philosophical differences about whether the complexity of strict error handling justifies the maintenance burden given platform inconsistencies. Meanwhile, Siddharth Shrimali's series modernizing t7004-tag.sh received positive feedback from Junio Hamano, particularly praising the elimination of global state dependencies in test assertions.

**In-memory ODB backend approved**  
Patrick Steinhardt's 17-part series implementing an in-memory object database backend has cleared final review from Karthik Nayak. The production-ready implementation includes comprehensive unit tests and completes all required callback operations. This marks a significant milestone in the ODB abstraction effort, enabling features like git-blame worktree commits while cleaning up technical debt in Git's object storage layer.

**MIDX repacking refinements**  
Taylor Blau provided a status update on his incremental MIDX/bitmap-based repacking series, confirming consensus on the architectural approach with Jeff King. The upcoming v2 will switch from `string_list` to `strset` for tracking retained MIDX files and rename the `--checksum-only` option to `--no-write-chain-file` for clarity. This work represents a major scalability improvement for large repositories after multiple years of development.

**Submodule SSH configuration debate**  
A thread about submodule initialization evolved from workflow questions to design discussion, with Shibo Xia proposing a new per-submodule config injection mechanism for SSH commands during `git submodule update --init`. The proposal would extend Git's configuration flexibility but requires careful handling of precedence rules. Jeff King and Junio Hamano have offered alternative approaches, with the discussion now weighing architectural tradeoffs.

**In brief**  

**xdiff refactoring follow-up** -- Phillip Wood suggested post-merge improvements to Ezekiel Newren's xdiff cleanup series, proposing alternative INVESTIGATE handling logic that would make state transitions more explicit.

**Git v2.54.0-rc2 released** -- The second release candidate includes 744 non-merge commits, reverting the problematic writev() series while introducing experimental `git history` command and various performance optimizations.

**Git for Windows security update** -- Johannes Schindelin announced version 2.53.0(3) addressing CVE-2026-32631, which could expose NTLMv2 hashes via symbolic links during clone operations.

**Git for Windows 2.54.0-rc2** -- This parallel release removes `git svn` due to maintenance burden and updates several bundled components while fixing Windows-specific issues in terminal input and worktree operations.

**On the radar**  

**Pseudo-merge bitmap fixes** -- Taylor Blau's series addressing pseudo-merge bitmap bugs is undergoing thorough test infrastructure review, with Junio Hamano identifying a memory safety issue in the new `test-tool bitmap write` helper that needs resolution before the core fixes can proceed.