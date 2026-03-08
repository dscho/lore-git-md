# Git Mailing List Digest - 2025/11/20

**The day in brief.** A busy Thursday with 62 emails across 20 threads, dominated by technical discussions around the new `git-history` command, a segfault fix for config path handling, and ongoing Rust infrastructure work. Key developments include Junio marking the `git-history` series as nearing completion and a critical fix for Windows/MSVC builds in the Rust interoperability work.

## Notable threads

### `git-history` command approaches completion

Patrick Steinhardt's `git-history` series is now in its sixth iteration and has reached a "good stopping point" according to Junio Hamano, pending minor revisions. The series introduces new history editing capabilities including `reword` and `split` subcommands, with Elijah Newren and Phillip Wood providing extensive review feedback on both functionality and infrastructure changes. 

Key technical discussions centered on:
- The implementation of `run_add_p_index()` and its relationship to existing add-patch code
- Design questions around branch handling (single vs. multi-branch support)
- Index preservation behavior during commit splitting
- Commit metadata handling in edited commits

While the core functionality has consensus, Elijah Newren raised several substantive concerns that may require more than just minor adjustments, particularly around multi-branch handling and index state preservation. Junio suggested an incremental approach, starting with single-branch support and expanding later.

### Config path handling segfault fixed

A serious segfault in `git config get --path` when handling optional path values (`:(optional)`) was diagnosed and fixed through a two-patch series. The root cause was that `git_config_pathname()` could return success without setting the destination pointer for optional paths, which callers weren't prepared to handle. 

Jeff King's analysis revealed multiple affected callers beyond the original report, including `blame.ignorerevsfile` handling which could trigger double-free issues. The fix makes `git_config_pathname()` explicitly clear the destination pointer for missing optional paths and teaches `format_config()` to properly handle this case. 

Junio later clarified the intended behavior with multiple `:(optional)` declarations, confirming each should be treated independently. The discussion revealed deeper interface design questions about how optional paths should behave in various contexts.

### Rust infrastructure and Windows/MSVC builds

The SHA-1/SHA-256 interoperability work ran into Windows/MSVC build issues that were traced back to library naming conventions. brian m. carlson proposed a fix distinguishing between MSVC and GNU toolchain environments on Windows, while Ezekiel Newren revealed he's working on a broader `cargo-workspace` series to comprehensively fix GitHub CI problems.

The key technical issue was that Windows builds require different library naming conventions (`.lib` for MSVC vs. `.a` for GNU) and current CI only tests two Windows toolchain combinations. The thread shows clear division of labor with carlson handling core functionality while Newren tackles CI infrastructure.

## In brief

**`git describe` topological ordering fix** -- Ben Boeckel proposed a fix for `git describe`'s commit traversal order, addressing an issue where it doesn't follow topological ordering. Jeff King suggested optimizations using generation numbers or existing topological sorting infrastructure to avoid performance regressions.

**Documentation warnings for `--committer-date-is-author-date`** -- Kristoffer Haugsbakk's v2 patch adds strong warnings about this option in both `git-am` and `git-rebase` documentation, noting it can create commit histories that violate Git's timestamp ordering assumptions.

**Windows pthread emulation fix** -- A third version of a patch correcting the return value of `pthread_cond_init()` in Git's Windows pthread compatibility layer to be more strictly POSIX-compliant.

**`git-repo-info` future enhancements** -- After approval of the GSoC project, discussion turned to potential future additions like a `--keys` option to list available field names for better discoverability.

**Performance regression on big-endian** -- Jeff King investigated test failures in `git last-modified` on s390x and HP NonStop, suggesting the algorithm may not properly propagate blame through merges on big-endian architectures.

**`git cp` feature request** -- Martin Guy proposed a new command to handle file copying while preserving history/blame information, sparking discussion about Git's existing copy detection capabilities versus explicit tracking needs.

## On the radar

**Static function analysis** -- Junio made `git_configset_get_pathname()` static, sparking broader discussion about identifying functions that could be made static, with notes that some hash functions must remain public for Rust FFI.

**Submodule `--single-branch` behavior** -- Discussion continues about whether `--single-branch` should default to the branch specified in `.gitmodules` when initializing submodules, rather than just cloning HEAD.