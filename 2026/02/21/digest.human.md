Here's the daily digest for February 21, 2026:

## The day in brief

February 21 saw steady progress across multiple fronts, with several long-running threads reaching completion while new discussions emerged. The day's 45 emails across 21 threads were dominated by finalizations of major features (configurable branch comparisons in `git status`, Linux fsmonitor support) and design discussions around `git format-patch` cover letter formatting. Junio's "What's cooking" report provided a comprehensive snapshot of the project's current state.

## Notable threads

**Configurable branch comparisons in `git status` finalized**  
After 28 iterations spanning months, Harald Nordgren's effort to add configurable branch comparisons to `git status` is ready for merging. The feature implements `status.compareBranches` configuration to show comparisons against multiple branches, initially supporting only `@{upstream}` and `@{push}` syntax per Junio's phased approach. Jeff King contributed foundational changes to `repo_dwim_ref()` to handle edge cases, and extensive test coverage (337 lines) verifies the behavior. The thread's conclusion was nearly derailed when Junio noted the topic had gone stale in his branches, prompting a final check for real-world interest before merging.

**Linux fsmonitor implementation nears completion**  
Paul Tarjan's inotify-based Linux fsmonitor backend (now at v4) appears ready after months of development. The patch has been stable in production for two months across a large deployment, addressing earlier concerns about memory leaks and GPL compliance. The only remaining issues are two small memory leaks (512-byte during startup and 40-byte in IPC handling) that Junio suggests could be fixed post-merge. Patrick Steinhardt's meson build support patch will be split off to unblock the main functionality, marking the end of a long effort to bring Linux to parity with existing Windows/macOS fsmonitor backends.

**Symbolic refs crossing namespace boundaries spark security debate**  
Troels Thomsen's bugfix for `receive-pack` crashes when handling symbolic refs pointing beyond their namespace has evolved into a security policy discussion. While the NULL dereference fix is uncontroversial, Junio argues the underlying behavior (allowing cross-namespace symrefs) may violate namespace isolation principles. The thread is at an impasse between flexibility (allowing intentional cross-namespace links) and safety (preventing accidental security holes), with Junio now suggesting explicit rejection of such updates. This touches deep design questions about Git's namespace model that may require broader discussion.

**Cover letter format customization takes shape**  
Mirko Faina's RFC about improving `git format-patch`'s cover letter commit list format has evolved into a design for configurable formatting. The current author-grouped shortlog obscures patch series flow, prompting a proposal for numbered chronological lists. Junio countered with a more flexible `format.commitListFormat` configuration using printf-style strings, automatically prepending `[N/M]` position markers. Discussion now centers on command-line override syntax, with consensus forming around the configurable approach. The thread shows Git's design process in action - from specific pain point to generalized solution.

**`the_repository` removal advances in merge-ort**  
Elijah Newren's series eliminating `the_repository` from merge-ort and replay reached v3, now fixing the last identified issue with prefetch logic. The changes propagate repository context through the merge machinery, replacing global variable usage with `opt->repo` parameters. The final patch enforces compile-time prevention of future `the_repository` usage via a `DO_NOT_USE_THE_REPOSITORY` macro. This represents significant progress in the long-term effort to remove implicit repository dependencies, with Elijah's merge-ort expertise ensuring the changes maintain correct behavior.

## In brief

**Histogram diff edge case fix** -- Yee Cheng Chin confirms the v2 patch addressing shifted change groups in XDF_HISTOGRAM_DIFF is coming soon, with all technical aspects resolved.

**Path normalization refactoring** -- Pushkar Singh's extraction of `skip_slashes()` in `path.c` is ready after clarifying the commit message to match the reduced scope.

**Config-based hooks parallel execution** -- Adrian Ratiu clarifies parallel hook support is coming in a separate series that will maintain backward compatibility.

**Gitweb mobile responsiveness** -- Rito Rhymes' mobile CSS improvements are merge-ready after addressing Eric Sunshine's feedback about organization.

**String list sorting standardization** -- Elijah Newren and Amisha Chhajed finalize patches ensuring consistent sorted+deduplicated output across commands.

**Interactive patch selection controls** -- Samuel Abraham's `--[no-]auto-advance` for `add -p` et al. completes its GSoC journey with maintainer approval.

**Pack-redundant memory leak fix** -- A straightforward 4-line patch fixes `llist` leakage when `open_pack_index()` fails.

**Test modernization** -- Lambert Duclos-de Guise replaces `test -f` with `test_path_is_file` in t2004 as part of GSoC work.

**HTTP protocol documentation** -- The spec now explicitly defines the "ERR no wants received" response for empty upload-pack requests.

## On the radar

**Send-email character encoding prompt** -- Shreyansh Paliwal's first contribution improves the 8-bit encoding prompt clarity, with Junio weighing in on respecting user input.

**Client certificate support for send-email** -- David Timber addresses Junio's questions about Perl's asymmetric PKCS12/PEM handling in SSL integration.

**GSoC contributor onboarding** -- Lambert Duclos begins the standard introduction process with patch submission guidance from Usman Akinyemi.