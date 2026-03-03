# Git Mailing List Digest - 2025/05/19

**The day in brief.** A moderately busy day with 82 emails across 19 threads, featuring significant progress on several technical fronts. Key developments include the completion of the promisor-remote protocol enhancements, final approval for batched reference updates, and ongoing discussions about build system standardization and test modernization. The day also saw the emergence of a new proposal to make `git reset --hard` safer by preserving state in the reflog.

## Notable threads

### Promisor-remote protocol enhancements finalized

Christian Couder's 5-patch v3 series enhancing the promisor-remote protocol with configurable validation of remote attributes has reached completion. The series introduces server-side field advertisement (`promisor.sendFields`) and client-side validation (`promisor.checkFields`) for partial clone filters and authentication tokens. Key improvements in this version include replacing string_list with direct struct members for better type safety, comprehensive documentation updates, and strict handling of unknown fields. The changes maintain backward compatibility while adding security validation capabilities, with all major design decisions settled through previous review. Junio Hamano's final review is the only remaining step before integration.

### Batched reference updates approved for 2.51

Junio Hamano has given final approval to Karthik Nayak's performance optimization series introducing batched reference updates to `git-fetch` and `git-receive-pack`. The changes show dramatic speedups - 22x faster fetches and 18x faster receive-pack operations for the reftable backend, with more modest 1.25x improvements for the files backend. The v3 iteration addressed all technical concerns around memory management and error handling, including fixing a memory leak in send-pack and implementing proper error handling for batched updates in receive-pack. Junio suggests targeting Git 2.51 to allow additional baking time given the high-impact nature of these changes.

### Build system standardization ready for integration

Ramsay Jones's build system standardization series has cleared all validation requirements after successful `make test` runs on Cygwin. The 5-part series aligns path handling across Git's build systems (Make, Meson, Autoconf), with key components including system-wide config path handling (`ETC_GITCONFIG`, `ETC_GITATTRIBUTES`), cross-build path standardization (`GIT_EXEC_PATH`, `GITWEBDIR`), and platform detection unification (sysinfo() checks). The v4 changes were minimal - removing redundant comments in meson_options.txt per Patrick Steinhardt's feedback - and the series now has all reviewer approvals for integration.

### Non-standard object type removal nears completion

Jeff King's 13-patch series removing support for non-standard object types is nearly complete after maintainer approval of all technical changes. The final patch eliminates `write_object_file_literally()` and its helper functions, completing the removal of write-side support for unknown object types. The only remaining open question is documentation approach for the `--allow-unknown-type` no-op change in `git cat-file`, where discussion continues about whether to add explicit deprecation notices. The series has built comprehensive test infrastructure (including new `test-tool zlib` helpers) to verify behavior after removing this legacy support.

### Proposal for safer `reset --hard` behavior

A new discussion emerged about making `git reset --hard` safer by automatically preserving the pre-reset state in the reflog. Josh Bleecher Snyder proposed the feature to prevent accidental data loss, with Brian m. carlson and Junio Hamano weighing in on implementation approaches. Key considerations include whether to use the stash mechanism (with potential script compatibility issues) versus direct reflog operations, and whether the behavior should be default or configurable. The thread represents early-stage discussion of a user-facing safety enhancement that could significantly impact Git's destructive operations workflow.

## In brief

**String-list test modernization** -- shejialuo's series converting string-list tests to C unit tests has completed review, addressing final feedback from Patrick Steinhardt about test isolation and commit message clarity. Jeff King provided historical context about removed string-list functionality.

**Merge-tree --quiet option approved** -- Elijah Newren's `--quiet` flag for `git merge-tree` (originally proposed as `--dry-run`) has been approved by Phillip Wood and queued by Junio Hamano, providing hosting platforms with efficient mergeability checking.

**Submodule configuration safety** -- K Jayatheerth's series preventing `.gitmodules` overwrites during path reuse needs final polish from splitting one combined patch into two focused changes per Junio's review.

**Email credential documentation** -- Aditya Garg's v4 series reorganizing email provider configuration documentation is ready for integration after addressing Junio's feedback about patch base references.

**MPTCP compatibility discussion** -- Junio Hamano questioned the need for Git-specific MPTCP support given the protocol's design for transparent operation, citing RFC 6897's backward compatibility guarantees.

## On the radar

**Thunderbird patch script future** -- The discussion about whether to update or remove the `contrib/thunderbird-patch-inline` script remains unresolved, with Junio suggesting silent users may exist despite lack of reported issues.

**Bitmap corruption testing** -- lidongyan's work on memory leak detection during corrupt bitmap loading is being reorganized per Jeff King's suggestion to use shell-based corruption via `dd` rather than custom C code.

**Midx packfile optimization** -- Jeff King and Junio Hamano agreed on an approach to cache failed pack lookups in the multi-pack-index to reduce syscalls, with implementation details being finalized.