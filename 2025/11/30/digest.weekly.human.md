# Git Mailing List Digest - 2025/11/24 -- 2025/11/30

**The week in brief.** A busy week with 248 emails across 89 threads saw significant progress on multiple fronts. Key developments included the completion of Adrian Ratiu's hook subsystem refactoring, major design decisions around `git replay --revert`, and the introduction of Rust infrastructure changes via RFC. The week also featured important fixes for `git last-modified`, resolution of the `git clone` config parsing debate, and Johannes Schindelin's Windows symlink compatibility series. Junio Hamano's "What's cooking" reports provided steady guidance as multiple series reached maturity.

## Key developments

### Hook subsystem modernization completes

Adrian Ratiu's 10-part series refactoring Git's hook subsystem reached completion this week after addressing critical memory leaks identified by Junio Hamano. The changes standardize hook execution across Git, introducing callback-based stdin handling and output capture capabilities while maintaining backward compatibility. The final iteration fixed memory management in receive-pack hook conversions, simplified receive hook stdin callback logic, and improved documentation for the new hook.h and run-command.h interfaces. This foundational work enables future features like config-based hooks and parallel execution, with the series now awaiting final procedural adjustments before merging.

### `git replay` gains revert capability

Siddharth Asthana introduced server-side revert functionality for `git replay`, designed primarily for GitLab's Gitaly service to avoid client-side roundtrips. The implementation reuses cherry-pick infrastructure by swapping base and pickme trees in merge operations. After extensive review from Junio Wood and Elijah Newren, the design converged on making `--revert` a distinct operation mode rather than a combinable flag. The thread also surfaced important questions about naming the operation modes, with Junio Hamano suggesting `--rebase`/`--cherry-pick` as clearer alternatives to the current `--onto`/`--advance` names. The feature represents an important expansion of `git replay`'s capabilities but will need to coordinate with Patrick Steinhardt's parallel sequencer refactoring work.

### Rust infrastructure RFC proposed

Ezekiel Newren proposed a significant restructuring of Git's Rust infrastructure into a Cargo workspace architecture (13-patch series). The RFC aims to improve modularity and compilation efficiency by splitting the Rust code into multiple crates (gitcore, link-with-c, generate-headers) rather than maintaining a single monolithic crate. The series undoes some of Patrick Steinhardt's prior Rust work and introduces new build system integration, though it remains incomplete with known issues around Windows builds and CI workflows. This proposal represents a major architectural shift that would affect how Rust integrates with Git's build system, with the RFC status indicating the author is seeking feedback before finalizing the approach.

### `git last-modified` cross-platform fixes

Toon Claes addressed critical issues in the recently merged `git last-modified` optimization that were causing test failures on s390x and NonStop systems. The bug, initially thought to be big-endian specific, was actually an algorithmic problem in merge commit handling that affected all platforms. The fix corrects how merge commit parent ordering is processed in the generation-sorted priority queue while preserving the command's 2.5x-111.7x performance gains. A separate follow-up patch fixed bitmap initialization that was causing memory corruption, particularly visible on s390x. These fixes demonstrate Git's commitment to robust cross-platform support even for performance-sensitive features.

### Windows symlink compatibility preparation

Johannes Schindelin submitted a comprehensive 10-patch series preparing Git's test suite for MSYS2's upcoming default enablement of symlink support on Windows. The changes address various test failures that would occur when symlinks become fully functional, including Perl test path handling fixes, `git apply` executable bit regressions, and platform-specific handling of `/dev/null` symlinks. With MSYS2 planning to enable symlink support by default, these preemptive fixes will help ensure Git's test suite remains reliable across platforms. Junio Hamano praised the solutions as "clever and clean," particularly the approach to handling Windows path formats in t7800-difftool.sh.

## In brief

**ASan hardening** -- Jeff King's 12-part series introducing robust integer parsing functions concluded with buffer-safe utilities now integrated into core components like fsck.c and cache-tree.

**`git clone` config parsing** -- The whitespace-trimming regression debate concluded with the stricter parsing being deemed an "inadvertent bugfix" rather than a regression, establishing new expected behavior.

**Reference backend URIs** -- Consensus emerged on using `GIT_REF_URI` environment variables for temporary reference store overrides, with remaining questions about URI format syntax (`://` vs `:`).

**Documentation standardization** -- Jean-Noël Avila completed converting `git fetch`, `git pull`, and `git push` man pages to consistent AsciiDoc synopsis style across 10 files (800+ lines).

**Lisp userdiff driver** -- Scott Burson's v2 series introducing a unified "lisp" userdiff driver achieved technical resolution after addressing feedback about whitespace handling and backward compatibility.

**Scalar configuration** -- Derrick Stolee added comprehensive documentation for Scalar's configuration approach while removing stale config values, with Junio Hamano calling it "superbly written."

**Performance optimizations** -- René Scharfe introduced a new `diff_same()` function that optimizes `diff-index` performance when using `--find-copies-harder`, demonstrating a 1.33x speedup.

## Looking ahead

**Rust infrastructure debate** -- Ezekiel Newren's RFC series will likely generate significant discussion about Git's Rust integration strategy, particularly around platform support and build system implications.

**Reference backend migration** -- With the environment variable approach now accepted, attention turns to finalizing the URI format syntax as Karthik Nayak's series nears completion.

**Parallel hook processing** -- The foundational refactoring is complete, leaving the door open for follow-up work on parallel execution capabilities enabled by the new hook API.

**`git replay` enhancements** -- The basic revert functionality is taking shape, with future work expected on disconnected commit handling and possible operation mode renaming.