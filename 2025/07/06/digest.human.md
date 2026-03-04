# Git Mailing List Digest - 2025/07/06

**The day in brief.** A moderately busy Sunday with 53 emails across 11 threads, featuring significant progress on several fronts. The standout developments include a new `git repo-info` command reaching v3, policy discussions around contributor identities, and multiple test reorganization efforts. Documentation refinements and bugfix discussions rounded out the day's activity.

## Notable threads

### New `git repo-info` command reaches v3

Lucas Seiki Oshiro's GSoC project to create a dedicated repository metadata reporting tool advanced to its third iteration with a comprehensive 7-patch series. The command provides machine-readable access to repository properties currently scattered across `git rev-parse` and other commands. The v3 implementation introduces a null-terminated output format (matching `git config --list -z` syntax) alongside JSON, removes the `--allow-empty` flag, and adds initial field implementations for reference storage format, bare status, and shallow repository detection. The series shows careful attention to output formats and field organization, with thorough build system integration and test coverage in t1900-repo-info.sh. While marked as experimental, this represents significant progress toward a more structured approach to repository introspection.

### Policy discussion on contributor identities

A policy discussion emerged from the SSH signing bugfix thread, focusing on contributor identity requirements in Git's DCO sign-offs. brian m. carlson and Jeff King aligned on accepting pseudonyms in cases where contributors have valid reasons (such as gender transition considerations) while maintaining traceability by prohibiting truly anonymous contributions. The discussion showed consensus forming around updating Git's documentation to reflect more flexible naming policies, with brian m. carlson committing to send documentation patches. This represents an important step in making the project more inclusive while preserving accountability, following Linux kernel's precedent of moving from "real name" to "known identity" requirements.

### SSH signing tempfile leak fix finalized

redoste's bugfix for a tempfile leak in SSH signing operations reached its final form with v2 of the patch. The fix ensures proper cleanup by using `xstrdup()` rather than `strbuf_detach()` in `sign_buffer_ssh()`, maintaining symmetry with other code paths. The patch includes improved test coverage using `find` in an isolated `tmpdir` for verification, addressing prior feedback about test methodology. brian m. carlson provided a positive review, marking the technical resolution of this memory management issue. The parallel policy discussion about contributor identities (which originated in this thread) continues separately with promised documentation updates.

## In brief

**Documentation formatting discussion** -- Jean-Noël Avila and Kristoffer Haugsbakk continued refining documentation presentation, debating the `␣` symbol for explicit space notation in verbatim blocks while acknowledging toolchain limitations in handling special characters.

**`git stash` message handling discrepancy** -- Jeff King analyzed why custom messages from `git stash create` appear in commit objects but not reflog entries, proposing either documentation clarification or behavioral changes to `stash store`.

**`git grep` first-match proposal** -- Markus Elfring suggested showing only the first match per file, with Phillip Wood noting the existing `--max-count=1` option may already provide this functionality.

**Test reorganization series** -- A 7-patch series completed consolidating help tests for seven commands (`checkout-index`, `for-each-ref`, etc.) into t1517-outside-repo.sh, following Patrick Steinhardt's suggestion for better test organization.

**X-Change-ID header debate** -- Aditya Garg raised email provider compatibility concerns about Drew DeVault's proposed change ID tracking, prompting discussion about header reliability versus body-based alternatives.

**`git init` template documentation** -- Jeff King suggested documenting both `--template=` (empty path) and `--no-template` behaviors for skipping template copying during repository initialization.

## On the radar

**DCO policy documentation** -- brian m. carlson's promised update to formalize Git's acceptance of pseudonyms in contributor identities, following the productive discussion in today's SSH signing thread.

**`git refs list` subcommand** -- Shejialuo's late clarification in the ongoing discussion about whether to create a new command or better document `for-each-ref` usage patterns, aligning with Junio's wrapper preference.