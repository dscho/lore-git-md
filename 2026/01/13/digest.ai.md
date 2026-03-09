Here's the Git mailing list digest for January 13, 2026:

## The day in brief

A busy day with 91 emails across 22 threads, dominated by major feature finalizations and bugfixes. The standout developments include the long-awaited completion of the submodule path encoding series (ready for merging after 10 iterations) and significant progress on Patrick Steinhardt's `git-history` command (now at v11 with all technical feedback addressed). Several important bugfixes also emerged, particularly around hook behavior and HTTP authentication.

## Notable threads

### Submodule path encoding reaches completion

After 10 iterations spanning months, Adrian Ratiu's submodule path encoding series received final approval from Patrick Steinhardt and Junio Hamano. The implementation provides robust filesystem-safe handling of submodule paths through URL encoding and hashing fallbacks, with comprehensive collision resolution. Josh Steadmon confirmed production readiness by verifying it can replace Google's internal implementation. This marks a major milestone in submodule handling reliability.

### `git-history` command nears readiness

Patrick Steinhardt's `git-history` series (v11) implementing the `reword` subcommand cleared its final technical hurdles after extensive review from merge machinery expert Elijah Newren. The series refactors replay infrastructure into libgit.a while adding detached HEAD support and standardized conflict reporting. While some documentation questions remain open, the core functionality appears technically complete and well-tested with 391 lines of test coverage.

### Pre-push hook output regression fix

A regression causing pre-push hook stdout to be incorrectly redirected to stderr (breaking tools like Git LFS) prompted a thorough discussion about hook API behavior. Adrian Ratiu's fix makes the output handling configurable while maintaining backward compatibility, with Junio Hamano confirming through code archaeology that pre-push hooks uniquely required separate streams. The solution includes new test coverage to prevent similar regressions.

### HTTP authentication fix for large clones

A bug in `remote-curl.c` causing authentication failures during large clone operations (when requests exceeded `http.postBuffer`) was fixed with thorough test coverage. The issue occurred when `probe_rpc()` requests failed to include authentication headers despite the main request succeeding. The minimal (2-line) fix comes with comprehensive tests using `git-credential-msal` to verify bearer token flows.

### MIDX compaction final polish

Taylor Blau's multi-pack-index compaction series received final review on its bitmap handling implementation, with Patrick Steinhardt and Taylor discussing naming conventions and edge cases. The series now correctly handles checksum validation and maintains bitmap ordering during compaction. Taylor outlined plans for follow-up geometric repacking work building on this foundation.

## In brief

**`git status` push tracking finalized** -- Harald Nordgren's 25th iteration adding push branch divergence info to `git status` was approved, though Jeff King later noted ergonomic concerns for triangular workflows.

**Tree parsing `the_repository` removal** -- René Scharfe's series eliminating implicit `the_repository` usage in tree parsing functions concluded with discussion about Coccinelle rule maintenance.

**Configuration variable scoping** -- Olamide Caleb Bello's Outreachy series moving configs into `struct repo_config_values` prompted Junio Hamano to question whether all configs should be repository-scoped.

**Test modernization** -- Shreyansh Paliwal's GSoC work fixed exit code suppression in t5500 test #365 using `test-tool pkt-line pack`, earning Junio's approval as "the right tool for the job."

**Documentation style clarification** -- Michael Lyons' git-bisect.adoc conversion led to discussion about when to use `[synopsis]` blocks versus verbatim formatting in examples.

## On the radar

**Hook performance regression** -- A newly reported issue shows pre-push hooks reading stdin became dramatically slower after commit 857f047e, with Adrian Ratiu preparing a fix.

**`git shortlog` option behavior** -- Documentation discrepancies around `--committer` prompted discussion about whether to implement pattern matching or clarify docs, leaning toward the latter.

**Global config variable migration** -- Junio's questions about Olamide's series suggest the project may need clearer guidelines about which configs should remain global versus becoming repository-specific.