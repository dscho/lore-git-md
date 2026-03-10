Here's the Git mailing list daily digest for February 17, 2026:

---

### The day in brief
A busy day with 98 emails across 23 threads, dominated by ongoing refactoring efforts and final polish on several patch series. Key developments include finalization of the ref backend selection series, security hardening of repository discovery, and multiple discussions around worktree API improvements. The HTTP 429 retry support and `git format-patch --from` fixes also reached completion.

---

### Notable threads

**Ref backend selection reaches final polish**  
Karthik Nayak and Patrick Steinhardt's series enabling zero-downtime migrations between ref backends (files<->reftable) is now complete with all technical concerns resolved. The v6 iteration added environment variable support (`GIT_REFERENCE_BACKEND`), centralized stub management, and comprehensive path handling. Junio Hamano has indicated readiness to merge to 'next', with only minor test verification discussions remaining about alternate ref directory behavior. This concludes a multi-version effort critical for GitLab's large repository needs.

**Security hardening for repository discovery**  
Tian Yuchen's v4 patch series hardening `.git` file validation during repository discovery has addressed all substantive feedback. The changes distinguish between fatal errors (FIFOs/sockets) and non-fatal cases (missing paths), while maintaining symlink functionality through careful use of `stat()` over `lstat()`. Junio Hamano and Karthik Nayak provided final review, with consensus on the error handling strategy. Remaining work is limited to test script renaming and meson.build integration before merging.

**Worktree API refactoring progresses**  
The ongoing effort to standardize worktree handling saw extensive discussion about NULL parameter semantics and repository integration. Phillip Wood proposed `get_worktree_from_repository()` to construct worktree structs from repository data, while Shreyansh Paliwal continued parallel wt-status.c cleanup. Junio Hamano emphasized the need for clear documentation of assumptions, particularly around "current worktree" identification. The thread shows careful coordination between multiple contributors on this complex refactoring.

**HTTP 429 retry support finalized**  
Vaidas Pilkauskas's implementation of RFC-compliant handling of rate-limited HTTP responses (status 429) reached v3 with all feedback addressed. The series now uses blocking sleep() for retries, supports curl's native Retry-After parsing, and includes comprehensive trace2 instrumentation. Jeff King and Taylor Blau's architectural concerns have been resolved, making this production-ready for services like GitHub and GitLab that implement API rate limits.

---

### In brief

**`git format-patch` cover letter fix** -- Mirko Faina's patch ensures the `--from` option applies consistently to cover letters, fixing an inconsistency that had existed since 2013. Documentation is updated to clarify the behavior.

**German translation correction** -- Matthias Schwarzott fixes a misspelling of "allow-indentation-change" in the German localization that was causing error message loops.

**MacOS credential helper build fix** -- Koji Nakamaru addresses a build failure on older macOS systems where the `.depend` directory wasn't being created during dependency generation.

**Meson build dependency tracking** -- D. Ben Knoble finalizes dependency handling for `config-list.h`, ensuring rebuilds when documentation changes, with all path escaping and performance considerations addressed.

**`git apply --directory` path handling** -- Patrick Steinhardt suggests using `strbuf_normalize_path()` for more robust handling of path prefixes like `./` in the `--directory` option.

**GSoC project discussion** -- Tian Yuchen explores scope for a potential Google Summer of Code project focused on `the_repository` removal, receiving guidance from Karthik Nayak about subsystem priorities.

---

### On the radar

**Push certificate security discussion** -- Junio Hamano raised important concerns about Lorenz Leutgeb's proposal to modify push certificate "pushee" headers, highlighting potential security implications around replay attacks that need resolution.

**`core.trustctime` migration** -- Ayush Jha's RFC patch moving this config to `struct repo_settings` prompts discussion about lazy loading versus early validation tradeoffs in configuration handling.