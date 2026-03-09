Here's the daily digest for November 29, 2025:

---

### The day in brief
A moderately busy day with 40 emails across 14 threads, featuring Junio's "What's cooking" status update, continued work on Windows test compatibility, and several documentation improvements. The most active discussions centered around hook subsystem refactoring and a bug investigation in `git fetch` with submodules.

---

### Notable threads

**Windows test suite prepares for symlink support**  
Johannes Schindelin submitted a 10-patch series addressing test failures that would occur when MSYS2 enables symlink support by default. The changes accommodate Windows path handling quirks and MSYS2's symlink normalization behavior, fixing issues in tests ranging from credential cache handling to difftool operations. This continues the long-running effort to improve Windows testing fidelity, with most fixes being surgical adjustments to test expectations rather than core behavior changes.

**Hook subsystem refactoring progresses**  
Adrian Ratiu's hook subsystem refactoring saw patch 2/10 reviewed, introducing stdin handling via callback as an alternative to file-based input. The change received positive feedback and sign-offs, though Junio noted email formatting issues in the submission. The series appears on track for inclusion, with only minor documentation fixes planned for v4.

**ARM64 fetch crash traced to commit-graph issue**  
A thread investigating an ARM64-specific SIGSEGV during submodule fetches with commit-graph writing narrowed the cause to a specific configuration (`fetch.writeCommitGraph=true` and `submodule.recurse=true`). Jeff King identified d70f554cdf ("commit-graph: retain commit slab when closing NULL commit_graph") as the likely fix, which landed between Git 2.43.0 and 2.52.0. The discussion provided a clear example of how subtle memory management issues can manifest differently across platforms.

**In brief**  
**`git replay` documentation fixes** -- Junio reviewed corrections to the git-replay manual, clarifying it accepts only a single revision range and fixing misleading references to non-existent options.

**`last-modified` sparse checkout fix** -- A bugfix enables `git last-modified` to work with sparse checkouts by properly handling the `--` separator syntax.

**Scalar config documentation** -- Matthew Hughes and Derrick Stolee submitted documentation patches for `scalar` configuration, though part of the work was found to overlap with an already-merged PR.

**CI test output visibility finalized** -- Johannes Schindelin signed off on Junio's changes to Docker CI test output permissions, resolving the thread with no further action needed.

**`git check-ignore` exit code discussion** -- David Goldstein provided additional context about discovering the exit code discrepancy while debugging ripgrep behavior, reinforcing that this is a low-priority issue given the command's debugging nature.

**`git diff --max-depth` path handling bug** -- Han Jiang reported a bug where the command fails to produce output when paths end with slashes or are just '.', though no fix has been proposed yet.

---

### On the radar
**Rustification effort** -- While no new patches appeared today, this remains an active area of discussion following recent exchanges between Ezekiel Newren and Patrick Steinhardt about architectural direction.

**ODB abstraction** -- Patrick Steinhardt's object database refactoring work was noted in Junio's "What's cooking" as continuing to progress through the integration branches.