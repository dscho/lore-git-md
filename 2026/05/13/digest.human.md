# Git Mailing List Daily Digest — 2026/05/13 (Wednesday)

**The day in brief.** A moderately busy day with 65 emails across 22 threads, dominated by the arrival of v4 of Derrick Stolee's 13-patch `--path-walk`/`--filter` integration series — the day's most significant development, including a substantial rewrite of the `tree:0` handling to fix a correctness bug that had escaped detection. Harald Nordgren's `git branch --prune-merged` series also reached v9, resolving Junio's final objection and appearing ready for `next`. Several smaller series received their final reviews and are now merge-ready.

## Notable threads

**`--path-walk`/`--filter` integration reaches v4 with a `tree:0` rewrite.** Derrick Stolee posted the full 13-patch v4 of the series making `git pack-objects --path-walk` work with `--filter` options. The headline change is a substantial rewrite of the `tree:0` handling (patch 11/13), fixing a bug where tagged trees that were not also reachable from any commit could be silently dropped — a correctness issue that Taylor Blau had "correctly smelled" in v3 but that had escaped detection because the existing test data happened to use tagged trees that were also root trees from commits. A new preparatory test (patch 10/13) creates trees reachable only through tags to prevent future regressions. The series also adds `object:type=<type>` filter support (patch 12/13) with careful handling of the "trees off, blobs on" case, and a `combine` filter type (patch 13/13) that composes multiple filter specifications via recursive descent. Performance data continues to show dramatic speedups for `sparse:oid` (60% time reduction on git/git). The series has received no independent review across all four versions — only co-author Taylor Blau has engaged, and his comments have been surface-level. The `tree:0` rewrite and the recursive filter preparation in patch 13/13 are both subtle enough that a second pair of eyes would be valuable before this lands.

**`git branch --prune-merged` v9 resolves Junio's final objection.** Harald Nordgren sent v9 of the five-patch series adding safe, automated local branch cleanup. The sole change from v8 is the removal of the `--force` override that Junio had objected to: the reachability check is now always enforced, and `--force` has no special meaning with `--prune-merged`. The synopsis drops `[-f]`, and the error message now says "delete it with 'git branch -D' if you are sure" rather than suggesting `--force`. Patches 1 and 2 are unchanged; patch 4 (the `branch.<name>.pruneMerged` per-branch opt-out) and patch 5 (`--all-remotes` convenience flag) are updated to match. The series looks ready for `next`. Separately, Harald floated a `fetch.pruneLocalBranches` config concept in a follow-up email, but that is a preliminary idea with no code attached and no reviewer reaction yet.

**Maintenance subsystem lockfile and `gc.auto` fixes ready for merge.** Patrick Steinhardt's v3 two-patch series for the maintenance subsystem received Junio's concurrence. Patch 1 fixes a real bug where `git maintenance run --detach` released its lockfile immediately after forking, leaving the background work unprotected. The fix, adopting Taylor Blau's alternative design, automatically transfers ownership of all open tempfiles and lockfiles from parent to child inside `daemonize()` itself. Patch 2 restores the `gc.auto=0` behavior that was silently broken when the default maintenance strategy stopped shelling out to `git gc` — a user-reported regression. Both patches are now settled, with Junio signaling agreement and no outstanding change requests.

**`includeIf worktree:` series converges.** Chen Linxuan's v4 two-patch series adding `worktree:<pattern>` and `worktree/i:<pattern>` conditions to Git's `includeIf` config mechanism received a positive review from Phillip Wood ("this looks good to me"). The series addresses a genuine pain point in multi-worktree setups where `gitdir` patterns are useless because linked worktrees share the same `.git` directory. The implementation is minimal — about six lines of C in `config.c` — with thorough test coverage (113 new lines) covering bare repos, symlinked worktree paths, case-insensitive matching, early config reads, and non-repository scenarios. The series is ready for maintainer pickup.

**`strbuf_add_oid_hex()` patch reviewed and benchmarked.** René Scharfe's single-patch cleanup introducing `strbuf_add_oid_hex()` received a strong endorsement from Jeff King, who called it worth doing "solely for removing more instances of global buffers." René followed up with benchmark data showing a consistent ~7-8% speedup on `git cat-file --batch-all-objects --batch-check='%(objectname)'` on Apple M1 and M5 hardware. The patch includes a Coccinelle semantic patch that mechanically converts all 17 existing call sites across 14 files. No outstanding concerns remain.

**`git branch --forked` and `--prune-merged` v9 patches posted.** Harald Nordgren sent all five patches of the v9 series. Patch 1 (`--forked <remote>`) is unchanged from v8 and provides the listing infrastructure. Patch 2 is a pure refactoring of the deletion machinery. Patch 3 implements the core `--prune-merged` logic with the `--force` override removed. Patch 4 adds the `branch.<name>.pruneMerged` per-branch opt-out config. Patch 5 adds the `--all-remotes` convenience flag. The series is mechanically consistent with Junio's feedback and appears ready for `next`.

**`git status` push-branch advice fix at v2.** Harald Nordgren sent v2 of a single-patch fix for `git status` advice text when comparing against a push branch. The patch changes the hint from a bare `git pull` (which follows the upstream remote, potentially a different remote from the push target) to `git pull <remote> <branch>`. The v2 narrows scope from v1 by removing the divergence advice from the push-branch path, based on real usage experience. No reviews have been posted yet.

**`--word-diff` documentation clarification.** A single documentation patch from Michael Montalbo adds a paragraph to the `--word-diff` man-page entry explaining that word diff is a two-stage process (line-level diff first, then word-level within each hunk), which is why whitespace-only changes can produce word-level differences. The patch addresses a real confusion surfaced by Vincent Lefevre. No reviews have been posted yet.

**Config key error hint for `=` syntax.** A single patch adds a hint to `git_config_parse_key()` when the offending character is `=`, suggesting the correct `git config set key value` syntax. The heuristic is narrow by design to avoid false positives. No tests are included, which is a minor shortcoming for a user-facing error message change.

**Lowercase `head` resolves to wrong worktree HEAD on case-insensitive filesystems.** Alexander Sandström reported a bug where `git rev-parse head` in a linked worktree on macOS resolves to the main worktree's HEAD instead of the linked worktree's HEAD, because lowercase `head` falls through to filesystem-level ref lookup and accidentally matches `.git/HEAD`. D. Ben Knoble replied with links to prior 2024 discussions, noting that the filesystem-backed ref store is fundamentally incapable of fixing this and that the reftable transition should eliminate the problem. No patch has been submitted.

## In brief

**`strbuf_add_uint()` series ready for pickup** -- René Scharfe's four-patch series adding `strbuf_add_uint()` and converting four call sites has been independently benchmarked by both author and Jeff King, with all implementation variants showing equivalent performance. The thread is closed with no outstanding issues.

**`http-walker.c` alternate URL parsing fix accepted** -- Jeff King's one-character fix (`strchr` to `strchrnul`) for a NULL-pointer bug in dumb HTTP alternate URL parsing has been accepted by Junio. The bug has been present since 2005.

**`pretty.c` RFC 2047 encoder cleanup accepted** -- Jeff King's removal of a dead `strbuf_grow()` call from the RFC 2047 email header encoding path has been accepted by Junio, who suggested a broader cleanup of ineffective `strbuf_grow()` calls across the codebase as a `#leftoverbits` item.

**Sideband SGR parser accepts colon-separated parameters** -- Jeff King's one-character change to `sideband.c` adds `:` to the set of accepted separators in ANSI SGR escape sequences, fixing compatibility with 256-color and true-color escape codes that use the ECMA-48 colon-subparameter encoding.

**`git log --graph` parentless-commit rendering** -- Jeff King flagged a Coverity dead-code warning in Pablo Sabater's v3 patch for improved graph rendering of commits with excluded parents. The issue is a redundant guard inside an `else` branch where the guarded condition is already guaranteed false by the enclosing control flow.

**`git cherry`/`git grep` prefetch series** -- Elijah Newren replied to Derrick Stolee's architectural suggestion to centralize prefetch logic into the diff library, explaining why the cherry-specific filtering constraints (commit-level accumulation, binary-file skipping) do not generalize. The series remains on track for its re-roll with the pathspec test and redundant-prefetch bug fix.

**`git branch --prune-merged` v8 review** -- Junio C Hamano gave a split review of v8: strong approval of the upstream-reachability safety model, but objection to `--force` bypassing the mergedness check. That objection is resolved in v9.

**`includeIf worktree:` v3 review follow-up** -- Patrick Steinhardt clarified that even the known-broken early-config case must be captured as a test rather than left undocumented, using `test_expect_failure` if the limitation is intentional.

**`git maintenance --detach` lockfile fix** -- Junio endorsed the design direction of Patrick Steinhardt's v3 fix, agreeing that the daemonized child is logically a continuation of the original process and should inherit cleanup responsibility.

**`git diff --word-diff` documentation** -- Michael Montalbo sent a procedural follow-up confirming that a formal patch has been submitted addressing the documentation gap, with Vincent Lefevre's endorsement.

**Worktree-shared exclude file documentation fix** -- Phillip Wood gave a one-line "this looks good to me" approval for D. Ben Knoble's v3 patch correcting stale `$GIT_DIR/info/exclude` references to `$GIT_COMMON_DIR/info/exclude` across six files.

**`-U` and `--inter-hunk-context` negative number fix** -- Junio acknowledged receipt of Michael Montalbo's v2 series and confirmed it will be queued for `next` after a brief window for late comments.

**`git config set` syntax hint** -- A single patch adds a hint when `=` appears in a config key, suggesting the correct `git config set key value` syntax. No tests included.

## On the radar

**`includeIf worktree:` early-config asymmetry** -- The `worktree` condition does not work during early-config parsing, unlike the existing `gitdir` condition. Chen Linxuan is working on a fix but is unsure whether this is a must-fix. The asymmetry will need to be resolved before the feature can be considered complete.

**`fetch.pruneLocalBranches` concept floated** -- Harald Nordgren proposed a separate `fetch.pruneLocalBranches` config option that would automatically delete local branches during `git fetch` when their upstream counterpart is removed. This is a preliminary idea with no code or reviewer reaction yet, but it represents a significant expansion of scope from the `--prune-merged` series.