Here's the daily digest for February 16, 2026:

**The day in brief.** A moderately busy day with 60 emails across 20 threads, featuring several significant patch series reaching maturity. Key developments include final approvals for the `git repo info --keys` feature, completion of the promisor remote enhancements, and performance optimizations for shallow pushes. The `the_repository` removal effort also saw progress with configuration variable migrations.

**Notable threads**

**Promisor remote enhancements complete**  
Christian Couder's 9-patch series implementing secure field storage and dynamic filter combination for promisor remotes has reached its fourth iteration, now with all review feedback addressed. The series introduces `promisor.storeFields` configuration for securely storing server-advertised fields and implements `--filter=auto` mode for dynamic filter combination during clone/fetch operations. Patrick Steinhardt has approved the series as merge-ready after Jeff King identified and fixed a NULL pointer dereference issue. This represents a significant enhancement to partial clone capabilities, allowing servers to suggest optimal filters while maintaining security.

**UTF-8 alias support finalized**  
Jonatan Holmgren's series enabling UTF-8 in Git alias names through config subsection syntax (`[alias "förgrena"]`) has reached its fifth iteration. The v5 series addresses a shell completion bug discovered after v4 review while maintaining all functionality. The implementation now includes comprehensive test coverage for Swedish, CJK, and space-containing aliases, with Junio Hamano and Jeff King having approved earlier versions. The shell completion fix was the final outstanding issue before merge readiness.

**Worktree API cleanup progresses**  
Phillip Wood has built on Shreyansh Paliwal's earlier work to clean up NULL parameter handling in Git's worktree API. The new patches introduce `get_worktree_from_repository()` to construct worktree structs from repository data and remove the redundant `repository` parameter from `worktree_git_path()`. These changes follow the architectural direction validated by Junio Hamano, moving toward embedded worktree references in repository structures while maintaining backward compatibility.

**Shallow push performance**  
Patrick Steinhardt has submitted a performance optimization series targeting shallow push handling in `git receive-pack`. The three patches work together to improve how `lookup_commit_reference_gently()` handles commit parsing, addressing real-world production issues where shallow pushes were causing excessive memory usage and multi-minute delays. Benchmarks show significant improvements - up to 3.93x speedup and 60x reduction in memory allocations in test cases.

**In brief**

**`git repo info --keys` approved** -- Patrick Steinhardt confirms the v6 changes address all feedback for this machine-readable key discovery interface, with Junio Hamano queuing it for `next`.

**Subtree regression fix** -- D. Ben Knoble notes a minor formatting issue in Colin Stagner's patch fixing a subtree split regression, suggesting trailer line adjustments.

**Documentation typo fixed** -- Ayush Jha responds to Chandra Pratap's review of a tree-walk.h comment typo fix, agreeing to adjust the subject line if needed.

**Gitweb mobile improvements** -- Rito Rhymes sends v2 of their mobile responsiveness series, reorganizing patches per Eric Sunshine's feedback while keeping the 74 lines of CSS additions.

**`format-patch` cover letter fix** -- A bugfix ensuring `--from` works consistently for cover letters now includes improved commit message and test coverage per Junio's review.

**On the radar**

**Rustification effort** -- Ezekiel Newren's work to introduce Rust code into Git remains active but hasn't seen new patches this week. The effort continues to face platform support concerns from Randall S. Becker regarding NonStop compatibility.

**ODB abstraction** -- Patrick Steinhardt's major object database refactoring (16 patches) is noted as ready for merging after a long review period in his response to Junio's "What's cooking" report.