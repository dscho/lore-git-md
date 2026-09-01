# Git mailing list daily digest for 2026/08/31

## The day in brief
The Git project saw significant progress on several fronts today. The `--linearize` option for `git replay` reached a mature state, ready for integration. A critical race condition in geometric repacking was resolved, preventing crashes in `git replay` and other operations. The ODB abstraction effort advanced with multiple topics graduating to `master`, while new refactoring work on alternates handling and fsck pluggability entered the pipeline. Documentation improvements and minor bugfixes also made progress.

## Notable threads

### `git replay --linearize` reaches maturity
**What changed:** The v9 iteration of Toon Claes's series introducing `--linearize` to `git replay` addresses all prior feedback, including UX terminology refinements and the multi-branch ambiguity resolution.

**Problem/goal:** Add a `--linearize` option to `git replay` that flattens merge commits, producing a linear history as an alternative to Johannes Schindelin's earlier merge-replay implementation.

**Technical details:**
- New CLI option: `git replay --linearize`
- Restrictions: Cannot be used with multiple branches or `--contained`
- Implementation: Merge commits are dropped, subsequent commits reparented onto the last non-merge commit
- Files touched: `replay.c`, `replay.h`, `builtin/replay.c`, `Documentation/git-replay.adoc`, `t3650-replay-basics.sh`

**Impact:** This feature provides a predictable, all-or-nothing flattening behavior that avoids dangling commits. The series is technically complete and ready for integration, with Junio having removed the previous version from `next` and queued v9 for review.

**Today's developments:** Toon accepted all UX terminology suggestions from Elijah Newren and Justin Tobler, simplifying documentation and error messages. The single-branch restriction now applies consistently to `--linearize`, `--revert`, and `--advance` via the shared `set_up_branch_mode()` helper.

### Geometric repacking race condition fixed
**What changed:** Elijah Newren's four-patch series fixing a race condition in geometric repacking graduated to `master`, resolving crashes in `git replay` and other operations.

**Problem/goal:** Fix a race condition where MIDX references a packfile removed by geometric repacking, causing processes using the stale MIDX to fail to locate objects.

**Technical details:**
- Root cause: When an object exists in multiple packs and the preferred pack is removed, the code fails to find the object in remaining packs
- Solution: Introduce a recovery mechanism in `odb/source-packed.c` that transparently recovers objects when MIDX references a removed packfile
- Implementation: Tri-state return values from `midx_fill_entry()` and `SECOND_READ`-gated fallback
- Files touched: `replay.c`, `odb/source-packed.c`, `midx.c`, `midx.h`, `builtin/mktree.c`

**Impact:** This fix prevents SIGSEGV in `git replay` and missing-object errors in other operations. The series also fixes a leak in `git mktree --batch` and demonstrates problems with `OBJECT_INFO_QUICK` readers.

**Today's developments:** Jeff King provided final confirmation that the series should proceed to `master`, resolving the last procedural step before graduation.

### ODB abstraction effort advances
**What changed:** Multiple ODB-related topics graduated to `master`, while new refactoring work entered the pipeline.

**Graduated topics:**
- `ps/odb-eagerly-load-alternates`: Alternates loaded at ODB creation time
- `ps/odb-generic-corrupt-objects`: ODB layer distinguishes "missing" from "corrupt"
- `ps/odb-pluggable-pack-generation`: Packfile generation moved behind ODB interface
- `ps/odb-geometric-repack-loose-threshold`: Loose-object count threshold for geometric repacking

**New topics in `next`:**
- `ps/odb-alternates-at-creation`: Defer alternates setup to `odb_create_on_disk()`
- `ps/odb-pluggable-fsck`: Move fsck checks into ODB backends

**Problem/goal:** Restructure Git's object database to support pluggable backends and improve maintainability.

**Technical details:**
- Alternates handling: Moved to ODB creation time, removing lazy-loading calls
- Error reporting: Unified mechanism for distinguishing missing vs corrupt objects
- Packfile generation: Abstracted behind ODB interface used by `upload-pack`, `send-pack`, and `bundle`
- Fsck checks: Moved into backend-specific implementations

**Impact:** These changes prepare the codebase for future pluggable ODB backends while improving error handling and maintainability. The alternates refactoring resolves TOCTOU race conditions by adopting the lockfile API.

**Today's developments:** Patrick Steinhardt addressed review feedback on the alternates refactoring, agreeing to adopt the lockfile API for writing the `info/alternates` file. The fsck pluggability series received final sign-off from Karthik Nayak.

### `git checkout -m` autostash conflict handling
**What changed:** Harald Nordgren posted v2 of the series refining `git checkout -m` behavior when autostash is involved.

**Problem/goal:** Visually separate autostash conflict advice from the branch-switch confirmation message to eliminate visual clutter.

**Technical details:**
- Files touched: `builtin/checkout.c`, `sequencer.c`, `t/t7201-co.sh`
- Change: Repurpose return value to encode three states (success=0, error=-1, conflict=1)
- Behavior: Print blank line to stderr when conflicts occur

**Impact:** Pure UX improvement that makes conflict messages clearer without altering underlying logic.

**Today's developments:** Junio identified a correctness issue - the patch treats any non-zero return from `run_command()` as a conflict, but `git stash apply` can fail for other reasons. Harald will need to address this in v3.

### CI workflow cancellation optimization
**What changed:** Harald Nordgren's patch to cancel stale GitHub Actions workflow runs for pull requests graduated to `next`.

**Problem/goal:** Reduce wasted CI runner capacity by ensuring only the latest workflow run for a pull request remains active.

**Technical details:**
- File touched: `.github/workflows/main.yml`
- Change: Make `concurrency.group` expression pull-request-aware and workflow-specific
- Behavior: Stale workflow runs for the same PR are canceled on new pushes

**Impact:** This optimization reduces CI resource consumption without affecting functionality.

**Today's developments:** Junio accepted the trade-off regarding potential duplicate workflow runs for the same commit and applied the series to `next`.

## In brief

- **`git last-modified` Bloom filter optimization:** Toon Claes posted v3 of the series reusing revision-walk Bloom-filter logic, addressing the last test gap with trace2 instrumentation.
- **`git whoami` → `git ident`:** Andrew Pleeter posted v2 of the identity command with improved flag design, machine-readable output, and X.509 signing key support.
- **HTTP OCSP staple validation:** The `http.sslVerifyStatus` feature graduated to `next`, enabling OCSP staple validation for government and FIPS-compliant users.
- **Worktree basename fixes:** René Scharfe's series fixing out-of-bounds reads and trailing-slash handling in `worktree_basename()` graduated to `next`.
- **Branch name validation inconsistency:** Bence Csókás reported that `git checkout -b @` allows creating a branch named `@` despite `git check-ref-format` rejecting it.
- **Typo fixes:** Multiple typo-fix patches made progress, with some requiring procedural splits between Git and Git GUI changes.
- **"What's cooking" report:** Junio's integration status report showed 12 topics graduated to `master`, 11 new topics in `next`, and 8 stalled topics awaiting author response.