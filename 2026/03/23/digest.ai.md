Here's the daily digest for March 23, 2026:

**The day in brief.** A moderately active day with 115 emails across 24 threads, featuring several series reaching completion and important architectural discussions. Key developments include final approvals for HTTP 429 retry handling and hook configuration improvements, significant progress on `the_repository` removal in fsck, and security-focused reviews of promisor remote whitelisting.

**Notable threads**

**HTTP 429 retry series concludes**  
Vaidas Pilkauskas's HTTP 429 rate limit retry series has received maintainer approval and will be merged into 'next'. The implementation adds configurable retry behavior for rate-limited HTTP requests through three new configs (`http.maxRetries`, `http.retryAfter`, `http.maxRetryTime`) with comprehensive test coverage. The v6 restructuring addressed all prior feedback, including proper curl version handling and cleaner parameter passing. This marks a significant enhancement to Git's HTTP transport capabilities.

**Parallel hooks and configuration finalized**  
Adrian Ratiu's parallel hook execution and hook configuration series have reached completion. The parallel hooks feature provides comprehensive controls through `hook.jobs` configuration and CLI options, while the configuration series adds `--show-scope` output and disabled hook visibility to `git hook list`. Both series have addressed all review feedback and are queued for merging, representing the culmination of Adrian's multi-series effort to modernize Git's hook subsystem.

**Fsck subsystem refactored for repository independence**  
Patrick Steinhardt's v2 series (12 patches) refactors Git's fsck subsystem to remove its dependence on the global `the_repository` variable, incorporating Junio Hamano's feedback to store the repository pointer in `fsck_options` rather than passing it through function parameters. The changes systematically convert all fsck operations - from object checking to error reporting - to use explicit repository pointers. This architectural improvement prepares fsck for future pluggable backend support as part of the broader `the_repository` removal effort.

**Promisor remote whitelisting security review**  
Christian Couder's 16-part series introducing URL-based whitelisting for promisor remotes is undergoing rigorous security review. Junio Hamano has raised concerns about URL pattern matching rules, particularly the risk of `*` globs matching across `/` boundaries (e.g., `https://*.example.com` matching `evil-hacker.net/fake.example.com`). The discussion is converging on restricting `*` matching in domain portions while maintaining compatibility with existing configurations. The series aims to reduce manual configuration burden while maintaining security controls.

**In brief**  

**Graph lane limiting** -- Pablo Sabater's v4 series adds `--graph-lane-limit` to constrain `git log --graph` width, now standardizing on `int` type and improving commit message clarity.

**Backfill argument handling** -- Derrick Stolee confirms his upcoming revision argument support for `git backfill` will handle validation through Git's standard revision parsing, making Siddharth Shrimali's argument validation patch obsolete.

**Diff-highlight improvements** -- Jeff King's v2 series enhancing the contrib script has been acked by Junio, addressing Perl 5.8 compatibility and test improvements.

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**MacOS build workaround** -- Johannes Schindelin provides context on the Homebrew clang/REG_ENHANCED issue, with Junio proposing to document rather than apply the workaround.

**On the radar**  

**Rustification effort** -- The discussion around hash algorithm handling patterns in oidtree code surfaces ongoing considerations for Rust integration, with brian m. carlson noting the need to stop treating zero algo fields as `the_hash_algo`.

**Partial clone disk space** -- Yuvraj Singh Chauhan's GSoC proposal exploration for `git backfill --evict` continues, with analysis of implementation approaches favoring a new `git evict` command over automatic pre-blame downloads.