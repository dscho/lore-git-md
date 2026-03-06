# Git Mailing List Digest - 2025/10/18

**The day in brief.** A moderately busy Saturday with 31 emails across 9 threads, dominated by technical refinements to the `git diff --quiet` regression fix and discussions around `git grep`'s handling of binary files. The most notable developments include consensus forming on the diff output suppression approach and a promising direction emerging for attribute-based file exclusion in grep.

## Notable threads

### `git diff --quiet` regression fix approaches consensus

The discussion around fixing the `--quiet` regression in Git's diff machinery reached technical consensus today. Jeff King and Lidong Yan worked through implementation details, with Junio Hamano providing architectural guidance. The agreed approach involves:

1. An immediate fix for `maint` using `/dev/null` redirection to suppress output
2. Maintaining the `dry_run` optimization path for performance
3. Potential future simplification once the safety mechanisms prove reliable

Junio outlined a phased deployment strategy, suggesting the `/dev/null` workaround could remain indefinitely with a `NEEDSWORK` comment if needed. This careful approach balances immediate correctness with long-term code health, showing the project's maturity in handling subtle behavioral regressions.

### `git grep` binary file debate evolves toward attributes

What began as a proposal to make `git grep` ignore binary files by default has evolved into a discussion about more general attribute-based filtering. Jeff King referenced a 2012 "filetype" concept that could provide flexible exclusion mechanisms without breaking existing workflows. Junio Hamano endorsed this direction, noting that binary files are just one case of many that users might want to exclude.

The thread revealed real-world use cases where binary file searching is valuable (SQL environments, PDFs with text, XMP metadata) while pointing toward a more comprehensive solution. This shift from a binary-specific toggle to attribute-based approach shows the community's preference for extensible solutions over narrow fixes.

### Submodule `ignore=all` behavior made consistent

A new patch series from an unlisted contributor (email address redacted) aims to make `git add` respect the `ignore=all` submodule setting consistently with other commands. The five-part series:

1. Adds `ignored_too` parameter plumbing
2. Implements core ignore logic
3. Adds comprehensive tests
4. Updates existing tests
5. Documents the behavior

This brings `git add` in line with `status`, `diff`, and `log`, requiring `--force` to stage changes to ignored submodules. The thorough implementation and test coverage suggest this well-motivated change has good prospects for acceptance.

## In brief

**Color handling in `git status --short -z`** -- Follow-up confirmations show Jeff King's fix enables practical workflows like piping colored output to `fzf --read0`, validating the approach of preserving colors in this machine-readable format.

**SSH agent test security** -- Jeff King acknowledged an unquoted evaluation vulnerability in test setup but noted limited practical risk since attackers controlling $HOME already have significant access.

**`git-reset` documentation refinements** -- Final wording tweaks continue for Julia Evans' man page overhaul, focusing on precise terminology for `--patch` mode behavior descriptions.

**Spam alert** -- An off-topic conference invitation was mistakenly sent to the list, promptly ignored by contributors.

## On the radar

**Dry-run output suppression** -- While consensus is reached on the approach, the comprehensive fix remains pending integration through Git's normal workflow after the immediate regression fix hits `maint`.

**Attribute-based file filtering** -- The promising "filetype" concept from 2012 may see renewed interest as a solution for `git grep` exclusion needs, though implementation complexity remains to be explored.