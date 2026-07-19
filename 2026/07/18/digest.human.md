Here’s the digest for **2026/07/18**, a Saturday with moderate traffic (47 emails across 16 threads). The day was dominated by follow-ups to long-running series, regression reports, and design discussions—no major milestones, but several threads edged closer to resolution.

---

### The day in brief
A quiet but productive Saturday: **two regression reports** (one blocking a queued series, one for an experimental command) and **design debates** over protocol capabilities (`no-ref-delta`) and Bloom-filter optimizations (`git last-modified`) kept the list active. The `git cat-file --batch-command` series cleared its last build hurdles, while `git history squash` and the sequencer’s dropped-commit notes fix await final polish. A first-time contributor’s `wt-status` patch landed with full reviews.

---

### Notable threads

#### **`git cat-file --batch-command` remote-object-info: v20 clears build issues**
**Headline**: Pablo Sabater’s 20-iteration series enabling remote object metadata queries (size, future type) is now **technically complete** and queued for `next`. Today’s emails resolved the last build-breaking issue: a signed/unsigned comparison warning in `write_fetch_command_and_capabilities()` (SZEDER Gábor, Pablo Sabater). The fix—changing `int` to `const unsigned int`—was mechanical and uncontroversial. Junio’s prior "ready for `next`" stance still holds, pending minor cosmetic nits (translation context, alphabetical ordering). The series is a **GSoC success story**, with 747 lines of new tests and robust input validation (10,000-object batch limit, 8KB URL cap).

**Why it matters**: This is the plumbing for a long-awaited feature—efficient object metadata queries without full downloads—critical for partial clones and write-through proxies. The dynamic capability negotiation (patch 13/13) ensures forward compatibility as the protocol expands.

---

#### **Sequencer regression: dropped-commit notes still copied in interactive rebase**
**Headline**: Uwe Kleine-König reported a **blocking regression** in Phillip Wood’s recently approved sequencer series: notes from dropped commits are still copied in interactive rebase (`-i`), though the fix works for non-interactive rebase. Phillip’s follow-up clarified the root cause: the series only handles commits dropped by `--empty=drop` (non-interactive default), not **user-initiated drops** (e.g., `git rebase --skip` or `--continue` without committing). The interactive backend’s `--empty=ask` quirk (drops on `--continue`, keeps on `--skip`) further complicates the fix.

**Why it matters**: This undermines the series’ core behavioral fix. Phillip’s proposed solution—revising `--empty=ask` to keep empty commits on `--continue` and drop them on `--skip`—would be a **breaking change**, requiring careful coordination. The thread is now a **high-priority follow-up** for the series, which was otherwise ready for `next`.

---

#### **`git history squash`: v9 addresses all feedback, ready for integration**
**Headline**: Harald Nordgren’s `git history squash` series (v9) is **functionally complete** and ready for Junio’s queue. Today’s emails resolved the last minor nits: Matt Hunter’s documentation feedback (redundant sentences, paragraph structure) and a reflog message truncation issue (acknowledged for future work). Junio’s "Will replace" sign-off on v7 still stands, and the series now aligns with `git rebase -i --autosquash` for `fixup!`/`squash!`/`amend!` handling.

**Why it matters**: This command is a **foundational piece** of Git’s future history-editing architecture. Junio’s endorsement of using `git history squash` and `git replay` as a faster, non-interactive backend for `git rebase --autosquash` elevates it to a **project-level aspiration**, with the eventual goal of replacing `git rebase -i`’s internals entirely.

---

#### **`no-ref-delta` capability: design debate reopens**
**Headline**: Jeff King (Peff) and Taylor Blau reopened the design discussion for the `no-ref-delta` protocol capability. Peff’s latest email probed whether the stricter `no-ref-delta` (banning `REF_DELTA` outright) is necessary, or if a narrower "bases-before-deltas" capability (enforcing Git’s default ordering) could achieve the same performance goals with broader compatibility. Taylor defended the stricter design, arguing that even a backward `REF_DELTA` requires OID lookups, complicating the write-through proxy’s offset-based reconstruction.

**Why it matters**: This is a **protocol-level decision** with long-term implications. The write-through proxy’s use case is real, but the trade-off between strict enforcement and protocol flexibility remains contested. No consensus yet, but the thread is now focused on **technical trade-offs** (memory overhead, pipeline stalls) rather than motivation.

---

#### **`git last-modified` Bloom-filter optimization: correctness vs. performance**
**Headline**: Taylor Blau and Jeff King resolved the `--show-trees` correctness bug in Toon Claes’s `git last-modified` series. The issue: Bloom filters only cover the original pathspec (e.g., `d/a`), but `--show-trees` adds ancestor tree entries (e.g., `d/`) to the output, making the pre-filter unsafe. The agreed solution: **remove uncovered ancestor trees** from the output, preserving the performance benefit while avoiding misleading results. Taylor provided a tested code snippet implementing this in `add_path_from_diff()` using `match_pathspec()`.

**Why it matters**: This is a **design pattern** for future Bloom-filter optimizations—balancing correctness with performance by filtering output rather than disabling the optimization. The series is now **technically sound** but awaits resolution of Peff’s maintainability concerns in patch 4/4 (managing `bloom_filter_settings` independently of `revision.c`).

---

### In brief
- **`the_repository` removal**: Tian Yuchen’s v7 series to migrate `trust_executable_bit` and `has_symlinks` hit a **build-breaking issue** (SZEDER Gábor). The fix—marking an unused `repo` parameter as `UNUSED`—is straightforward but blocks integration.
- **Trace2 recursion crash**: Derrick Stolee and Taylor Blau converged on a **two-part fix**—Taylor’s `wrapper.c` change to disable `libintl_vsnprintf()` redirection for `xsnprintf()`, and Derrick’s patch to harden trace2 against reentrancy.
- **`git bisect --auto-reset`**: Johannes Sixt’s **naming suggestion** (`--reset-when-found`) and **platform-compatibility concern** (Windows file deletion) remain unresolved but are non-blocking.
- **`git repack --drop-filtered`**: Christian Couder’s **CLI design feedback** (syntax alternatives) was acknowledged but not addressed in the RFC.
- **Swift userdiff driver**: Johannes Sixt’s **substantive review** (regex optimizations, test ambiguities) prompted a v2, but the patch remains queued in `seen` pending validation from Swift users.
- **ODB abstraction**: SZEDER Gábor identified a **build-breaking issue** in Patrick Steinhardt’s loose-object backend refactoring (missing evil merge resolution). A v2 is required.
- **`wt-status` performance**: Sahitya Chandra’s **first-time contribution** (O(n log n) refactoring) landed with full reviews (Peff, Patrick Steinhardt). The patch is **ready for integration**.
- **`git history fixup` worktree bug**: Phillip Wood proposed adopting `git rebase --update-refs` behavior to prevent index corruption in other worktrees. This is a **safety gap** in the new `git history` command.
- **`git branch -d` error messages**: René Scharfe’s patch to clarify checkout reasons (e.g., "for bisect") sparked a **design discussion** about a broader branch-safety API. Junio’s feedback frames this as a template for future work.

---

### On the radar
- **Sequencer regression**: Phillip Wood’s fix for user-initiated drops in interactive rebase. **High priority**—blocks the series from `next`.
- **`no-ref-delta` capability**: Taylor Blau and Peff’s design debate. **Protocol-level decision** with long-term implications.
- **`git last-modified`**: Peff’s maintainability concerns in patch 4/4. **Needs resolution** before the series can proceed.
- **`git history fixup` worktree bug**: Phillip Wood’s proposed safety mechanism. **Critical for the new command’s stability**.