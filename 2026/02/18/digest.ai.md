Here's the Git mailing list digest for February 18, 2026:

**The day in brief.** A busy day with 126 emails across 34 threads, dominated by ongoing architectural work (the_repository removal, ODB abstraction) and several feature refinements nearing completion. Key developments include final versions of the HTTP 429 retry support, config-based hooks, and UTF-8 alias handling, plus Junio's biweekly "What's cooking" report tracking progress across all active topics.

**Notable threads**

**Ref backend selection reaches final polish**  
Patrick Steinhardt's series enabling zero-downtime migrations between ref backends (files<->reftable) has secured maintainer approval after addressing all technical concerns. The v6 implementation now includes environment variable support (`GIT_REFERENCE_BACKEND`), URI-based configuration, and centralized stub management. Recent discussion confirmed path safety through Git's directory structure invariants, with only minor documentation refinements remaining before merging to 'next'.

**git replay gains --revert capability**  
Siddharth Asthana's v3 series adds server-side revert functionality to `git replay`, particularly useful for GitLab's Gitaly service. The implementation treats reverts as merges with swapped arguments and follows `git revert` message conventions. Junio Hamano established that as plumbing, `git replay` will ignore user configurations by default, resolving a key design question. The series has been rebased on recent refactoring that moved replay logic into a library.

**HTTP 429 retry support ready**  
Vaidas Pilkauskas' v4 series implementing RFC-compliant handling of rate limiting (Retry-After headers) is now complete with strbuf fixes and comprehensive tests. The changes introduce configurable limits (`http.maxRetries`, `http.retryAfter`) while falling back to immediate retry when no header is provided. Jeff King and Taylor Blau's reviews led to simplifications, replacing a complex non-blocking delay mechanism with simple blocking sleep().

**Config-based hooks implementation**  
Adrian Ratiu's v2 series enables defining hooks via Git configuration (`hook.<name>.command`) rather than filesystem scripts, adding a repository-level strmap cache for performance-critical cases like `reference-transaction` hooks. The implementation includes a `git hook list` subcommand and follows patterns established by Emily Shaffer's earlier work. Patrick Steinhardt's performance analysis justified the caching approach for high-frequency hooks.

**In brief**  
**Subtree split history fix** -- Colin Stagner removes a problematic optimization that incorrectly excluded commits during subtree splits, following regression reports from George Dietrich and Christian Heusel.  

**UTF-8 alias support** -- Jonatan Holmgren's v7 adds non-ASCII alias support via config subsections (`[alias "förgrena"]`), with zsh completion now matching bash behavior.  

**Shallow push optimizations** -- Patrick Steinhardt's 3-part series speeds up `receive-pack` handling of shallow pushes by 3.93x through commit-graph-aware object lookups.  

**Worktree API refinements** -- Phillip Wood addresses edge cases in `get_worktree_from_repository()` helper, fixing bare repo handling while leaving broader "current worktree" semantics for future discussion.  

**Security hardening** -- Tian Yuchen's v5 validates `.git` file types during repository discovery, rejecting dangerous entries (FIFOs) while allowing legitimate directory symlinks.  

**the_repository removal** -- Elijah Newren converts merge-ort and replay to use explicit repository parameters, adding compile-time guards against regression.  

**Documentation** -- Kristoffer Haugsbakk makes `format.noprefix` respect boolean values like its `diff.noprefix` counterpart, updating docs to match actual behavior.  

**On the radar**  
**Submodule initialization** -- Olamide Bello's repo_config_values work remains pending resolution of cross-repository scoping questions, particularly for submodules.  

**Parallel hook execution** -- Adrian Ratiu's preparatory state management changes suggest this capability may emerge from the config-based hooks foundation.  

**Rustification** -- Ezekiel Newren's effort to introduce Rust components continues separately from Sebastian Thiel's gitoxide project, though platform support concerns remain.