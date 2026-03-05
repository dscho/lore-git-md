# Git Mailing List Digest - 2025/09/03

**The day in brief.** A busy Wednesday with 90 emails across 25 threads, dominated by ongoing discussions about Rust adoption strategy, `git-history` refinements, and documentation improvements. The Rust infrastructure debate saw key contributions from Patrick Steinhardt and Junio Hamano outlining a potential transition path, while Julia Evans' `git-checkout` documentation series reached its final polishing stage.

## Notable threads

**Rust adoption strategy takes shape**  
Patrick Steinhardt proposed making Rust optional until Git 3.0 to allow for build system iteration and platform preparation time, responding to Brian Carlson's arguments for immediate adoption. Junio Hamano raised concerns about the limited areas where Rust features could be truly optional without either maintaining dual implementations or restricting Rust to narrow new features. Ramsay Jones added concrete evidence of platform impact, noting he's already stopped building Git on Cygwin due to Rust's unavailability there. The discussion revealed tensions between innovation and stability, with Steinhardt's gradual approach gaining traction but key implementation questions remaining unresolved.

**git-history subcommand refinements**  
Patrick Steinhardt's RFC series for the new `git-history` command saw multiple interface refinements. D. Ben Knoble reported issues with the `split` subcommand's diff visibility, while Kristoffer Haugsbakk identified an incorrect error message in `reword`. Steinhardt acknowledged these as valid bugs to address in future iterations. Knoble also proposed adding `squash` and `fixup` subcommands inspired by vim-fugitive's functionality, though this was framed as a future enhancement rather than a requirement for the current series. The thread demonstrated active engagement in shaping this new porcelain command's user experience.

**Documentation polish for git-checkout**  
Julia Evans' v3 series to improve `git-checkout.adoc` reached its final polishing stage after extensive review. Junio Hamano provided detailed line notes on phrasing, suggesting changes like replacing "left unchanged" with "changes follow you" for more intuitive descriptions of file preservation during branch switches. The discussion focused on balancing technical precision with accessibility, particularly around explaining merge conflict resolution options. With all major technical accuracy concerns resolved in earlier versions, this final round addressed subtle improvements to the pedagogical approach.

**Case-insensitive ref handling fixes**  
Karthik Nayak's series addressing case-sensitivity issues in the files backend saw thorough review from both Patrick Steinhardt and Junio Hamano. The patches modify error handling to allow partial application of non-conflicting references when case conflicts occur, rather than failing entire transactions. Hamano questioned whether the solution adequately distinguishes genuine case conflicts from other lockfile scenarios, suggesting additional verification logic. The changes represent a pragmatic compromise while consistently promoting reftable as the long-term solution for case-insensitive environments.

**Slab allocator API finalized**  
A bugfix series reworking Git's slab allocator API to prevent dangling pointer issues reached its final form after addressing Jeff King's observation about a technically incorrect NULL check in `alloc_state_free_and_null()`. The v4 patch corrects the defensive programming pattern where `*s_` was dereferenced before NULL check, eliminating potential undefined behavior. With all prior reviewers approving the technical approach, this refinement marked the last outstanding issue before merging.

## In brief

**Upload-pack protocol v2 fix** -- A two-patch series addresses duplicate ACKs being sent for non-commit objects when clients send repeated "have" lines, modernizing tests first then fixing the core protocol behavior.

**Ref transaction hook behavior** -- Jeff King clarified that showing zero OID for updates in reference transaction hooks is the documented behavior when no old value is specified to the transaction, though acknowledged this design makes it impossible to distinguish creates from force updates.

**Default branch test strategy** -- Phillip Wood and Patrick Steinhardt reached consensus on maintaining `GIT_TEST_DEFAULT_INITIAL_BRANCH_NAME` while completing the test suite's transition from "master" to "main" references.

**Worktree config typo fix** -- Documentation correction fixes `extension.relativeWorktrees` to `extensions.relativeWorktrees` in worktree.adoc, aligning with the actual config name.

**Midx-write error handling** -- Derrick Stolee's series standardizing error returns in `write_midx_internal()` saw discussion about initialization patterns but no blocking concerns, with the patches addressing a segfault bug and improving type safety.

## On the radar

**git whatchanged deprecation** -- Ongoing user feedback highlights challenges in transitioning from the deprecated command, with discussions about alias limitations and output format preferences continuing post-merge. Jeff King's proposal to allow aliases to override deprecated commands remains a potential solution.

**Header file placement debate** -- Junio Hamano's definitive stance that reusable components like `pack-refs.h` belong in the top-level directory may require additional implementation changes to properly relocate the corresponding `.c` file in the `git refs optimize` series.