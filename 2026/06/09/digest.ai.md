Here is the Git mailing list digest for June 9, 2026:

**The day in brief.** A busy Wednesday with 109 emails across 31 threads, featuring significant progress on several fronts. Key developments include final refinements to the security-hardened `git cat-file --batch-command` series, resolution of the promisor remote auto-configuration debate, and multiple performance optimizations nearing completion. Junio's "What's cooking" report provides a comprehensive snapshot of the current development landscape.

**Notable threads**

**Security-hardened cat-file reaches final polish**  
Pablo Sabater's 12-iteration series implementing dynamic capability-based validation for `git cat-file --batch-command` is in its final polishing phase. Today's exchanges resolved the last few nits around type safety and test infrastructure organization. The series, which enables secure object metadata queries from remotes without full downloads, has received thorough review from core contributors including Junio Hamano and Patrick Steinhardt. With all technical concerns addressed, it appears ready for merging once these final documentation tweaks are incorporated.

**Promisor remote configuration consensus reached**  
The long-running discussion about promisor remote auto-configuration reached resolution as Christian Couder and Toon Claes found common ground on the `acceptFromServer`/`acceptFromServerUrl` interaction. The v4 series preserves the fall-through behavior Christian advocated while addressing Toon's concerns about configuration clarity. Junio's subsequent approval clears this production-ready enhancement for merging, completing work that enables secure, flexible remote object access policies.

**ODB abstraction uncovers refdb issues**  
Patrick Steinhardt's major ODB abstraction series (converting packed object storage to use `struct odb_source`) uncovered deeper problems in Git's refdb implementation during final review. The thread now documents three significant issues: memory leaks in repository ref storage, duplicate refdb creation, and initialization ordering problems with conditional logic. Patrick has prepared 11 patches to address these but notes they depend on his earlier setup series being merged first. This demonstrates how foundational changes can expose hidden architectural debt in related subsystems.

**In brief**  
**Ref-filter performance fix** -- Tamir Duberstein's patch restoring prefix-scoped iteration for ref-filter commands shows 25x speedups when listing branches/tags in repos with many loose refs.  

**Describe optimization** -- Tamir's other patch limiting `git describe` ref iteration to tags by default achieves 17x speedups in large repositories.  

**Worktree diff performance** -- Jeff King identified racy-git behavior as the root cause of worktree diff slowdowns, suggesting USE_NSEC builds as the proper solution.  

**Config include disablement** -- Patrick Steinhardt endorsed Derrick Stolee's binary disable approach for config includes despite earlier suggesting an allow-list mechanism.  

**Reachability refactoring** -- A clean-up removed redundant `get_reachable_subset()` in favor of the newer generation-number-aware `tips_reachable_from_bases()`.  

**Update-ref rename option** -- Junio proposed adding `--rename` to `git update-ref` for low-level ref renaming with reflog preservation.  

**On the radar**  
**Branch deletion series** -- Harald Nordgren's `git branch --delete-forked` v14 addresses the test failures Junio identified, but platform-dependent issues remain under investigation.  

**History reword behavior** -- Pablo Sabater's RFC for `git history reword` continues evolving based on Junio's feedback, now considering silent handling of no-op cases rather than explicit abort.  

**Worktree metadata** -- Norbert Kiesel's worktree timestamp/description tracking awaits series reorganization per Phillip Wood's review to improve commit message clarity.