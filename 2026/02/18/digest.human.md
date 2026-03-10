Here's the Git mailing list digest for February 18, 2026:

**The day in brief.** A busy day with 126 emails across 34 threads, dominated by ongoing architectural work (the_repository removal, ODB abstraction), security hardening, and test infrastructure improvements. Key developments include finalization of the ref backend selection series, progress on config-based hooks, and multiple security patches reaching maturity.

**Notable threads**

**Ref backend selection reaches final polish**  
Patrick Steinhardt's series enabling zero-downtime migrations between ref storage backends (files<->reftable) is in its final stages with maintainer approval secured. Today's discussion focused on stub management for reference directories, moving logic from backend-specific code to the generic refs layer while preserving files backend behavior. The implementation now supports configuration, environment variables, and URI-based control, primarily serving GitLab's migration needs. With all technical concerns resolved and comprehensive test coverage, this multi-version effort appears ready for merging.

**git replay gains --revert capability**  
Siddharth Asthana's v3 series adds server-side revert functionality to `git replay`, particularly useful for GitLab's Gitaly service. The implementation treats reverts as merges with swapped arguments to reverse diffs, creating new commits with messages following `git revert` conventions. Junio Hamano established that as plumbing, `git replay` should ignore user configurations by default, settling an architectural question. The series has addressed all review feedback and been rebased on recent upstream changes that moved replay logic into a separate library.

**HTTP 429 rate limiting support finalized**  
Vaidas Pilkauskas's series implementing RFC-compliant handling of Retry-After headers is now complete with strbuf fixes and comprehensive test coverage. The implementation supports both delay-seconds and HTTP-date formats with configurable limits (http.maxRetries, http.retryAfter, http.maxRetryTime). Jeff King and Taylor Blau's reviews led to significant simplifications in v4, including replacing a complex non-blocking delay mechanism with simple blocking sleep(). The strbuf fixes grew into their own preparatory patches after Junio Hamano identified broader implications in the initial fix.

**Security hardening for repository discovery**  
Tian Yuchen's v5 series hardening Git's repository discovery by validating `.git` file types has been restructured per Junio Hamano's request. The patches now clearly separate error handling refinements from the security checks that reject dangerous entries (FIFOs/sockets) while allowing valid symlinks. A new test script verifies behavior with comprehensive cases. The series addresses all feedback but surfaces deeper questions about worktree/repository relationships that may need follow-up work. The security model itself is uncontested, with the implementation now properly distinguishing fatal errors from benign conditions.

**In brief**  
**Subtree split history fixes** -- Colin Stagner's series removes a problematic optimization that was incorrectly excluding commits during subtree splits, with tests confirming the fix preserves history integrity.

**UTF-8 alias support finalized** -- Jonatan Holmgren's series enabling UTF-8 in Git alias names via config subsections reaches v7 iteration with zsh completion fixes and output format standardization.

**Iconv test fixes** -- Patrick Steinhardt addresses CI failures on Windows/MSVC by making tests properly conditional on ICONV prerequisite when checking for the `iconv(1)` executable.

**Worktree API refinements** -- Phillip Wood corrects `is_bare` flag handling in `get_worktree_from_repository()` while documenting broader design questions about current worktree semantics.

**format.noprefix behavior change** -- Kristoffer Haugsbakk makes `format.noprefix` respect boolean values like its `diff.noprefix` counterpart, rejecting non-boolean configurations with clear errors.

**On the radar**  
**Push certificate privacy** -- Lorenz Leutgeb's proposal for multi-`pushee` headers to enable end-to-end verification through a transparency log model continues to evolve, balancing security and privacy requirements.

**Repository statistics** -- Justin Tobler's `git repo` metrics feature awaits final restructuring per Junio's suggestion to use array indexing for object type handling.