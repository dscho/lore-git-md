# The Git Project -- Weekly Digest for 2026/08/24 -- 2026/08/30

## The period in brief

This week (2026/08/24--2026/08/30) saw sustained high-volume traffic on the Git mailing list, with six active days and a mix of routine patch flow and architectural debates. The most consequential developments were the resolution of the `git history squash` autosquash marker policy, the graduation of the `receive-report` hook to `seen`, and a sweeping proposal to redesign built-in command interfaces. Two long-running series -- the trace2 hardening effort and the ODB abstraction work -- faced deeper architectural scrutiny, while a use-after-free regression in `git stash show` emerged as an urgent production concern.

## Key developments

### `git history squash` autosquash marker resolution policy settled
The `git history squash` series by Harald Nordgren reached a critical milestone when Junio C Hamano reaffirmed the project’s direction toward strict case-sensitive matching for `fixup!`/`squash!`/`amend!` markers. The decision requires a one-line mechanical change in Patch 7/8 -- replacing `istarts_with()` with `starts_with()` -- and removes the last technical blocker for the series. The implementation now aligns with Git’s historical convention of emitting only lowercase hexadecimal OIDs, avoiding the ambiguity of mixed-case matching.

The series aims to efficiently collapse commit ranges into their oldest commit while preserving descendant history, addressing a long-standing pain point for users who currently rely on `git rebase --autosquash` with its repeated conflict stops. The v15 reroll incorporating the `starts_with()` change and test coverage is expected shortly, with graduation to `next` likely in the coming week.

### `receive-report` hook graduates to `seen`
Karthik Nayak’s three-patch series introducing a `receive-report` hook for `git-receive-pack` was queued in `seen` after resolving all prior feedback, including the hook naming objection (renamed from `report` to `receive-report`) and exit status semantics. The hook enables server administrators to intercept and modify the status report sent to clients after ref updates, addressing GitLab’s need for MVCC (multi-version concurrency control). The hook runs after all ref updates are committed but before the status report is sent, receiving the pkt-line encoded report on stdin and replacing it with its stdout. A non-zero exit status rewrites all ref status lines to `"receive-report hook failed"` and discards the hook’s stdout, matching the behavior of the `pre-receive` hook.

The series touches `builtin/receive-pack.c`, `Documentation/githooks.adoc`, `Documentation/git-receive-pack.adoc`, and a new test script `t/t5412-receive-report-hook.sh`. It is technically complete and awaits graduation to `next`.

### Architectural redesign of built-in command interfaces proposed
Junio C Hamano proposed a sweeping architectural change to built-in command interfaces, submitting an 8-patch series that replaces the `struct repository *` parameter in all built-in commands (`cmd_foo()`) with a `bool has_repo` flag. The proposal also refactors `git checkout`, `git switch`, and `git restore` to eliminate the monolithic `checkout_main()` helper and extract reusable logic into standalone helpers. The design addresses segfault risks (e.g., `cd / && git foo -h`), clarifies the architectural boundary between built-ins and reusable utility code, and redirects the `the_repository` removal effort toward utility code outside `builtin/`.

The series touches 135 files and demonstrates the approach by refactoring the checkout/switch/restore codebase, moving reusable logic (e.g., `post_checkout_hook()`) to top-level files like `checkout.c` and `checkout.h`. Junio’s self-review identified a shortcoming in the final patch: `post_checkout_hook()` still hardcodes `the_repository` and `the_hash_algo`, undermining its reusability. The proposal has not yet attracted objections but its scale suggests it will require careful review and coordination.

### Trace2 hardening effort faces deeper architectural questions
Derrick Stolee’s seven-patch series to eliminate `die()`-triggering helpers from the trace2 API faced expanded critique from Jeff King (Peff), who argued the current approach is a "tip of the iceberg" that may require a ground-up rewrite. The series replaces banned functions with defensive fallbacks, but Peff’s review highlights that indirect calls via helpers like `strbuf` or `json-writer.c` could still crash Git. Elijah Newren agreed to drop patch 3/4 entirely, replacing it with a targeted fix for `git mktree --batch` that removes the `OBJECT_INFO_QUICK` flag while retaining `SKIP_FETCH_OBJECT`.

The core fix (patch 4/4) remains ready for merging, having adopted Peff’s tri-state design for `fill_midx_entry()`. The series introduces `banned-die.h` as a new architectural enforcement mechanism, but the discussion now centers on whether the hardening effort should expand to address indirect dependencies or accept the current approach as a pragmatic first step.

### Use-after-free regression in `git stash show` reported
Nicolas Le Cam reported a use-after-free regression in `git stash show` when `--src-prefix` or `--dst-prefix` is used, introduced in Git 2.52.0 by commit 3ea35c64b ("stash: tell setup_revisions() to free our allocated strings"). The issue causes garbage in the diff header (e.g., `diff --git Uf.txt Uf.txt`) instead of the expected `diff --git a/f.txt b/f.txt`, breaking tools like `lint-staged` that rely on parseable diff output. The root cause is `OPT_STRING_F` storing pointers into `argv` elements that are later freed while `struct diff_options` still holds dangling pointers. The corruption is deterministic across versions but varies between runs in 2.53.0, strongly suggesting heap memory reuse after free.

The report includes exact reproduction steps and a plausible root cause analysis, making it actionable for a near-term fix. The regression affects scriptability and is likely to see rapid attention.

### Geometric repacking race condition fixes integrated
A four-patch bugfix series addressing a race condition in Git’s geometric repacking mechanism was integrated into Junio’s `next` branch. The series fixes crashes in `git replay` and missing-object errors in other operations (`git merge-tree`, `git diff`, `git rev-list`) caused by multi-pack-index (MIDX) references to removed packfiles. Jeff King (Peff) provided final endorsements for the core recovery logic in `odb/source-packed.c`, confirming the gating on `OBJECT_INFO_SECOND_READ` and iteration over `m->num_packs + m->num_packs_in_base` is correct and optimal.

The series is well-motivated by production evidence and includes thorough test coverage. The core fix implements a tri-state return from `midx_fill_entry()` (`MIDX_FILL_MISS`, `MIDX_FILL_HIT`, `MIDX_FILL_OWNER_UNAVAILABLE`) and gates recovery on `SECOND_READ` to optimize performance for `QUICK` callers. The function has been renamed to `midx_fill_entry()` to reflect its new semantics.

### ODB abstraction and fsck pluggability series progress
Patrick Steinhardt’s ODB abstraction effort continued with two significant series: an 8-patch series refactoring ODB alternates handling during repository creation, and a 10-patch series making ODB fsck checks pluggable. The alternates series removes the ability to write alternates after repository creation, simplifying the ODB interface, while the fsck series moves verification logic into backend-specific implementations, preparing for future pluggable ODB backends. Both series are technically complete and under review, with Karthik Nayak providing light feedback on the fsck series.

Junio raised a security concern about patch 6/8 in the alternates series, which opens the `info/alternates` file in append mode (`"a"`) rather than using Git’s standard lockfile API. This could lead to partial writes or TOCTOU races during repository creation. The discussion may delay the series until the locking strategy is resolved.

## In brief

**`git worktree add` basename handling** -- René Scharfe’s four-patch series fixing edge cases in `worktree_basename()` was marked ready for `next` after Junio C Hamano approved the code changes and applied a minor commit-message tweak. The series addresses an out-of-bounds read, rejects malformed paths, and trims trailing slashes from derived branch names.

**`git commit --amend` during conflict resolution** -- Elijah Newren’s single-patch bugfix extending protection against `git commit --amend` during conflict resolution to cover `git am`, `git revert`, and all forms of `git rebase` reached v3, addressing all substantive feedback. The patch is uncontroversial and likely to graduate soon.

**`git repo info` path keys** -- K Jayatheerth’s seven-patch series adding path-related keys to `git repo info` faces an architectural concern: duplication of logic between `git repo info` and `git rev-parse`. Junio proposed consolidating shared logic into a new helper library to avoid long-term maintainability issues.

**`git stash` branch-aware design** -- Vladimir Sitnikov’s RFC proposing a branch-aware or worktree-aware stash design advanced with a reflog-based solution that associates stashes with the specific commit they were created on. The proposal avoids new configuration or ref namespaces and enables per-worktree isolation for detached HEAD worktrees.

**`git branch -d` upstream protection** -- Harald Nordgren’s two-patch series proposing to protect local upstream branches from deletion via `git branch -d` faces a substantive backward-compatibility concern. Junio C Hamano objected to changing the long-established semantics of the `-d` flag, which could break existing workflows.

**`git format-rev` formatting options** -- Kristoffer Haugsbakk’s five-patch series adding `--abbrev`, `--color`, and `--date` options to `git format-rev` is uncontroversial but has received no review feedback. The series brings `git format-rev` closer to parity with `git log`.

**`git symbolic-ref` test coverage** -- Nikolaus Schuetz’s v3 test patch adding systematic test coverage for `git symbolic-ref`’s exit codes faces a documentation question: whether to document the current exit code behavior (128 vs. 1) in the man page. Junio C Hamano objected to "casting wrong behavior into stone."

**`git worktree add` `--guess-remote` ambiguity** -- Yoichi NAKAYAMA’s four-patch series improving error messages for ambiguous remote branch names in `git worktree add` was merged to `next`. The final patch ensures `--guess-remote` errors out when multiple matches exist, aligning it with `git checkout` and `git switch`.

**`http.sslVerifyStatus` for OCSP stapling** -- Grayson Gordon’s v6 patch adding a boolean `http.sslVerifyStatus` option to enable OCSP staple validation is technically complete and ready for `next`. The patch addresses a security gap in OpenSSL-linked Git binaries, which currently ignore OCSP staples.

**`gitk` AI contribution policy** -- Johannes Sixt’s documentation patch updating `gitk/README.md` to discourage AI-generated contributions is uncontroversial and aligned with upstream Git’s policy.

**`git whatchanged` deprecation feedback** -- User feedback confirmed the deprecation of `git whatchanged` (now requiring `--i-still-use-this`) does not disrupt workflows, as alternatives like `git log --oneline --name-status` suffice.

**Outreachy December 2026 cohort** -- Christian Couder invited volunteers, project ideas, and feedback on Git’s participation in Outreachy’s December 2026 cohort. Usman Akinyemi, a former Outreachy intern, volunteered to co-mentor again.

**Deprecation warning rewording** -- Junio C Hamano’s v2 documentation patch rewording the user-facing deprecation warning in `usage.c` is cooking in `next`. The new message is shorter, more direct, and guides users to the breaking changes documentation.

**`die_for_incompatible_opts()` helper** -- Junio C Hamano’s two-patch series introducing `die_for_incompatible_opts()` was withdrawn after the revised design was found unsatisfactory. René Scharfe proposed technical alternatives to improve compile-time safety.

**`gitremote-helpers` documentation fix** -- Lorenz Leutgeb documented the `if-asked` value for the `option pushcert` setting, aligning the documentation with the existing implementation.

**Negative pathspec handling** -- Diogo Castro’s bugfix patch addressing incorrect prefix-stripping for negative pathspecs in `git ls-files` and `git add` remains unresolved. The discussion centers on whether negative pathspecs should be treated as absolute or relative to the full tree.

**`post-worktree` hook proposed** -- Domen Kožar proposed a unified `post-worktree` hook to address Junio C Hamano’s design objection to the `post-worktree-*` series. The hook would use a subcommand-style interface (`add`, `move`, `remove`) and pass all relevant paths and the worktree ID as arguments.

**`dk/use-nsec-runtime` series** -- The series converting `USE_NSEC` to a runtime option (`core.useNanosec`) is ready for `master` but faces a long-term architectural concern: its new dependency on `repo_config_values()` could constrain future submodule support.

## Looking ahead

The coming week is likely to see progress on several fronts:

- The `git history squash` series is expected to graduate to `next` once the v15 reroll incorporating the `starts_with()` change is posted.
- The `receive-report` hook is poised to graduate from `seen` to `next` pending Junio’s final assessment.
- The architectural redesign of built-in command interfaces will attract broader review and may see follow-up patches addressing the `post_checkout_hook()` reusability shortcoming.
- The use-after-free regression in `git stash show` is likely to see rapid attention given its production impact and clear reproduction steps.
- The trace2 hardening series may see a v4 incorporating the decision to drop patch 3/4 and the targeted fix for `git mktree --batch`.
- The ODB alternates refactoring series may see a v2 addressing the lockfile API concern, while the fsck pluggability series is likely to graduate soon.