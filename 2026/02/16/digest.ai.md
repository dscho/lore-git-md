Here's the daily digest for February 16, 2026:

**The day in brief.** A moderately busy day with 60 emails across 20 threads, featuring several significant patch series reaching maturity. Key developments include finalization of the `git repo info --keys` interface, completion of promisor remote enhancements, and performance optimizations for shallow pushes. Multiple series received final review approvals and are ready for integration.

**Notable threads**

**Promisor remote enhancements complete**  
Christian Couder's 9-patch series implementing secure field storage and dynamic filter combination for promisor remotes has reached its fourth iteration with all feedback addressed. The series introduces `promisor.storeFields` configuration for securely storing server-advertised fields and implements `--filter=auto` mode for dynamic filter combination during clone/fetch operations. Patrick Steinhardt has approved the series as merge-ready after Jeff King identified and fixed a NULL pointer dereference in the filter release logic. This represents a significant enhancement to partial clone capabilities, allowing servers to suggest optimal filters while maintaining security.

**UTF-8 alias support finalized**  
Jonatan Holmgren's series enabling UTF-8 in Git alias names through config subsection syntax (`[alias "förgrena"]`) has reached its fifth iteration. The v5 series addresses a zsh completion bug discovered after v4 review while maintaining all implemented functionality. The changes include consolidated alias listing, proper subsection syntax handling, and comprehensive test coverage for non-ASCII aliases. Junio Hamano and Jeff King have approved earlier versions, and the shell completion fix was the final outstanding issue before merge readiness.

**Worktree API cleanup progresses**  
Phillip Wood has built on Shreyansh Paliwal's earlier RFC with two patches that advance the worktree API cleanup effort. The changes introduce `get_worktree_from_repository()` to construct worktree structs from repository data and remove the redundant `repository` parameter from `worktree_git_path()`. This continues the architectural direction validated by Junio Hamano to make worktree handling more explicit while maintaining backward compatibility. The patches interact with Paliwal's parallel wt-status cleanup work while keeping changes focused and reviewable.

**In brief**

**`git repo info --keys` finalized** -- Patrick Steinhardt confirms the v6 changes address all feedback for this machine-readable key discovery interface, with Junio Hamano queuing it for `next`. The format naming debate ("lines" vs "newline") is resolved while maintaining user experience consistency.

**`the_repository` config migration** -- Olamide Caleb Bello's v7 series migrates three configuration variables to `struct repo_config_values`, resolving a fuzz testing regression via Phillip Wood's `memset()` suggestion while maintaining safety checks.

**Gitweb mobile responsiveness** -- Rito Rhymes' v2 series adds mobile-specific CSS rules to fix layout issues on narrow screens, reorganizing patches per Eric Sunshine's feedback while preserving desktop behavior.

**Shallow push optimizations** -- Patrick Steinhardt begins a performance series targeting `git receive-pack` handling of shallow pushes, with the first patch showing 1.63x speedup and 60x memory reduction by avoiding unnecessary non-commit parsing.

**`git subtree` regression fix** -- Colin Stagner's patch removing a problematic optimization receives maintainer feedback on trailer formatting, with the technical approach appearing sound pending final polish.

**On the radar**

**Security hardening for `.git` discovery** -- Tian Yuchen's series to validate `.git` file types continues refining error handling classification, with Junio Hamano now agreeing on the core approach but seeking clarification on directory handling cases.