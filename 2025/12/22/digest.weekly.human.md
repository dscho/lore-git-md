# Git Mailing List Digest — 2025/12/22 -- 2025/12/28

**The week in brief.** A busy final week of 2025 with 196 emails across 66 threads saw significant progress on several major fronts. The standout developments were the completion of Adrian Ratiu's multi-year hook subsystem refactoring and Harald Nordgren's configurable branch comparison for `git status`, though the latter remains in design debate. Other key topics included batched reference update error reporting regressions, `pack-refs` performance investigations, and ongoing work to remove `the_repository` global state. The week balanced technical refinements with foundational architectural questions, setting up several important discussions to continue into the new year.

## Key developments

### Hook subsystem modernization completes

Adrian Ratiu's 11-part series refactoring Git's hook infrastructure reached completion this week after six iterations and years of groundwork. The v6 patches standardize hook execution on a new `hook.h` API, introducing structured handling for stdin, output capture, and parallel execution while maintaining backward compatibility. Key improvements include batched stdin processing for receive hooks (500 lines per callback) and removal of direct `find_hook()` calls across the codebase. With sign-offs from Ævar Arnfjörð Bjarmason and Emily Shaffer, plus Junio Hamano's final ack, this foundational work is now queued for merging in an upcoming release cycle.

### `status.goalBranch` design debate persists

Harald Nordgren's effort to add configurable branch comparison to `git status` saw nine iterations this week (v1-v9) but remains in limbo due to fundamental design questions. The feature allows tracking divergence from a secondary "goal" branch (like upstream/main) via `status.goalBranch` configuration. While technically mature with comprehensive test coverage, Junio Hamano questioned whether it justifies a new config variable rather than leveraging existing tracking infrastructure. Nordgren and D. Ben Knoble defended the explicit configuration approach for triangular workflows, keeping the discussion alive as the year ends.

### Batched reference update error reporting regression

A regression in detailed error messages for batched reference updates (introduced in Git 2.51.0) saw thorough analysis this week. Jeff King and Karthik Nayak traced the issue to how error details are lost during transaction processing, affecting push, fetch, and update-ref operations across all ref backends. The discussion converged on attaching error details to individual failed ref updates rather than whole transactions, with Nayak planning implementation work in January. This addresses a significant usability regression where specific "non-commit object" messages were replaced with generic errors.

### `pack-refs` performance investigation

Performance analysis of `git pack-refs` on repositories with millions of refs revealed surprising overhead from unnecessary tag peeling. Jeff King and Brian m. carlson identified that Git verifies object existence for all refs during packing, even those under `refs/heads/` that shouldn't need peeling. The discussion highlighted potential long-term advantages of the reftable backend, which can maintain pre-peeled references to avoid this overhead. The findings may influence future optimization work for large-scale deployments.

### `the_repository` removal continues

René Scharfe advanced the ongoing effort to eliminate Git's `the_repository` global variable with a 4-part series converting tag handling code to explicit repository parameters. The changes make core tag operations properly repository-aware, pushing context through call chains in describe, pack-objects, and verify-tag commands. This continues Git's architectural shift away from global state, particularly in security-sensitive paths like tag verification where repository isolation matters.

## In brief

**SGID bit preservation** -- Hadmut Danisch reported Git clears SGID bits (mode 2770) during operations, breaking container workflows relying on automatic group inheritance. Discussion explored alternatives like ACLs but no consensus emerged on changing Git's behavior.

**PID-based lockfile debugging** -- A feature to track processes holding locks via `~pid.lock` files reached v3, with Junio Hamano reviewing documentation and error handling details before final approval.

**`git branch --show-current` behavior** -- The command will explicitly error when combined with `-v` rather than silently ignoring the flag, addressing what Junio now considers buggy behavior.

**Windows MSI packaging inquiry** -- A user inquiry highlighted the lack of MSI packages for Git for Windows enterprise deployment via Active Directory Group Policy.

**Promisor remote enhancements** -- A 9-part series introduced `promisor.storeFields` configuration and `--filter=auto` mode for dynamic filter combination from promisor remotes, improving partial clone workflows.

**New contributor onboarding** -- Three potential GSoC 2026 contributors (Andrew Chitester, Deveshi Dwivedi, Ayush Jain) received standard guidance from Christian Couder about the contribution process.

## Looking ahead

Several major topics will carry into January: the `status.goalBranch` design discussion may see alternative proposals from D. Ben Knoble, while Karthik Nayak's work on batched reference error reporting and Patrick Steinhardt's ODB abstraction efforts are poised for new iterations. The SGID bit preservation debate may also resurface as container workflows become more prevalent. With the hook subsystem refactoring now complete, attention may shift to its follow-on benefits like config-based hooks and parallel execution support.