# Git Mailing List Digest - 2025/11/05

**The day in brief.** A moderately busy day with 55 emails across 16 threads, featuring several significant developments. The atomic ref updates for `git replay` reached merge-ready status, Junio Hamano advanced his incomplete-line whitespace detection series to v2, and discussions continued on extended attribute reporting in `git diff`. The day also saw progress on signature handling in `git fast-import` and trailer support for `git rebase`.

## Notable threads

**Atomic ref updates for `git replay` merge-ready**  
Siddharth Asthana's series implementing atomic reference updates in `git replay` reached its final form with v7, addressing all remaining review feedback. The changes standardize `git replay`'s behavior to perform atomic updates by default while maintaining backward compatibility through a `--ref-action=print` option. The implementation uses Git's ref transaction API (`ref_store_transaction_begin`/`_commit`) and includes comprehensive tests for both CLI and config-driven behavior. With maintainer approval from Junio Hamano and positive reviews from Elijah Newren and Christian Couder, the series is now queued for merging, representing a significant improvement in `git replay`'s robustness.

**Incomplete-line whitespace detection v2**  
Junio Hamano posted version 2 of his 12-part series introducing configurable detection of files missing terminating newlines. The updated series includes fixes for line number reporting in `git apply --check`, expanded test coverage, and better documentation of the new `WS_INCOMPLETE_LINE` bitmask (0x8000). The implementation spans both diff and apply sides of Git's pipeline, with the final patch enabling enforcement across Git's own codebase via `.gitattributes`. The thorough preparatory refactoring and comprehensive test coverage suggest this feature is nearing readiness, though it may see another revision based on feedback.

**Extended attribute reporting in `git diff`**  
The discussion about extending `git diff --raw` to report file attributes like binary status continued, with Justin Tobler and Ben Knoble weighing in on output design considerations. The thread has shifted from a binary-specific solution toward designing a general attribute reporting framework, with active debate about whether to use key=value pairs ("binary=tt") or positional letter codes ("tt,ic") for machine parsing. Junio Hamano expressed concerns about the parseability of key=value formats, keeping the discussion open-ended as participants seek an optimal balance between human readability and script-friendliness.

**Signature handling in `git fast-import`**  
Christian Couder proposed a new 'strip-if-invalid' mode for `git fast-import`'s `--signed-commits` option, designed to help tools like `git filter-repo` handle invalid signatures during history rewriting. The three-part series includes careful refactoring of commit buffer handling and signature verification before introducing the new mode. However, Junio Hamano raised significant concerns about the cryptographic implications of signature stripping during history rewriting, particularly around key availability and timestamp validity. These questions suggest the feature may need additional design consideration before proceeding.

**Trailer support for `git rebase`**  
A four-part series from Kristoffer Haugsbakk added `--trailer` support to `git rebase`, building on preparatory refactoring to move trailer processing in-process rather than forking to `interpret-trailers`. The implementation currently works only with the merge backend and includes thorough validation of trailer syntax. Junio Hamano provided detailed feedback on the API design, particularly around documentation and parameter naming for the newly public `process_trailers()` function. The series appears well-considered but may see another revision to address the maintainer's concerns about interface clarity.

## In brief

**HTTP protocol documentation finalized** -- A documentation patch specifying server error responses for invalid `want` lines was approved, completing a small but precise improvement to Git's protocol specifications.

**`git add` exclude patterns discussion** -- Junio Hamano raised architectural questions about pathspec handling in `dir.c`, suggesting the current approach might need deeper refactoring rather than just bugfixes.

**Delta pager bug identified** -- The thread about `git grep -l` displaying dashes as colons was traced to a known Delta pager bug (#1259), shifting focus from Git itself to pager interaction issues.

**Windows credential helper updates** -- The wincred credential helper's Makefile was updated to use modern conventions like `LDFLAGS` and `gitexecdir`, bringing it in line with project standards.

**Reflog error message formatting** -- Peter Krefting fixed missing spaces in error messages from both files and reftable ref backends, a trivial change approved for the upcoming release.

## On the radar

**Git data model documentation** -- Julia Evans' comprehensive `gitdatamodel.adoc` is in final polishing stages, with ongoing discussion about how to best visualize the distinction between commit objects and their hexadecimal names.

**Reftable optimization checks** -- Karthik Nayak's 'is-needed' subcommand for `git maintenance` is technically complete but awaits resolution of dependency chain logistics before merging.