# Git Mailing List Digest - 2025/10/18

**The day in brief.** A moderately active Saturday with 31 emails across 9 threads, dominated by technical discussions around diff output handling and submodule behavior. The most notable developments include converging solutions for the `git diff --quiet` regression and a new series making `git add` respect `ignore=all` submodule configurations. Several documentation refinements also progressed toward completion.

## Notable threads

### `git diff --quiet` regression fix approaches consensus

The thread exploring fixes for a regression in `git diff --quiet` output suppression (introduced in commit b55e6d36eb) reached technical consensus today. Jeff King and Lidong Yan worked through the tradeoffs between using `/dev/null` redirection versus the `dry_run` optimization path, with Junio Hamano ultimately suggesting a phased approach: apply Jeff's conservative `/dev/null` fix to `maint` immediately while continuing to improve the `dry_run` safety on `master`. The discussion revealed careful consideration of both correctness (ensuring no output leaks) and performance (preserving early termination optimizations where possible).

### `git add` to respect `ignore=all` submodule setting

A new five-patch series from an unlisted contributor proposes making `git add` respect the `ignore=all` submodule configuration, aligning its behavior with `status` and `diff`. The changes introduce an `ignored_too` parameter (equivalent to `--force`) in the cache update machinery, with comprehensive test coverage added in t2206. This addresses a long-standing inconsistency where submodules could be accidentally added despite being configured to ignore changes. The implementation appears thorough, modifying core read-cache functionality while maintaining backward compatibility.

### `git grep` binary file debate evolves

The discussion about whether `git grep` should ignore binary files by default shifted toward a general attribute-based filtering mechanism. Jeff King referenced a 2012 "filetype" proposal as a more flexible solution than binary-specific toggles, which Junio Hamano endorsed as promising. This moves beyond the initial debate sparked by Randall S. Becker and Brian m. carlson's concerns about breaking workflows that legitimately search binary files (SQL dumps, PDFs with text layers). The thread now focuses on designing a comprehensive attribute filtering system rather than changing defaults.

## In brief

**Color handling in `git status --short -z`** -- Langbart and Jeff King confirmed the recent fix works well for real-world use cases like piping colored output to `fzf --read0`, validating the solution's practical utility.

**Documentation wording refinements** -- Queen Jessa addressed a line length issue in the MyFirstContribution.adoc guide, while Ben Knoble and Junio Hamano continued polishing terminology in the `git-reset` man page overhaul regarding `--patch` mode descriptions.

**SSH agent test security** -- Jeff King acknowledged but downplayed a command injection risk in the test suite's unquoted `ssh-agent` evaluation, noting an attacker controlling $HOME already has significant access.

## On the radar

**Dry-run output suppression** -- Lidong Yan's comprehensive fix for diff output in dry-run mode is progressing through review, with Jeff King now approving the approach despite preferring a two-step solution. The changes may land soon after final test coverage verification.

**Submodule ignore behavior** -- The new series making `git add` respect `ignore=all` settings bears watching as it enters review, as it touches core cache functionality and affects workflows for projects using submodules extensively.