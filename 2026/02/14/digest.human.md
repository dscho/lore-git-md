Here's the daily digest for February 14, 2026:

**The day in brief.** A moderately active day with 48 emails across 20 threads, featuring significant progress on several long-running efforts. Key developments include the finalization of the ref backend selection series for GitLab migrations, approval of the `--keys` flag for repository metadata access, and continued refinement of security measures for patch parsing. The day also saw Junio's "What's cooking" report highlighting ongoing work across the project.

**Notable threads**

**Ref backend selection reaches final form**  
Patrick Steinhardt and Karthik Nayak's series enabling zero-downtime migrations between ref storage backends (files<->reftable) has completed its final polishing phase with maintainer approval. The v6 implementation now supports both configuration (`extensions.refStorage`) and environment-based (`GIT_REFERENCE_BACKEND`) control, with comprehensive const-correct string handling throughout the worktree reference path resolution and URI parsing components. This multi-version effort, primarily serving GitLab's needs for large repository migrations, is now ready for integration after addressing all substantive technical concerns.

**Repository metadata access approved**  
Lucas Seiki Oshiro's addition of a `--keys` flag to `git repo info` for programmatic repository metadata access has received final approval from Junio Hamano after six iterations. The implementation provides machine-readable key discovery with standardized output behavior, including both newline-delimited and NUL-terminated format options. The series also renamed internal format representations from "keyvalue" to "newline_terminated" for better accuracy while maintaining user-facing "lines" terminology for consistency. A minor documentation capitalization issue remains to be fixed in a follow-up.

**Security measures for patch parsing refined**  
Phillip Wood's security-focused commit-msg hook implementation reached v3 with refined regex patterns for detecting embedded diffs. The discussion clarified edge cases in scissors line handling (`--cleanup=scissors --verbose`) and validated the chosen regex approach for robustness when comment strings contain regex metacharacters. This hook is part of a layered solution addressing the long-standing issue of accidental patch application from commit messages, with other components including documentation warnings and Jeff King's proposed `--strict` mode for `git am` also progressing.

**Worktree API refactoring debate**  
Shreyansh Paliwal's RFC series to standardize primary worktree representation (using "/" instead of NULL) sparked detailed discussion about historical API semantics. Phillip Wood clarified the distinction between `wt==NULL` (meaning "use current worktree") and `wt->id==NULL` (meaning "main worktree"), leading Junio Hamano to endorse a gradual transition path via a new `get_current_worktree()` function. The exchange revealed deeper architectural questions about worktree identification that will need addressing in future iterations.

**In brief**  

**Shallow repository fixes** -- Samo Pogačnik's series addressing memory leaks and edge cases in relative-depth fetching is in final polishing, with discussion now focused on whether to keep depth measurement logic in a separate function or merge it into `get_shallow_commits()`.

**LOP series bugfix** -- Jeff King identified and fixed a potential NULL pointer dereference in the Large Object Promisors series' `list_objects_filter_release()` function, caught by Coverity static analysis.

**Path normalization refactor** -- Pushkar Singh's minimal refactoring of `normalize_path_copy_len()` (extracting slash-skipping logic into `skip_slashes()`) received a follow-up ping after being overlooked post-review.

**TRACE2 ancestry tracing** -- Derrick Stolee confirmed his review concerns have been addressed in Matthew John Cheetham's v2 series extending process ancestry tracing to macOS, including test coverage additions.

**NEEDSWORK documentation** -- Junio Hamano's patch formalizing `NEEDSWORK` comment conventions received final grammatical polish from Oswald Buddenhagen, completing the documentation addition.

**Date parsing behavior** -- Junio confirmed Git's deliberate use of current time rather than midnight for incomplete date specifications, citing Linus Torvalds' 2005 design rationale.

**Interactive patch navigation** -- Abraham Samuel Adekunle addressed Junio's feedback on type safety in the `--no-auto-advance` implementation, committing to revise `ssize_t` usage for better platform compatibility.

**On the radar**  

**Rustification effort** -- brian m. carlson's interoperability work continues, noted in Junio's "What's cooking" report as making progress alongside the object database abstraction efforts.

**MIDX/bitmap optimizations** -- Taylor Blau's incremental repacking work remains active, with further improvements expected in upcoming cycles.