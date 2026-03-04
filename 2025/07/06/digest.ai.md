# Git Mailing List Digest - 2025/07/06

**The day in brief.** A moderately busy Sunday with 53 emails across 11 threads, featuring significant progress on several fronts. The standout developments include a new `git repo-info` command proposal reaching v3, policy discussions around contributor identities, and test suite reorganization. Meanwhile, documentation refinements and bug reports kept the conversation lively.

## Notable threads

**New `git repo-info` command proposal reaches v3** -- Lucas Seiki Oshiro's GSoC project to create a dedicated repository metadata reporting tool advanced with a v3 series introducing the `git repo-info` command. The implementation now includes null-terminated output format (matching `git config -z`), removal of the `--allow-empty` flag, and initial field implementations for reference storage format, bare status, and shallow status detection. The command is designed as a machine-readable alternative to `git rev-parse` for repository-specific information, with a `<category>.<field>` query syntax and JSON/nul-terminated output options. The series appears well-structured with clear commit messages and documentation, though questions remain about future field expansion and command scope.

**Policy discussions around contributor identities** -- A significant policy discussion emerged from a technical thread about SSH signing tempfile leaks, focusing on Git's DCO sign-off requirements. brian m. carlson and Jeff King aligned on accepting pseudonyms in contributor identities, particularly for gender transition cases where legal names may be undesirable. The discussion showed consensus forming around updating Git's documentation to reflect more flexible naming policies while maintaining traceability. brian m. carlson committed to sending documentation patches to formalize this "identifiable but pseudonymous" framework, marking progress on an issue that has been percolating in open source communities.

**Test suite reorganization consolidates help tests** -- Two parallel 7-patch series (from different submitters) completed the consolidation of command help tests into `t1517-outside-repo.sh`. The changes relocate `-h` flag tests for seven commands (including `checkout-index`, `for-each-ref`, and `verify-tag`) from their original context-specific test files to a unified location. The mechanical moves follow Patrick Steinhardt's suggestion to group tests of command behavior outside repositories, maintaining identical verification of exit code 129 and usage message patterns. The reorganization demonstrates clean separation of concerns, particularly for commands like `verify-tag` that were previously tested in GPG-specific contexts despite their help behavior being format-independent.

## In brief

**Documentation formatting refinements** -- Jean-Noël Avila continued discussions about space notation in verbatim blocks and syntax parsing challenges in the git-log documentation conversion, highlighting tooling limitations in asciidoc.py versus asciidoctor.

**`git stash` message handling discrepancy** -- A bug report revealed that custom messages from `git stash create` appear in commit objects but not reflog entries during `git stash store`. Jeff King analyzed the distinction between commit and reflog messages, proposing solutions ranging from documentation clarification to behavioral changes.

**`git grep` first-match proposal** -- Markus Elfring suggested adding an option to show only the first match per file, with discussion revealing that `--max-count=1` may already provide this functionality, though use cases for seeing the line content (not just filename) remain.

**SSH signing tempfile leak fix** -- redoste's v2 patch addressing a memory management issue in SSH signing operations received positive review, completing the technical resolution while policy discussions about contributor identities continued separately.

**X-Change-ID header debate** -- Discussion continued about the reliability of email headers versus body content for tracking change IDs, with Drew DeVault arguing headers are technically correct despite infrastructure concerns raised by Aditya Garg.

**`git init` template documentation** -- Jeff King suggested improving documentation of `--template=` and `--no-template` behaviors, noting the empty string case remains relevant for config/environment variables despite the newer flag being more ergonomic.

## On the radar

**Documentation synopsis-style conversion** -- Jean-Noël Avila's ongoing effort to convert man pages continues generating discussion about micro-level formatting challenges, particularly around special character handling in the new AsciiDoc format.

**`git refs list` subcommand design** -- The thread about introducing a new plumbing command for ref listing awaits redesign to follow Junio Hamano's preferred wrapper pattern around `for-each-ref`, after initial confusion about implementation approach.