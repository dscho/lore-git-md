# Here's the daily digest for May 20, 2026:

---

### The day in brief
A moderately busy Wednesday with 45 emails across 19 threads saw several significant patches reach final approval, including major optimizations to bitmap generation and maintenance subsystem fixes. The day was marked by thorough technical discussions and multiple series clearing their final review hurdles.

---

### Notable threads

#### **Final approvals for major optimizations**
Taylor Blau's 16-patch series optimizing MIDX repacking received its final review confirmation from Junio Hamano, clearing the way for integration into 'next'. The changes enable efficient repacking of multi-pack indexes without full repacks, representing a significant performance improvement for large repositories. Jeff King had previously approved the technical approach, with SZEDER Gábor catching a minor build issue that was promptly fixed.

In a related development, Justin Tobler's ODB transaction interface rework was also approved for merging. This 7-patch series refactors Git's object writing to be more pluggable, serving as foundational work for Patrick Steinhardt's in-memory ODB backend effort. The changes had been through multiple review iterations and addressed a memory leak identified by Jeff King.

#### **Maintenance subsystem fixes ready**
Patrick Steinhardt's maintenance bugfix series concluded its review cycle with Jeff King reconfirming approval for both patches. The changes address two issues: preventing stale locks in detached maintenance runs and restoring correct behavior for `gc.auto` configuration. Taylor Blau suggested these fixes may warrant backporting to the stable 2.54.x series due to their severity.

#### **Git-gui bugfix series approved**
A comprehensive 11-patch series fixing git-gui's repository and worktree detection received maintainer approval from Johannes Sixt. The changes address long-standing edge cases that caused cryptic failures, particularly around bare repositories and missing worktrees. The implementation now uses `git rev-parse` consistently for repository discovery and adds explicit subcommands for controlling startup behavior.

#### **Commit-graph edge case resolved**
Derrick Stolee provided final approval for Jeff King's commit-graph bugfix that handles an edge case with `git clone --dissociate`. The patch adds a fallback path to read trees directly from disk when graph-based lookup fails, properly handling of repository-specific hash algorithms. Stolee noted this issue became more visible after Git 2.54.0's switch to geometric maintenance.

---

### In brief

**Promisor remote naming consistency** -- Junio C Hamano suggested renaming `promisor_info_internal_name()` to `promisor_info_local_name()` for clarity in Christian Couder's series, though not blocking on this change.

**Dynamic mailmap toggling approved** -- Siddharth Asthana's v5 patch implementing runtime control of mailmap behavior in `git cat-file --batch-command` received Junio's final approval.

**Negotiation options series ready** -- Derrick Stolee's bidirectional negotiation controls (`--negotiation-include` and `--negotiation-restrict`) completed review and await integration.

**`the_repository` removal milestone** -- Patrick Steinhardt's 18-patch series converting setup.c functions to explicit repository parameters was approved for 'next', marking significant progress in eliminating global state.

**`--max-count-oldest` CI fixes** -- Mirko Faina addressed portability issues in the test suite for the new `--max-count-oldest` option, resolving the last remaining issues before merging.

**`git status` pull advice refined** -- Harald Nordgren's v3 patch improves advice for triangular workflows by conditionally showing verbose `git pull` commands only when push/pull remotes differ.

**External notes command discussion** -- Siddh Raman Pant clarified the motivation for his external notes command proposal, explaining it addresses latency issues in large distributed repos like the Linux kernel.

**`git stash -p` optimization review** -- Junio questioned whether Aditya Garg's fsmonitor optimization could use `git read-tree -m HEAD` rather than new code, and noted a behavioral difference in index handling.

---

### On the radar

**`git log --follow` and subtree merges** -- Miklos Vajna's patch to align behavior with `blame --follow` awaits review from domain experts Elijah Newren and Jeff King after Junio raised design questions.

**Documentation workflow discussion** -- The thread about commit-graph documentation has evolved into a broader discussion about patch trailer conventions and maintainer responsibilities, with Kristoffer Haugsbakk providing practical analysis of current practices.

**Bitmap optimization series** -- Taylor Blau will send a v2 of his bitmap generation optimizations after SZEDER Gábor identified a build error in one patch, though the overall approach remains sound.

---

The day's activity reflects Git's ongoing evolution across performance optimizations, architectural improvements, and usability refinements, with multiple major efforts reaching maturity through the review process.