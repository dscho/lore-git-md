# Git Mailing List Digest — 2025/08/30

**The day in brief.** A moderately active day with 16 emails across 8 threads, dominated by technical discussions around midx-write.c fixes and a proposal for smoother command deprecation. The most notable developments include Derrick Stolee's v2 series addressing critical multi-pack-index segfaults and Jeff King's proposal for handling deprecated commands via aliases.

## Notable threads

### Multi-pack-index segfault fixes reach v2

Derrick Stolee submitted version 2 of his series fixing memory safety issues in midx-write.c that could cause segfaults during `git multi-pack-index repack` operations, particularly with 100+ packfiles. The six-patch series addresses a regression introduced in Git 2.47.0, with the key fix properly initializing packfile memory and restoring error handling paths that were lost during earlier refactoring. The changes have been reviewed by Taylor Blau and Junio Hamano, with additional discussion about standardizing error handling patterns. An EXPENSIVE test case demonstrates the fix under stress conditions. This appears ready for merging as it addresses a potentially serious stability issue affecting users with many packfiles.

### Proposal for smoother command deprecation

Jeff King responded to concerns about `git whatchanged` deprecation by proposing a code change to allow aliases to override deprecated commands. The patch would let users create a `whatchanged` alias pointing to `git log --raw` during the deprecation period, addressing workflow concerns raised by kernel maintainers. While acknowledging limitations (like container environments), the proposal shows flexibility in balancing deprecation policies with real-world usage. The technical change is minimal — modifying `git.c` to check for deprecated commands before enforcing the usual alias override restrictions — but represents an interesting compromise in the ongoing lifecycle management discussion.

### ODB source system integration clears final review

Derrick Stolee confirmed Patrick Steinhardt's ODB source system integration series properly handles the `--object-dir` option for MIDX operations, removing the last potential blocker noted in the thread. The review focused on ensuring the changes maintain correct behavior when referencing alternate object databases without full repository contexts. With this validation from all key reviewers (Taylor Blau, Karthik Nayak, and Stolee), the series appears ready for integration as previously indicated by Junio Hamano, completing the MIDX portion of the broader ODB abstraction effort.

## In brief

**Test reorganization acknowledgment** — Usman Akinyemi sent a thank-you note marking the successful merge of the v6 series standardizing help command testing, with all technical issues resolved.

**Sparse-checkout rebase strategy** — Derrick Stolee acknowledged Junio Hamano's decision to proceed with rebasing the sparse-checkout series independently rather than waiting for Ayush's stalled topic, while leaving a small coordination window open.

**Curl tracing documentation** — Jeff King and Kristoffer Haugsbakk discussed better practices for extracting manpage text to avoid formatting artifacts in documentation quotes.

**Contributor Summit registration** — Shejialuo sought clarification about remote participation requirements for the upcoming Git Contributor's Summit, indicating some remaining ambiguity in the registration workflow.

**Interactive add input validation** — A two-line bugfix addressed malformed numerical input handling in `git add -i`, preventing incorrect interpretation of inputs like "2m3" as range selections.