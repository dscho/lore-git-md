Here's the daily digest for February 14, 2026:

**The day in brief.** A moderately busy day with 48 emails across 20 threads, featuring significant progress on several long-running efforts including ref backend selection, interactive patch navigation, and repository security hardening. Key developments include the finalization of the ref backend series for GitLab's migration needs and Junio's "What's cooking" report highlighting ongoing work across the project.

**Notable threads**

**Ref backend selection reaches final form**  
Patrick Steinhardt and Karthik Nayak's series enabling zero-downtime migrations between ref storage backends (files<->reftable) has reached its final polishing phase with maintainer approval. The v6 implementation now supports both configuration (`extensions.refStorage`) and environment-based (`GIT_REFERENCE_BACKEND`) control, with comprehensive const-correct string handling in URI parsing and worktree reference resolution. This multi-version effort, primarily serving GitLab's needs for large repository migrations, includes centralized stub management and extensive test coverage in t1423-ref-backend.sh. The series has undergone extensive review from multiple contributors including Jeff King and Junio Hamano, with all substantive technical concerns resolved.

**Interactive patch navigation refined**  
Abraham Samuel Adekunle's series adding `--no-auto-advance` to interactive patch commands (`add -p`, `checkout -p` etc.) saw detailed review feedback addressed in v5. The changes allow manual navigation between files during patch selection rather than automatic advancement, implemented through careful refactoring of the `patch_update_file()` logic. Junio Hamano provided type safety guidance about using `ssize_t` versus Git's conventional `int` for indexing, which the author has committed to address in the next version. The series demonstrates Git's attention to both user experience improvements and code quality standards.

**Repository security hardening progresses**  
Tian Yuchen's v3 patch hardening Git's repository discovery by validating `.git` file types implements Junio Hamano's suggested error taxonomy, distinguishing between different failure modes when checking repository entries. The changes address security concerns while preserving backward compatibility with symlink usage, using `stat()` rather than `lstat()` to maintain historical behavior. The refined implementation includes new error codes in setup.h and updated logic in `setup_git_directory_gently_1()`, with test coverage for various `.git` file types (regular files, symlinks, FIFOs).

**In brief**  

**`git repo info --keys` finalized** -- Lucas Seiki Oshiro's feature addition to the experimental `git repo` command is queued for integration, providing machine-readable way to list available metadata keys with both newline` and NUL-terminated output options.

**Shallow repository fixes** -- Samo Pogačnik's series fixing memory leaks and edge cases in relative-depth fetching sees final discussion about code organization, with consensus forming around keeping depth measurement logic separate from commit generation.

**LOP series bugfix** -- Jeff King identified and fixed a potential NULL pointer dereference in the Large Object Promisors series' filter release logic, caught by Coverity static analysis.

**Path normalization refactoring** -- Pushkar Singh's minimal refactoring of `normalize_path_copy_len()` with Junio-approved `skip_slashes()` helper receives a follow-up ping after review completion.

**Process ancestry tracing for macOS** -- Derrick Stolee confirms Matthew John Cheetham's extended TRACE2 support for macOS addresses all concerns, with test coverage now matching Linux implementation.

**Patch ID documentation** -- Kristoffer Haugsbakk's series improving `git-patch-id` man pages with batch processing guidance and practical examples receives final approval from D. Ben Knoble.

**`NEEDSWORK` comment guidelines** -- Junio C Hamano's documentation codifying conventions around `NEEDSWORK` comments receives final grammatical polish from Oswald Buddenhagen.

**`git config list --type` optimization** -- Derrick Stolee continues performance work on type-specific formatters, with Junio suggesting minor style improvements to the integer handling implementation.

**`sendfile()` optimization discussion** -- Platform-specific constraints emerge in George Hu's proposed `sendfile()` optimization, with FreeBSD's socket-only limitation complicating a portable solution.

**Worktree API refactoring** -- Phillip Wood clarifies historical distinction between `wt==NULL` and `wt->id==NULL` in worktree handling, suggesting `get_current_worktree()` for gradual cleanup.

**On the radar**  

**`git am` security solutions** -- Phillip Wood's security-focused commit-msg hook sees final edge case verification, completing one layer of the multi-pronged approach to patch parsing vulnerabilities.

**`the_repository` removal** -- Ayush Jha's RFC series making `read_attr()` repository-aware receives process guidance from Lucas Seiki Oshiro as it nears final review.