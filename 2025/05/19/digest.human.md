# Git Mailing List Digest - 2025/05/19

**The day in brief.** A moderately active day with 82 emails across 19 threads, featuring significant progress on several technical fronts. Key developments include the completion of the promisor-remote protocol enhancement series, finalization of batched reference updates for performance gains, and ongoing discussions about build system standardization. The day also saw healthy debate about policy decisions around contrib script maintenance and a new proposal to make `reset --hard` safer.

## Notable threads

### Promisor-remote protocol enhancements finalized

Christian Couder's 5-patch v3 series enhancing the promisor-remote protocol with configurable validation of remote attributes has reached completion after extensive review. The changes introduce server-side field advertisement (`promisor.sendFields`) and client-side validation (`promisor.checkFields`) for partial clone filters and authentication tokens. Key improvements in this version include replacing string_list with direct struct members for better type safety, comprehensive documentation updates, and strict handling of unknown fields. The series has addressed all technical feedback and awaits only Junio's final review before integration.

### Batched reference updates ready for 2.51

Karthik Nayak's performance optimization series introducing batched reference updates to `git-fetch` and `git-receive-pack` has been approved for integration targeting Git 2.51. The v3 iteration shows impressive speedups - 22x faster fetches and 18x faster receive-pack operations for the reftable backend. Junio Hamano gave final approval after all technical issues were resolved, including memory leak fixes and refined error handling. The changes build on commit 23fc8e4f61 and represent a major optimization for operations involving many references.

### Build system standardization complete

Ramsay Jones's build system standardization series has cleared all validation requirements after successful testing on Linux, Cygwin, and Solaris. The 5-part v4 series aligns path handling between Make and Meson builds, particularly for system-wide config files (`ETC_GITCONFIG`, `ETC_GITATTRIBUTES`). The final changes removed redundant comments in meson_options.txt per Patrick Steinhardt's feedback. With all reviewers satisfied and cross-platform testing complete, Junio has queued the series for integration.

### Non-standard object type removal progresses

Jeff King's 13-patch series removing support for non-standard object types is nearing completion, with only minor documentation questions remaining about the deprecation of `--allow-unknown-type` in `git cat-file`. The technical work is done, having simplified fsck handling, removed write-side support, and introduced test infrastructure for low-level object manipulation. Junio has approved the technical approach, leaving just the final decision about how explicitly to document the no-op status of the deprecated option.

## In brief

**String-list test modernization** -- shejialuo's series converting string-list tests to C unit tests received final polish suggestions from Patrick Steinhardt, focusing on test isolation and commit message clarity.

**Submodule configuration safety** -- K Jayatheerth's submodule safety patch needs splitting into two focused changes per Junio's review - one for path reuse protection and another for active flag optimization.

**Merge-tree --quiet approved** -- Elijah Newren's `--quiet` flag for merge-tree (providing efficient mergeability checking) has been approved after evolving through multiple naming iterations (`--mergeability-only` -> `--dry-run` -> `--quiet`).

**Thunderbird script policy debate** -- Discussion continues about whether to update or remove the thunderbird-patch-inline contrib script, with Patrick Steinhardt and Collin Funk favoring removal while Junio suggests considering silent users.

**Email documentation reorganized** -- Aditya Garg completed the consolidation of email credential helper documentation into git-send-email.adoc, removing the last vestiges from gitcredentials.adoc.

**MyFirstContribution tutorial updated** -- K Jayatheerth's series modernizing the new contributor tutorial with current API patterns has been queued by Junio after final whitespace fixes.

## On the radar

**Reset --hard safety proposal** -- A new discussion emerged about making `git reset --hard` automatically preserve state, with Junio cautioning against using the stash mechanism by default and suggesting reflog-based approaches instead.

**MPTCP support debate** -- Junio weighed in on the MPTCP discussion, questioning whether Git needs explicit protocol awareness given RFC 6897's backward compatibility guarantees.

**Bitmap corruption tests** -- The test infrastructure for detecting memory leaks during corrupt bitmap loading is being reorganized per Jeff King's suggestion to use shell-based corruption via `dd` rather than custom C code.