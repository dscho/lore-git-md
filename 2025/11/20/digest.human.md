# Git Mailing List Digest — 2025/11/20

**The day in brief.** A busy Thursday with 62 emails across 20 threads, dominated by technical discussions around the new `git-history` command and a critical bugfix for config path handling. Key developments include Junio marking the `git-history` series as nearing completion and a segfault fix for `git config get --path` with optional paths.

## Notable threads

### `git-history` command approaches completion

Patrick Steinhardt's `git-history` series saw extensive review today, with Junio Hamano indicating it's at a "good stopping point" pending minor revisions. The discussion focused on:

- Infrastructure refactoring to separate add-patch and add-interactive subsystems
- Implementation of the `split` subcommand for interactive commit splitting
- Design questions around branch handling and index preservation during edits

Elijah Newren raised substantive concerns about multi-branch scenarios and commit metadata handling, suggesting these may require more than just minor adjustments. Junio advocated for an incremental approach, proposing to limit initial scope to single-branch cases while leaving room for future expansion. The thread shows productive collaboration between Steinhardt, Newren, and Phillip Wood, with the series now awaiting a final revision before potential integration.

### Config path handling segfault fixed

A serious bug in `git config get --path` with `:(optional)` prefixes was analyzed and fixed through coordinated effort:

- Jeff King identified the root cause in `git_config_pathname()`'s contract change
- Multiple affected callers were found beyond the original report
- Junio provided authoritative clarification of the intended behavior
- Two patches were posted to properly handle NULL returns for optional paths

The fix required careful consideration of interface design and thorough updates to callers across the codebase. D. Ben Knoble later noted the need to integrate the new test with Meson builds, which Junio promptly addressed.

### `git describe` traversal order fix

Ben Boeckel proposed fixing `git describe`'s commit traversal to use proper topological ordering rather than commit-date order. While technically correct, Jeff King identified performance concerns with the initial implementation:

- Current approach could be O(n²) without commit-graphs
- Suggested leveraging existing revision machinery for efficiency
- Discussion now focused on balancing correctness with performance

The fix would change tag descriptions in some cases (e.g., from "tag-release-7-g<hash>" to correct "tag-release-5-g<hash>") but may need optimization before merging.

## In brief

**Documentation warnings for `--committer-date-is-author-date`** — Kristoffer Haugsbakk's v2 patch adds strong warnings to `git-am` and `git-rebase` docs about violating Git's timestamp assumptions, now with final review approval.

**Windows pthread emulation fix** — Third version of a patch correcting `pthread_cond_init()` return value in Windows compatibility layer, now with accurate subject line.

**`git-repo-info` future enhancements** — Discussion of potential `--keys` option to list available field names for interactive use and shell completion, post-merge.

**Windows/MSVC Rust build fixes** — Ezekiel Newren and brian m. carlson coordinated fixes for library naming conventions in Rust infrastructure, with broader CI improvements planned.

**`git last-modified` big-endian issues** — Jeff King confirmed performance regression on s390x and NonStop platforms, with investigation ongoing into merge commit misattribution.

**Lisp userdiff driver discussion** — D. Ben Knoble argued for separate Scheme/Lisp drivers due to differing indentation and definition patterns.

## On the radar

**`git cp` feature request** — Ongoing discussion about adding a copy command to preserve history/blame, with debate about whether dynamic detection suffices or explicit tracking is needed.

**SHA-1/SHA-256 interoperability** — Waiting on Ezekiel Newren's `cargo-workspace` CI fixes before finalizing Rust infrastructure changes.

**Config error message bug** — New report that `git config set` suggests non-existent `--add` option when it should recommend `--append` for multi-valued keys.