# Git Mailing List Digest - 2026/02/16 -- 2026/02/22

## The week in brief

A busy week with 598 emails across 189 threads saw significant progress on multiple fronts. Key developments include the completion of several long-running efforts: ref backend selection (enabling zero-downtime migrations between files and reftable backends), UTF-8 alias support in config subsections, and Linux fsmonitor implementation. The week also featured important security hardening for repository discovery, parallel hook execution capabilities reaching maturity, and ongoing architectural work to remove `the_repository` global variable. Junio Hamano's "What's cooking" report on February 21 provided a comprehensive snapshot of the project's current state.

## Key developments

### Ref backend selection finalized

Karthik Nayak's series enabling zero-downtime migrations between ref backends (files<->reftable) reached completion after seven iterations. The implementation provides three control mechanisms: `extensions.refStorage` config, `GIT_REFERENCE_BACKEND` environment variable, and URI-based specification. This primarily serves GitLab's migration needs, with all technical concerns resolved and comprehensive test coverage in place. The final patches focused on stub management centralization and environment variable behavior, receiving positive reviews from Patrick Steinhardt and others before Junio Hamano's approval.

### Config-based hooks with parallel execution

Adrian Ratiu's config-based hooks series expanded to include parallel execution capabilities, reaching v2 this week. The redesigned implementation makes parallel execution opt-in (`hook.<name>.parallel`) with job count configuration (`hook.jobs`), addressing safety concerns from earlier reviews. Building on prior configurable hooks work (using `hook.<name>.command`, `hook.<name>.event`, and `hook.<name>.enabled`), the series includes extensive test coverage and careful output stream handling to maintain backward compatibility. Review feedback has been consistently positive, with the v2 changes reflecting design refinements from Patrick Steinhardt and Phillip Wood.

### Linux fsmonitor implementation ready

Paul Tarjan's inotify-based Linux fsmonitor backend (now at v4) appears ready after months of development. The patch has been stable in production for two months across a large deployment, addressing earlier concerns about memory leaks and GPL compliance. The only remaining issues are two small memory leaks (512-byte during startup and 40-byte in IPC handling) that Junio suggests could be fixed post-merge. This brings Linux to parity with existing Windows/macOS fsmonitor backends, enabling faster status operations through event watching rather than directory scanning.

### Security hardening for repository discovery

Tian Yuchen's series hardening Git's repository discovery by validating `.git` file types reached v6 this week. The patches clearly separate error handling refinements from the security checks that reject dangerous entries (FIFOs/sockets) while allowing valid symlinks. A new test script verifies behavior with comprehensive cases. The security model itself is uncontested, with the implementation now properly distinguishing fatal errors from benign conditions. The series addresses all feedback but surfaces deeper questions about worktree/repository relationships that may need follow-up work.

### `the_repository` removal advances

Significant progress was made in removing the global `the_repository` variable, particularly in Elijah Newren's merge-ort series which reached v3. The changes propagate repository context through the merge machinery, replacing global variable usage with `opt->repo` parameters. The final patch enforces compile-time prevention of future `the_repository` usage via a `DO_NOT_USE_THE_REPOSITORY` macro. Separately, Burak Kaan Karaçay's GSoC contribution removing `the_repository` from mailmap handling was approved after addressing header inclusion issues.

### Configurable branch comparisons in `git status`

After 28 iterations spanning months, Harald Nordgren's effort to add configurable branch comparisons to `git status` is ready for merging. The feature implements `status.compareBranches` configuration to show comparisons against multiple branches, initially supporting only `@{upstream}` and `@{push}` syntax per Junio's phased approach. Jeff King contributed foundational changes to `repo_dwim_ref()` to handle edge cases, and extensive test coverage (337 lines) verifies the behavior. The thread's conclusion was nearly derailed when Junio noted the topic had gone stale in his branches, prompting a final check for real-world interest before merging.

## In brief

**UTF-8 alias support** -- Jonatan Holmgren's series enabling UTF-8 in config subsections (e.g., `[alias "förgrena"]`) is now merge-ready after seven iterations, with comprehensive test coverage including Swedish, CJK, and space-containing aliases.

**Submodule ignore behavior** -- The series addressing `git add` behavior with `ignore=all` configurations received maintainer approval, making `git add` respect `submodule.<name>.ignore=all` by default while requiring explicit `--force` to override.

**HTTP 429 retry support** -- Vaidas Pilkauskas's series implementing rate-limited HTTP response handling is complete with strbuf fixes and comprehensive test coverage for both delay-seconds and HTTP-date formats.

**Shallow push performance** -- Patrick Steinhardt's optimization series targeting shallow push handling in `git receive-pack` shows significant improvements - up to 3.93x speedup and 60x reduction in memory allocations in test cases.

**Gitweb mobile improvements** -- Rito Rhymes' mobile responsiveness series (74 lines of CSS additions) is merge-ready after reorganizing patches per Eric Sunshine's feedback.

**Promisor remote enhancements** -- Christian Couder's 9-patch series implementing secure field storage and dynamic filter combination for promisor remotes has all review feedback addressed and is merge-ready.

**Worktree API cleanup** -- Phillip Wood's changes introduce `get_worktree_from_repository()` to construct worktree structs from repository data while removing redundant `repository` parameters from `worktree_git_path()`.

**Format-patch cover letter fix** -- A bugfix ensuring `--from` works consistently for cover letters now includes improved commit message and test coverage per Junio's review.

**String list sorting standardization** -- Elijah Newren and Amisha Chhajed finalize patches ensuring consistent sorted+deduplicated output across commands using `string_list_sort_u()`.

**MIDX compaction** -- Taylor Blau's 18-patch MIDX compaction series appears complete after addressing review feedback but has been quiet since January.

## Looking ahead

**Namespace security debate** -- The discussion about receive-pack's handling of symbolic refs crossing namespace boundaries continues, with Junio Hamano maintaining security boundaries should be strict by default while others argue for flexibility.

**GSoC 2026 planning** -- Git's participation is confirmed with three core technical tracks: continuing `the_repository` removal, improving the `git repo` subcommand, and extending `git cat-file` with remote-object-info functionality.

**Send-email client certs** -- Initial patches adding SMTP client certificate support entered review this week, with Junio providing detailed documentation feedback on the new authentication options.

**Maintenance strategy change** -- Patrick Steinhardt's series switching the default maintenance strategy to geometric repacking completed test preparation and is considering whether to gate the change behind `feature.experimental`.

**Replay revert interface** -- Junio expressed preference for using subcommands (`revert`, `pick`) rather than flags in `git replay`, potentially requiring interface restructuring of the already-implemented `--revert` feature.