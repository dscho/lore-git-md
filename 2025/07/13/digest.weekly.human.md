# Git Mailing List Digest - 2025/07/07 -- 2025/07/13

**The week in brief.** A busy week with 489 emails across 141 threads saw significant progress on multiple fronts. Key developments included security releases addressing critical vulnerabilities, major refactoring work on the object database and `the_repository` removal, and finalization of several long-running patch series. The week also featured active discussions about community norms, platform compatibility challenges, and the upcoming Git 3.0 release.

## Key developments

### Security vulnerabilities addressed

The week began with coordinated security releases for Git (v2.43.7 through v2.50.1) and Git for Windows, addressing seven CVEs with potential remote code execution risks. The fixes targeted Gitk and Git GUI vulnerabilities allowing arbitrary file manipulation (CVE-2025-27613/27614), Windows-specific path handling issues (CVE-2025-46334/46835), CRLF handling in configs (CVE-2025-48384), and a wincred helper buffer overflow (CVE-2025-48386). These releases required careful test suite adjustments to maintain backward compatibility in older versions while securing critical attack vectors.

### Object database refactoring milestone

Patrick Steinhardt's 19-part series to remove `the_repository` dependencies from object-file.c reached completion, systematically eliminating global state in preparation for pluggable backends. The changes introduce repository-specific settings for compression and object creation modes, refactor loose object handling to use `struct odb_source`, and add a new `odb_write_object()` interface. A regression in `git fast-import` was reported where compression level validation now happens too late, highlighting an edge case in the new configuration handling approach that may need follow-up.

### `git last-modified` command finalized

The `git last-modified` series (formerly `blame-tree`) reached its final form in v4, addressing all major review feedback. Key changes included renaming from `blame-tree`, removing inconsistent optimizations, consolidating implementation, and adding Bloom filter support. The thread resolved its last open question about path handling behavior, with Junio agreeing the command should match `git diff-tree`'s output format rather than `git ls-tree`. A minor upcoming API change to Bloom filter functions was noted but won't require another revision.

### Bloom filter optimizations complete

Lidong Yan's six-part series optimizing pathspec handling with bloom filters has reached its final form, with all substantive feedback addressed. The patches introduce `struct bloom_keyvec` to support multiple literal pathspec queries while maintaining the strict "all must match" requirement. Benchmark results show significant speedups - 1.33x in Git's own repository and 7.5x in the LLVM repository for multi-pathspec cases. The implementation carefully preserves fallback behavior for wildcard pathspecs while optimizing the literal case.

### Reftable as default for Git 3.0

Junio Hamano acked Patrick Steinhardt's v2 series to establish reftable as the default ref storage format in Git 3.0, maintaining the versioned default transition pattern with build-time guard. This marks a significant milestone in the multi-year effort to modernize Git's reference storage, with reftable offering better performance and atomicity guarantees compared to the traditional files-based backend.

## In brief

**`git for-each-ref` pagination** -- The four-part series introducing `--start-after` pagination was approved, providing efficient O(log N) pagination across all ref backends through sorted reference seeking.

**`sparse-checkout clean` command** -- Derrick Stolee proposed a new command to address performance issues from leftover sparse directories in cone mode, generating discussion about safety defaults and output granularity.

**`core.commentChar=auto` deprecation** -- Phillip Wood's proposal to deprecate the problematic auto-comment feature reached final agreement on simplified deprecation advice.

**Windows 11 regression** -- Fabio Frumento reported Git for Windows 2.50.0 failing to launch on Windows 11 Pro 64-bit, with the terminal window appearing then closing immediately.

**RHEL 6 compilation issues** -- Азат Усманов reported compilation failures when building Git 2.50+ on RHEL 6 systems, with workarounds documented despite the platform being officially unsupported.

**Line ending behavior clarified** -- Discussion confirmed `core.eol=lf` only normalizes line endings during staging while preserving original line endings during checkout.

**Git daemon signal handling** -- Carlo Marcelo Arenas Belón's series transitioning from `signal()` to `sigaction()` was approved after addressing Windows compatibility concerns.

**Promisor-remote capability extension** -- Christian Couder's v5 series extending the promisor-remote protocol to support configurable validation of remote attributes was prepared for merging.

**String-list modernization** -- shejialuo's v3 series modernizing string-list tests and implementation was approved for merging, converting all functional tests from shell to properly isolated C unit tests.

**Looking ahead**

The upcoming week will likely see continued discussion around several key topics:

- **Git 3.0 preparations**: With reftable becoming the default and SHA-256 support progressing, expect more discussion about breaking changes and migration paths.
- **Platform compatibility**: The RHEL 6 and Windows line ending discussions may prompt broader policy discussions about support requirements.
- **Community norms**: The ongoing conversation about response expectations and contribution workflows may yield formal guidelines.
- **Object database refactoring**: Follow-up work is needed to address the fast-import regression and continue the pluggable backend effort.