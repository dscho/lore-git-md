Here's the daily digest for May 20, 2026:

---

### The day in brief
A busy Wednesday with 45 emails across 19 threads saw several significant developments in Git's development pipeline. Key highlights include final approvals for Patrick Steinhardt's `the_repository` removal in the setup subsystem, Derrick Stolee's negotiation options series, and Siddharth Asthana's dynamic mailmap toggling. Junio's "What's cooking" report provided a comprehensive overview of the current development landscape.

---

### Notable threads

#### **Promisor remote configuration finalized**
Christian Couder's promisor remote configuration series received its final review from Junio C Hamano, focusing on naming consistency in the auto-generated remote name handling. While technically complete and approved, Junio suggested renaming `promisor_info_internal_name()` to `promisor_info_local_name()` for clarity. This marks the conclusion of a significant enhancement to Git's promisor remote functionality, with both the LOP (Large Object Promisors) and URL-based auto-configuration features now ready for merging.

#### **`the_repository` removal milestone**
Patrick Steinhardt's 18-patch series to remove `the_repository` usage from Git's setup subsystem received final approval for merging into 'next'. The changes convert all remaining setup.c functions to use explicit `struct repository *` parameters, affecting 83 files with ~400 lines modified. This represents one of the largest remaining chunks in the multi-year effort to eliminate global state from Git's codebase, following a consistent three-step methodology that has been validated through extensive review.

#### **Negotiation options ready for integration**
Derrick Stolee's negotiation options series concluded its review cycle with final maintainer acknowledgment. The patches introduce bidirectional negotiation controls (`--negotiation-include` and `--negotiation-restrict`) to optimize object transfer in large repositories, particularly addressing expensive full-history transfers triggered by dropped refs in monorepos. With all technical feedback addressed and proper review tags in place, the series now awaits final integration.

#### **Git-gui bugfix series approved**
An 11-patch series comprehensively rewriting git-gui's startup logic received maintainer approval from Johannes Sixt. The changes address long-standing issues with repository and worktree detection, particularly around bare repositories and missing worktrees. The implementation now exclusively uses `git rev-parse` for repository discovery and adds explicit `gui`/`pick` subcommands for more predictable behavior. The series resolves issues dating back to 2014/2019 regressions while maintaining backward compatibility.

#### **Maintenance bugfixes cleared**
Patrick Steinhardt's maintenance subsystem bugfix series received final approval from Jeff King, addressing two distinct issues: preventing stale locks in detached mode and restoring expected behavior for the `gc.auto` configuration. The lockfile fix may be backported to the stable 2.54.x series due to its severity. The thread demonstrates Git's collaborative review process through three iterations with substantive feedback from multiple core contributors.

---

### In brief

**Dynamic mailmap toggling** -- Siddharth Asthana's v5 patch implementing mailmap toggling for `git cat-file --batch-command` received Junio's final approval, allowing runtime control of name/email rewriting.

**`--max-count-oldest` CI fixes** -- Mirko Faina addressed final portability issues in the test suite for the `--max-count-oldest` feature, using `test_line_count` instead of non-portable `wc -l`.

**Commit-graph lazy-loading** -- Jeff King's optimization for lazy-loading trees from commit-graphs cleared final administrative steps and was merged to 'next'.

**`git-jump` auto mode** -- Greg Hurrell's v2 patch adds intelligent mode selection to `git-jump`, now detecting merge conflicts, unstaged changes, and whitespace errors.

**`git status` pull advice** -- Harald Nordgren's v3 patch makes `git pull` advice more precise in triangular workflows while avoiding verbosity in standard cases.

**`git stash -p` optimization** -- A performance patch replaces external `read-tree` with in-process `unpack_trees()` to avoid fsmonitor refreshes on unchanged files.

**External notes commands** -- Siddh Raman Pant clarified the motivation for his proposed external notes feature, addressing latency issues in large distributed repos like the Linux kernel.

**Memory leak fix** -- Junio fixed a strbuf leak in `apply.c`'s patch application code, later recognizing Jeff King had independently addressed the same issue.

---

### On the radar

**`git log --follow` and subtree merges** -- Miklos Vajna's patch to align `log --follow` behavior with `blame --follow` for subtree merges awaits review from domain experts Elijah Newren and Jeff King.

**Bitmap optimizations** -- Taylor Blau's bitmap generation series saw a build error caught in review, with a straightforward fix needed before v2 submission.

**Word-diff documentation** -- Michael Montalbo's clarification of `--word-diff` behavior for whitespace changes is ready for merging after incorporating Junio's feedback.

--- 

The day's activity across performance optimizations, architectural improvements, and documentation refinements reflects Git's continued evolution on multiple fronts. Several long-running efforts reached significant milestones while new features progressed through the pipeline.