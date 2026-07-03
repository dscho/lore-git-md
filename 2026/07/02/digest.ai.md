# The Git Mailing List Daily Digest for 2026/07/02

**The day in brief.**
A busy day on the list with 68 emails across 21 threads. The standout developments: **`git replay --linearize` v6** fixes critical post-merge regressions and is now ready for re-merge; **`git history squash` v6** is complete and awaiting Junio’s final review; and **Jeff King’s 9-patch series** plugs memory leaks in non-default hash implementations. A lighter note: Git v2.55.0’s contributor list humorously included "Claude Sonnet 4.6," sparking lighthearted commentary.

---

## Notable threads

### `git replay --linearize` v6 fixes critical regressions
**Toon Claes** posted v6 of the `--linearize` option for `git replay`, addressing all three critical issues identified after v5 was merged:
1. **Silent commit dropping**: Restored the `replayed_base` logic (patch 2/3) to ensure all commits in the input range are replayed.
2. **Merge commit divergence**: Documented the behavior of preserving both sides of divergent merges (patch 3/3).
3. **CLI design**: Justified the `--linearize` naming in commit messages and documentation.

The series is now **ready for re-merge** to replace the flawed v5. **Johannes Schindelin** and **Patrick Steinhardt** provided key feedback, with Schindelin advocating for UX clarity over consistency with `git rebase`. The fixes are well-tested, including new edge cases for divergent merges and root commits.

---

### `git history squash` v6 complete, awaiting final review
**Harald Nordgren**’s 6-patch series introducing `git history squash` is now **code-complete** after addressing all prior feedback:
- **Template formatting**: Adopted a minimalist design (numbered subject list + editable body) to reduce visual clutter.
- **Input validation**: Added shape-based validation and sanity-checks for `rev-list` options.
- **Recoverability**: Documented limitations of `git reset --hard` for multi-ref operations, with **Patrick Steinhardt** proposing a long-term "oplog" solution.

**Junio C Hamano** and **Phillip Wood** converged on the template design, resolving the last open question. The series is **ready for Junio’s final review**, with no further design changes expected. A follow-up may address whether `--reedit-message` should be the default.

---

### Memory leaks in non-default hash implementations
**Jeff King** posted a 9-patch series fixing memory leaks in Git’s hash implementations when using OpenSSL or libgcrypt. The leaks were discovered via `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. Key changes:
- **Patch 1**: Removes redundant `discard_hashfile()` (approved by Junio).
- **Patch 2**: Introduces `git_hash_discard()` (initially inefficient but pragmatic).
- **Patch 9**: Replaces the dummy-buffer implementation with platform-specific discard functions.

The series is **technically sound** and unlikely to be controversial, as it addresses correctness issues without changing behavior. **Junio approved patch 1**, and the rest await review. The leaks only affect non-default configurations, limiting the impact.

---

### Reftable hardening against corruption
**Patrick Steinhardt**’s 12-patch series hardens the reftable backend against maliciously corrupted files. The series includes:
- **Fuzzing infrastructure**: Meson support for libFuzzer and a reftable fuzzer target.
- **Test helper**: `cl_reftable_write_block` to reduce boilerplate in unit tests.
- **Fixes**: 9 patches addressing out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory.

**Christian Couder** and **Junio** acknowledged the test helper improvement, and the series is **ready for substantive review**. The fuzzer has run for 2+ hours without surfacing new issues, indicating the fixes are comprehensive.

---

## In brief

**`git rev-list --exclude-first-parent-only` bug** -- Michael Hore reported a logic flaw in `process_parents()` where explicit commits cause incorrect exclusions. No patch yet, but the diagnosis points to `revision.c`.

**`git apply` memory leak and state corruption** -- A standalone patch fixes a leak in `find_header()` where abandoned Git-style diff headers corrupt subsequent parsing. The fix is minimal and well-motivated.

**`git rm -n *.json` recurses unexpectedly** -- Евгений Плискин reported that `git rm -n` with a glob pattern recurses into subdirectories, contradicting the documentation. No patch yet; the issue may involve shell glob expansion.

**`git rev-parse --parseopt` exit code fix** -- Brian M. Carlson’s merged series standardized help-flag exit codes, but **Jeff King** identified a shell-quoting bug in the `SVN` test prerequisite. A follow-up patch is forthcoming.

**ODB refactoring** -- Patrick Steinhardt’s 6-patch series refactoring `struct object_info` to use `source_infop` is **conceptually approved** by Junio. The series is ready for `next` after a minor reroll.

**CI and test improvements** -- Toon Claes’s 9-patch series makes `GIT_TEST_LONG` tests reliable and efficient for CI. **SZEDER Gábor** and **Jeff King** raised substantive questions about patch 3 (memory usage vs. runtime), but the rest are uncontroversial.

**Git v2.55.0 released** -- Junio announced the release, drawing lighthearted commentary about the inclusion of "Claude Sonnet 4.6" in the contributor list.

**Git for Windows 2.55.0(2) hotfix** -- Johannes Schindelin re-enabled NTLM authentication as an opt-in deprecated feature, addressing a premature disabling in 2.55.0.

**Test modernization** -- A patch modernized `t9811-git-p4-label-import.sh` to use `test_path_is_file` and `test_path_is_missing`.

**Git Rev News edition 136** -- Christian Couder announced the latest edition of the monthly digest.

---

## On the radar

**`git history squash` recoverability** -- **Patrick Steinhardt** proposed an "oplog" to enable atomic undo of multi-ref operations, a long-term solution to the recoverability gap exposed by `--update-refs`. **Junio** clarified that the reflog was never designed for undo, reinforcing the need for a separate system.

**`git replay --linearize` philosophical debate** -- The CLI design divergence from `git rebase` remains unresolved but is now documented. Future CLI changes may require explicit justification.

**`GIT_TEST_LONG` test labeling** -- **SZEDER Gábor**’s review of patch 3/9 raises a policy question: should the `EXPENSIVE` prerequisite account for memory usage, not just runtime? No resolution yet.