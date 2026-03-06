# Git Mailing List Digest - 2025/10/19

## The day in brief

A moderately active Sunday with 16 emails across 8 threads, featuring continued refinement of the dry-run diff fixes, substantive discussion about Scalar's configuration strategy, and several smaller improvements to SSH key handling and bash completion. The most notable developments were the v4 patchset completing the dry-run diff implementation and Matthew Hughes' probing questions about Scalar's configuration defaults.

## Notable threads

### Scalar configuration rationale examined

Matthew Hughes advanced the Scalar configuration discussion by shifting focus from technical implementation to substantive review of which settings Scalar should actually modify. His email identified several potentially outdated or Windows-specific configs (core.FSCache, credential.validate) and probed the reasoning behind CRLF handling, GC automation disablement, and index optimizations. This line of questioning could lead to meaningful simplifications in Scalar's configuration strategy as the project moves beyond its Windows origins.

### Dry-run diff fixes reach completion

The multi-day effort to properly implement dry-run behavior in Git's diff machinery reached resolution with Lidong Yan's v4 patchset. Following Junio Hamano's guidance to separate the immediate regression fix (using /dev/null redirection) from the comprehensive solution, this series now suppresses all output paths during dry-run operations. The implementation includes thorough test coverage for edge cases like file deletions and unmerged paths. Johannes Schindelin contributed additional analysis using CodeQL to validate the solution's completeness.

### SSH key handling modernization

Olamide Caleb Bello's v3 series to replace `strbuf_split*()` with `string_list_split*()` in SSH key operations received final review from Junio Hamano. While approving the technical correctness of the conversion, Junio mused about whether even simpler string operations might suffice. The patches modernize fingerprint parsing in `gpg-interface.c` while carefully maintaining existing behavior, representing incremental progress in the broader effort to use more appropriate string handling APIs.

## In brief

**Bash completion improvements** -- KIYOTA Fumiya enhanced tab completion for `git log` and `git shortlog`, adding the missing `--exclude=` option and properly separating `--committer` handling between commands. Junio noted this highlights an underlying UI inconsistency worth future discussion.

**`git diff --stat` path display proposal** -- A feature request emerged to add configuration options preventing path truncation in `--stat` output, addressing workflow issues when copying full paths from diff output.

**Submodule ignore behavior review** -- Phillip Wood provided substantive feedback on a patch changing `git add`'s handling of `ignore=all` submodules, questioning the use of `--force` and noting historical context from commit 5556808.

## On the radar

**Scalar configuration provenance** -- While today's discussion focused on which configs to keep, Patrick Steinhardt and Derrick Stolee continue working on the technical implementation for tracking and documenting Scalar's configuration changes. This foundational work will support future maintenance of these settings.