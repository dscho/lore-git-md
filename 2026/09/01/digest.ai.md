# Git mailing list daily digest for 2026/09/01

## The day in brief
The Git mailing list saw significant activity around several key topics: a new `--missing-only` option for `git rev-list` was clarified and approved; a major refactoring of `git checkout` internals progressed with design discussions; a critical bugfix for geometric repacking race conditions received final approvals; and a discussion about binary file corruption in checkouts revealed platform-specific issues. Several performance optimizations and architectural improvements also moved forward.

## Notable threads

### Git rev-list gains --missing-only option for GitLab's Gitaly workflow
**What changed**: Siddharth Asthana's patch series implementing `--missing-only` for `git rev-list` was approved after clarifying GitLab's use case.

**Problem/goal**: GitLab's Gitaly needed an efficient way to identify missing objects in partial clones during transaction packing without post-processing.

**Technical details**:
- New flag: `--missing-only` filters output to show only missing objects
- Output format: one OID per line (no `?` prefix) or `path=`/`type=` fields with `--missing=print-info`
- Implementation touches `builtin/rev-list.c`, documentation, and tests
- Rejects incompatible options like `--count` and `--disk-usage`

**Impact**: Enables GitLab's single-pass transaction packing workflow, eliminating the need to strip `?` prefixes from output. The feature is now ready for integration after resolving workflow questions about Gitaly's consumption pattern.

**Today's development**: [2026/09/01/20-27-51] Siddharth clarified that Gitaly uses the feature in a single-pass transaction packing workflow, not iterative onion-peeling, resolving Junio's workflow question.

---

### git replay --linearize feature series receives final approval
**What changed**: Elijah Newren's patch series adding `--linearize` to `git replay` received final Reviewed-by.

**Problem/goal**: Provide a simpler alternative to Johannes Schindelin's merge-replay implementation that flattens merge commits into linear history.

**Technical details**:
- New CLI option: `git replay --linearize`
- Restrictions: cannot be used with multiple revision ranges or `--contained`
- Implementation in `replay.c`, `revision.h`, and documentation
- Behavioral difference from `git rebase --no-rebase-merges`: drops merges and keeps only one branch

**Impact**: Offers predictable, all-or-nothing flattening behavior that avoids dangling commits. The series is technically complete and ready for integration after addressing all prior feedback.

**Today's development**: [2026/09/01/22-15-20] Elijah Newren provided Reviewed-by for the entire series, signaling technical completion and readiness for integration.

---

### HTTP daemon test helpers race condition fix completes review
**What changed**: Michael Montalbo's bugfix series for race conditions in HTTP daemon test helpers completed review.

**Problem/goal**: Fix non-atomic state management in CGI scripts (`apply-one-time-script.sh` and `http-429.sh`) that could cause HTTP 500 errors under concurrent requests.

**Technical details**:
- Replaces racy shell operations with atomic alternatives: `rm` without `-f` and `mkdir`
- Adds new test `t5567-one-time-script.sh` for deterministic verification
- Documentation updates in `t/lib-httpd.sh`
- Fixes observed flaky test failures in macOS CI runners

**Impact**: Makes test infrastructure more robust against concurrent execution, addressing real-world flakiness in CI environments.

**Today's development**: [2026/09/01/11-17-11] Patrick Steinhardt suggested minor wording improvement for documentation ("guarded by the marker" instead of "claims the marker"), which was accepted by the author.

---

### Trace2 hardening series explores architectural alternatives
**What changed**: Derrick Stolee's trace2 hardening series received architectural feedback about long-term solutions.

**Problem/goal**: Eliminate all `die()`-triggering helpers from trace2 to prevent Git crashes during telemetry operations.

**Technical details**:
- Introduces `banned-die.h` to enforce compile-time bans on `die()`-prone functions
- Replaces banned functions with defensive fallbacks
- Current approach blocks direct calls but misses indirect calls via helpers like `strbuf`

**Impact**: Makes trace2 more resilient to memory pressure and system call failures, though current implementation doesn't address all transitive dependencies.

**Today's development**:
- [2026/09/01/05-01-29] Jeff King proposed a "ground-up no-dependency rewrite" of trace2 as the only way to fully eliminate unsafe dependencies
- [2026/09/01/13-42-46] Derrick Stolee floated removing `banned-die.h` entirely in a hypothetical v4, focusing on incremental reductions

---

### git last-modified Bloom filter optimization completes review
**What changed**: Toon Claes's performance optimization series for `git last-modified` completed review.

**Problem/goal**: Speed up `git last-modified` by reusing Git's Bloom filter machinery for wildcard pathspecs.

**Technical details**:
- Reuses revision walk Bloom-filter logic
- Maintains per-path Bloom filters for wildcard pathspecs
- Restores `bloom_filter_settings` after `prepare_revision_walk()` clears it
- Adds trace2 instrumentation to verify Bloom filter fast-path

**Impact**: Achieves 3.7× speedup (60.7 ms → 16.2 ms) without `--show-trees` and 1.95× speedup (61.9 ms → 31.8 ms) with `--show-trees`.

**Today's development**: [2026/09/01/09-14-13] Toon confirmed Junio's test-environment fix for `GIT_TEST_COMMIT_GRAPH` is already incorporated in v4.

---

### Binary file corruption during checkout reveals platform-specific issues
**What changed**: A new discussion revealed silent corruption of binary files during checkout due to overly broad `.gitattributes` rules.

**Problem/goal**: Prevent binary file corruption (PNGs, JPEGs) during checkout caused by global `.gitattributes` rules like `* text diff=lfclean`.

**Technical details**:
- Issue affects binary files when global `.gitattributes` rules are applied
- Platform-specific advice includes `core.autocrlf=input` and explicit binary declarations
- Proposed safeguards include warnings and automatic binary detection

**Impact**: Highlights fragility of binary data handling in Git and the need for better safeguards against silent corruption.

**Today's development**:
- [2026/09/01/20-14-05] Skybuck Flying reported silent corruption of binary files
- [2026/09/01/21-37-42] Provided detailed reproduction steps and proposed safeguards
- [2026/09/01/21-45-38] Randall S. Becker shared platform-specific advice and warned about filter fragility

---

### git checkout refactoring series progresses with design discussions
**What changed**: Junio C Hamano's refactoring of `git checkout` internals received design feedback.

**Problem/goal**: Eliminate the monolithic `checkout_main()` choke point and extract reusable logic into standalone helpers.

**Technical details**:
- 8-patch series restructuring `git checkout`, `git switch`, and `git restore`
- Extracts pathspec and branch setup helpers
- Moves `post_checkout_hook` to `checkout.c`
- Passes `cb_option` explicitly

**Impact**: Places all callers on equal footing and prepares for future architectural improvements.

**Today's development**:
- [2026/09/01/11-31-21] Karthik Nayak proposed eliminating the `cb_option` parameter in favor of deriving information from `enum checkout_command`
- [2026/09/01/17-47-29] Junio asked for clarification on a correctness issue in the third patch regarding option-compatibility check placement

---

### Conflict-resolution safeguards extended to more commands
**What changed**: Elijah Newren's bugfix series extending conflict-resolution safeguards posted v4.

**Problem/goal**: Prevent unsafe commit operations (`git commit --amend` and `git commit <paths>`) during conflict resolution for `git am`, `git revert`, and all forms of `git rebase`.

**Technical details**:
- Blocks `--amend` and partial commits during conflict resolution
- Uses `sequencer_ongoing_operation()` helper to detect active operations
- Clear error messages for different scenarios
- Comprehensive test coverage

**Impact**: Improves user safety during conflict resolution by preventing operations that could corrupt repository state.

**Today's development**: [2026/09/01/22-24-36] Elijah posted v4 incorporating mechanical renames (`FROM_REBASE_NOW_EMPTY`, `is_from_rebase_now_empty()`).

---

### ODB alternates refactoring series receives surface-level approval
**What changed**: Patrick Steinhardt's ODB alternates refactoring series received surface-level approval.

**Problem/goal**: Remove the ability to write ODB alternates after repository creation, simplifying the ODB interface.

**Technical details**:
- 8-patch series deferring alternates setup to `odb_create_on_disk()`
- Removes ad-hoc alternates writing API
- Uses Git's standard lockfile API for atomic writes

**Impact**: Prepares for alternates to become an implementation detail of the ODB backend.

**Today's development**: [2026/09/01/06-09-39] Toon Claes provided surface-level approval: "This version looks good to me!"

---

### git ident feature discussion explores git var extension
**What changed**: Jeff King proposed extending `git var` instead of introducing a new `git ident` command.

**Problem/goal**: Provide a unified, user-friendly command to consolidate Git's identity and signing configuration.

**Technical details**:
- Current proposal: standalone `git ident` command
- Alternative: extend `git var` with new keys like `GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL`, `GIT_SIGNING_KEY`
- Both approaches support machine-readable output and scripting

**Impact**: Could avoid command proliferation while making `git var` more versatile.

**Today's development**: [2026/09/01/04-39-44] Jeff King proposed extending `git var` instead of introducing `git ident`.

---

### Use-after-free regression in git stash show receives final approvals
**What changed**: Jeff King's bugfix for a use-after-free regression in `git stash show` received final approvals.

**Problem/goal**: Fix use-after-free bug causing garbage diff headers when `--src-prefix` or `--dst-prefix` is used.

**Technical details**:
- Introduces `strvec argv_to_free` field in `struct rev_info`
- Delays freeing "freed" `argv` elements until `release_revisions()`
- Refactors `mark_argv_for_free()` to consolidate flag logic

**Impact**: Restores correct diff header output and prevents crashes in scripted workflows.

**Today's development**: [2026/09/01/11-08-18] Patrick Steinhardt provided final approval, accepting Jeff's rationale for the `const char *` parameter design.

---

### ODB ad-hoc source linking removal series receives review feedback
**What changed**: Patrick Steinhardt's ODB refactoring series received review feedback from Junio.

**Problem/goal**: Remove the mechanism allowing registration of in-memory sources, simplifying the ODB subsystem.

**Technical details**:
- 12-patch series removing ad-hoc source linking API
- Eliminates dual-use of ODB sources list for alternates and temporary sources
- Prepares for future migration of alternate handling into "files" backend

**Impact**: Simplifies ODB codebase and continues `the_repository` removal effort.

**Today's development**:
- [2026/09/01/22-03-44] Junio suggested making `cache_tree_fully_valid()` file-scoped static with a thin wrapper
- [2026/09/01/22-47-04] Junio identified two potential correctness issues in repository context handling

## In brief

- **[2026/09/01/04-19-21]** Junio pointed out test environment issue for `git last-modified` Bloom filter optimization
- **[2026/09/01/04-35-07]** Junio flagged typo "comparisions" in `core.useNanosec` documentation
- **[2026/09/01/05-00-56]** Junio endorsed Jeff King's `git var` extension proposal for identity configuration
- **[2026/09/01/06-28-15]** Jeff King introduced fix for use-after-free regression in `git stash show`
- **[2026/09/01/08-16-11]** Patrick Steinhardt fixed GitLab CI documentation job failure
- **[2026/09/01/11-26-20]** Patrick Steinhardt requested eviction of parallel packfile URI fetching topic
- **[2026/09/01/15-19-22]** Karthik Nayak posted v5 of `receive-report` hook series, ready for integration
- **[2026/09/01/15-26-25]** Derrick Stolee provided line-by-line review of MIDX missing pack fallback fix
- **[2026/09/01/17-32-44]** Junio confirmed typo fix for `core.useNanosec` documentation
- **[2026/09/01/17-48-21]** Junio acknowledged eviction request for parallel packfile URI fetching topic
- **[2026/09/01/17-58-32]** Junio endorsed Jeff King's observation about outdated Asciidoctor version pin
- **[2026/09/01/18-02-27]** Junio approved refactoring of `mark_argv_for_free()` helper
- **[2026/09/01/22-26-46]** Junio approved removal of transitional submodule ODB registration mechanism