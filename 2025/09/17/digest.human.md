Here's the Git mailing list digest for September 17, 2025:

## The day in brief
September 17 saw significant activity across multiple fronts in Git development, with 130 emails across 29 threads. The day was dominated by technical discussions around Rust integration, reference transaction handling, and documentation improvements, alongside several bugfixes nearing completion. Key highlights include the resolution of batched reference update issues on case-insensitive filesystems and ongoing coordination of Git's Rust adoption strategy.

## Notable threads

**Rust integration coordination challenges**  
The complex effort to integrate Rust into Git's codebase faced organizational hurdles as parallel patch series from Ezekiel Newren and Patrick Steinhardt revealed coordination gaps. Junio Hamano raised concerns about duplicate commits and conflicting version numbering between the two approaches. Elijah Newren outlined four potential resolution paths, while discussions also covered technical considerations like crate dependency management and Windows/MSVC compatibility. The thread reflects the growing pains of this major architectural shift, with build system changes, CI integration, and policy documentation all progressing simultaneously.

**Batched reference updates finalized**  
Karthik Nayak's v4 series fixing case-insensitive filesystem issues in batched reference updates received final approvals after addressing all reviewer feedback. The changes introduce new error types (`REF_TRANSACTION_ERROR_CASE_CONFLICT`, `REF_TRANSACTION_ERROR_NAME_CONFLICT`) and modify `lock_raw_ref()` to properly detect conflicts while allowing non-conflicting updates to proceed. The comprehensive solution handles four conflict scenarios and includes thorough test coverage, serving as an interim fix until the reftable backend becomes default for case-insensitive systems.

**Documentation reorganization**  
Julia Evans and others continued improving Git's documentation, with Junio Hamano approving her `git-checkout` man page updates for merging to 'next'. A separate effort restructured `git-push` documentation, splitting refspec syntax and push rules into dedicated sections for better readability. The changes follow established patterns of progressive disclosure, balancing technical precision with approachability for less experienced users.

**Command deprecation framework**  
Kristoffer Haugsbawk's v6 series implementing Git's command deprecation system addressed the final build system issue, properly excluding `git-whatchanged` from `BUILT_INS` when built with `WITH_BREAKING_CHANGES`. The changes complete the technical implementation with improved error messages, alias shadowing support, and comprehensive test coverage. The series now awaits final review before inclusion.

## In brief

**Shallow clone limitations** -- Elijah Newren clarified that commit hash-based `--shallow-exclude` is considered "flawed" in the current implementation due to technical constraints rather than fundamental protocol issues.

**Packfile store refactoring** -- Justin Tobler and Patrick Steinhardt discussed minor architectural points in the already-merged packfile store changes, focusing on encapsulation boundaries and function naming.

**git-history command design** -- SZEDER Gábor raised concerns about the potentially "scary" implications of automatically rewriting descendant commits and repointing branches in the proposed `git-history drop` subcommand.

**git-pull shorthand proposal** -- Julien Jerphanion provided a concrete use case (conda-forge workflows) justifying the `-u` shorthand for `--set-upstream` in `git pull`.

**Signature handling in fast-import** -- Christian Couder's v3 series added signature parsing controls to match fast-export's capabilities, with only minor design questions about mode naming remaining open.

**Unstaging command recommendations** -- Discussion continued about whether `git restore --staged` or `git reset` provides better UX for unstaging files, with a new proposal to make `restore` work consistently in unborn repositories.

## On the radar

**Rust adoption timeline** -- The planned transition (optional in 2.52, default in 2.53, mandatory in 3.0) now faces questions about SHA-1/SHA-256 interoperability implications for platforms without Rust support.

**git fetch --dry-run behavior** -- A bug report revealed that `--dry-run` with `--filter` unexpectedly modifies local config, sparking discussion about dry-run semantics with partial clones.

**Worktree safety checks** -- Gabriel Scherer proposed a new `--detach-other-worktrees` option as a middle ground between strict safety checks and permissive `--ignore-other-worktrees` behavior.