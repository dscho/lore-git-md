Here's the daily digest for September 12, 2025:

## The day in brief

A moderately active day with 72 emails across 20 threads, featuring significant progress on several fronts: the `sparse-checkout clean` series was approved for merging, a heated debate continued about removing `git-new-workdir`, and the SHA-256 transition team outlined remaining work for Git 3.0. Documentation improvements and signature handling for `fast-import` also saw substantial discussion.

## Notable threads

### `git-new-workdir` removal debate intensifies

The ongoing discussion about removing the `contrib/git-new-workdir` script reached a critical point as Gabriel Scherer pushed back against the removal, citing workflow impacts from losing the script's permissive multiple-checkout behavior. Phillip Wood and Junio Hamano clarified technical alternatives using `git worktree --ignore-other-worktrees`, but Scherer noted this doesn't address the loss of remote configuration sharing across workdirs. Junio firmly defended the removal as necessary maintenance reduction, stating `git worktree` was explicitly designed as the replacement despite some workflow differences. The thread reveals tensions between backward compatibility and long-term code health.

### `sparse-checkout clean` approved

Derrick Stolee and Victoria Dye's collaborative series adding a `git sparse-checkout clean` command was approved by Junio after addressing all review concerns. The feature removes tracked-but-sparse directories outside the sparse-checkout scope while preserving untracked files, with safety mechanisms matching `git clean` (7 patches)

### Signature handling for `fast-import`

Christian Couder's series adding `--signed-commits` to `git fast-import` progressed through detailed review, with v2 addressing feedback about error handling, warning messages, and test coverage. The implementation shares parsing logic with `fast-export` via refactored gpg-interface code. Junio questioned starting with commits rather than more commonly signed tags but accepted the phased approach. (2 patches)

### SHA-256 transition status update

Brian Carlson provided a comprehensive status report on SHA-256 interoperability work, identifying key areas needing help for the Git 3.0 target: test coverage, `git gc` integration, submodule handling, and repository conversion tools. With 93 patches already in their `sha256-interop` branch and 200-300 more expected, Carlson welcomed assistance particularly from organizations able to perform large-scale testing.

### Documentation improvements

Julia Evans' documentation series for `git-push` was merged after refining explanations of upstream branches and push behavior. The changes make the man page more accessible while maintaining technical precision. Follow-up discussion explored better ways to document pathspec usage across commands, with Kristoffer Haugsbakk suggesting a dedicated `gitpathspecs(7)` man page.

## In brief

**Ref transaction edge cases** -- Karthik Nayak refined error handling for batched reference updates on case-insensitive filesystems, addressing feedback about lock file conflicts. (1 patch)

**Bundle-URI resilience** -- Two independent series (from Patrick Steinhardt and Toon Claes) made `git-clone` more resilient to malformed bundle-URI responses, though debate continues about appropriate error handling strictness. (3 patches)

**Rust build system polish** -- SZEDER Gábor noted a minor oversight in Rust build artifact cleanup that will be addressed in the ongoing infrastructure series. (1 note)

**`user.email` validation debate** -- A lengthy discussion concluded that Git's permissive approach to `user.email` values is intentional, supporting use cases like GitHub's SSO token storage despite lacking email format validation.

**Merge behavior reports** -- Guo Tingsheng filed several reports about merge edge cases (indentation loss, false conflicts in Java method additions), with Elijah Newren requesting reproducible test cases to investigate further. (3 reports)

## On the radar

**SHA-256 transition** -- With Git 3.0 planning to make SHA-256 the default, Brian Carlson's status update highlights the scale of remaining work and opportunities for contributor involvement.

**Rust integration** -- The build system patches are in final polishing stages, with only minor cleanup items like proper `Cargo.lock` handling remaining before potential merging.