# Here is the digest for **2026/07/20 (Monday)**:

---

### The day in brief
A busy Monday on the Git mailing list, with **54 emails across 17 threads**. The day’s traffic was dominated by **two long-running refactoring efforts**—Patrick Steinhardt’s `lib/` directory reorganization and Tian Yuchen’s `the_repository` removal series—both of which saw **substantive debate and new iterations**. A **feature series for `git history squash` reached its tenth version**, functionally complete and ready for integration, while **bugfixes and documentation updates** filled out the rest of the queue. The tone was **collaborative but contentious in places**, particularly around the `lib/` reorganization, where **Johannes Schindelin and SZEDER Gábor** remain at loggerheads over the trade-offs between codebase hygiene and workflow disruption.

---

### Notable threads

#### **`lib/` directory reorganization: the debate intensifies**
**Thread**: [RFC PATCH v4] Move `libgit.a` sources into `lib/`
**Author**: Patrick Steinhardt
**Status**: Under active debate; no consensus yet

The most **contentious thread of the day** saw **Johannes Schindelin (Dscho)** and **brian m. carlson** double down on their support for Patrick Steinhardt’s proposal to move Git’s core library sources into a `lib/` directory, while **SZEDER Gábor** reiterated his objections about the **persistent disruption** to tools like `git log --follow`. Dscho framed the current root-directory clutter as an "ongoing burden" that outweighs the one-time cost of reorganization, drawing a parallel to historical resistance to Git itself. Brian’s reply was **surface-level but supportive**, calling the current state "odd" and endorsing the cleanup as long-overdue.

The **core disagreement** remains unresolved: proponents (Steinhardt, Dscho, Oswald Buddenhagen) prioritize **structural cleanup and codebase hygiene**, while critics (Gábor, Junio C Hamano, Phillip Wood) emphasize the **ongoing mental overhead** of file moves (e.g., `wt-status.c` → `lib/wt-status.c`) and the **lack of realistic tooling improvements** to mitigate the pain. **Kaartic Sivaraam’s proposal**—adding a hint to `git show` when a file appears renamed—remains a potential middle ground, but no one has yet volunteered to implement it.

**Key takeaway**: The series is **not yet ready for `next`**, but the pro-reorganization camp has **shifted the framing** from "disruption vs. benefit" to "current pain vs. future improvement." The next move likely rests with Patrick, who may need to **propose concrete tooling improvements** (e.g., `git log --follow` enhancements) to break the deadlock.

---

#### **`the_repository` removal: v8 lands, interactive rebase regression noted**
**Thread**: [PATCH v8 0/4] refactor `trust_executable_bit` and `has_symlinks`
**Author**: Tian Yuchen
**Status**: **Ready for `next`**; interactive rebase regression to be addressed in a follow-up

Tian Yuchen’s **four-patch v8 series**—migrating `trust_executable_bit` and `has_symlinks` into `struct repo_config_values`—**addressed all prior feedback**, including **Junio’s stylistic suggestions** and a **build-breaking issue** flagged by SZEDER Gábor. The series is now **mechanically complete and uncontroversial**, with all patches reviewed and approved. However, **Uwe Kleine-König reported a regression**: the fix works for non-interactive rebase but **fails for interactive rebase (`-i`)**, where notes from dropped commits are still incorrectly copied. Phillip Wood clarified that the series only handles commits dropped by `--empty=drop` (non-interactive default), not **user-initiated drops** (e.g., `git rebase --skip` or `--continue` without committing in interactive rebase).

**Key takeaway**: The series is **queued for `next`**, but a **follow-up is needed** to handle the interactive rebase edge case. The migration pattern established here may serve as a template for future configuration refactoring efforts.

---

#### **`git history squash` reaches v10, ready for integration**
**Thread**: [PATCH v10 0/5] `git history squash`: fold commit ranges into a single commit
**Author**: Harald Nordgren
**Status**: **Functionally complete**; Junio’s "Will replace" sign-off on v7

Harald Nordgren’s **`git history squash`**—a new subcommand to fold a commit range into its oldest commit while preserving descendant history—**reached its tenth iteration**, addressing the last minor feedback from v9. The series is now **functionally complete**, with **reflog message accuracy**, **rev-list option sanitization**, and **documentation clarity** all resolved. Junio’s "Will replace" sign-off on v7 signals intent to queue the series for the next release.

The command avoids the **repeated conflict stops** of a rebase-based approach, making it ideal for **automated workflows** (e.g., CI pipelines) and **interactive debugging**. The `--reedit-message` feature mirrors `git rebase -i --autosquash`, grouping `fixup!`/`squash!`/`amend!` commits under their targets. The only unresolved discussion—whether `--reedit-message` should be the default—is a **minor usability tweak** and does not block merging.

**Key takeaway**: The series is **ready for `next`** and positions `git history squash` as a foundational piece of a larger effort to **modernize Git’s history-editing infrastructure**, including potential integration with `git rebase --autosquash`.

---

#### **Rebase notes bugfix: systemic overhaul approved for `next`**
**Thread**: [PATCH v3 0/9] sequencer: avoid copying notes from dropped commits
**Author**: Phillip Wood
**Status**: **Approved for `next`**; interactive rebase regression noted

Phillip Wood’s **nine-patch systemic overhaul**—fixing a bug where `git rebase` incorrectly copies notes from dropped commits—**received maintainer approval** and is **queued for `next`**. The series addresses all known edge cases for `--empty=drop` and carries **Uwe Kleine-König’s Tested-by**. However, **Uwe reported a regression**: the fix **fails for interactive rebase**, where notes from dropped commits are still copied in **user-initiated drops** (e.g., `git rebase --skip`).

**Key takeaway**: The series is **ready for `next`**, but a **follow-up is needed** to handle the interactive rebase edge case. The `enum pick_result` refactoring introduced here may inform future sequencer improvements.

---

#### **`no-ref-delta` capability: design trade-offs debated**
**Thread**: [PATCH 0/4] pack-objects: introduce `no-ref-delta` capability
**Author**: Taylor Blau
**Status**: Under review; no consensus yet

Jeff King (Peff) **engaged deeply** with the technical trade-offs of Taylor Blau’s `no-ref-delta` capability, which enables receivers (e.g., a write-through proxy) to request packs containing only `OFS_DELTA` entries. Peff acknowledged that restricting packs to `OFS_DELTA` could enable **meaningful optimizations** for the proxy’s asynchronous delta resolution pipeline, particularly around **worker coordination and cache locality**. However, he remains **hesitant about merging a protocol capability that Git itself does not use**, citing concerns about **long-term maintainability** and the risk of the feature becoming a mismatch for future optimizations.

**Key takeaway**: The discussion remains **open-ended**, with no clear path forward. Peff’s feedback suggests the proxy’s optimizations might be more compelling if they were also applicable to Git’s native receiving code, but no concrete proposal has emerged.

---

### In brief

- **`git repo info` path keys**: K Jayatheerth’s GSoC series adding path-related keys (e.g., `path.toplevel`, `path.grafts`) saw **substantive feedback** from Lucas Seiki Oshiro and Junio C Hamano. The `path.grafts` behavior (returning paths even when the file does not exist) was **endorsed as intentional**, aligning with `git rev-parse --git-path`. The `path.index` keys may need to return an **empty string in bare repositories** to avoid misleading scripts.
  **Files**: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1900-repo-info.sh`

- **Swift userdiff driver**: Shlok Kulshreshtha’s **v2 patch** addressed all review feedback, but the series remains **queued in `seen`** pending validation from active Swift users. The regex patterns were **empirically validated** against 20,454 hunks from seven real-world Swift projects, showing 99.9% accuracy.
  **Files**: `userdiff.c`, `Documentation/gitattributes.adoc`, `t/t4018/`

- **`git last-modified` Bloom filter optimization**: Jeff King and Taylor Blau **agreed on a fix** for the `--show-trees` correctness bug, using `match_pathspec()` to filter out ancestor tree entries not covered by the original pathspec. The series is **ready for v2**, with only minor edge cases remaining.
  **Files**: `revision.c`, `revision.h`, `builtin/last-modified.c`

- **`git bisect --reset-when-found`**: Harald Nordgren’s **v3 series**—adding an option to automatically reset the working tree after bisect—**received maintainer approval** and is **queued for `next`**. The design is settled, with the rename from `--auto-reset` to `--reset-when-found` addressing Johannes Sixt’s feedback.
  **Files**: `bisect.c`, `builtin/bisect.c`, `Documentation/git-bisect.adoc`, `t/t6030-bisect-porcelain.sh`

- **`git repack --drop-filtered`**: Siddharth Shrimali’s RFC series for partial clones saw **surface-level feedback** from Christian Couder on CLI design. The author reaffirmed the separation of `--drop-filtered` and `--dry-run`, but a **blocking logical flaw** in the `--drop-filtered` vs `--write-bitmap-index` validation remains unresolved.
  **Files**: `builtin/repack.c`, `repack-filtered.c`, `t/t7706-repack-drop-filtered.sh`

- **Documentation synopsis-style conversion**: Jean-Noël Avila’s **four-patch series** converting `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the new AsciiDoc synopsis style **addressed all stylistic feedback** and is **ready for v2**.
  **Files**: `Documentation/git-*.adoc`

- **Push tracking after remote rename**: Harald Nordgren’s **two-patch series**—fixing `git status`’s push-tracking display after remote renames—**received maintainer feedback** requesting a clearer commit message. The series is **ready for v2**, with no technical objections.
  **Files**: `remote.c`, `transport.c`, `Documentation/revisions.adoc`, `t/t5505-remote.sh`

- **`git config --global` and `include.*`**: The thread **resolved the "is this a bug?" question**: the current behavior (not respecting `include.*` by default) is **intentional**, rooted in backwards compatibility. The focus has shifted to **educating downstream projects** (e.g., `lbmk`) about alternatives like `git var GIT_AUTHOR_IDENT`.
  **Files**: None (documentation-only discussion)

- **`contrib/fast-import/import-zips.py`**: Junio C Hamano **declined to accept** Chris Packham’s Python 3 update, favoring **removal of the script** from `contrib/` or replacement with a pointer to an external fork. The broader question of **AI-assisted contributions** remains unaddressed.

---

### On the radar
- **`lib/` directory reorganization**: The debate remains **polarized**, with no clear path to consensus. Patrick Steinhardt may need to **propose concrete tooling improvements** (e.g., `git log --follow` enhancements) to break the deadlock.
- **Interactive rebase regressions**: Both the **rebase notes bugfix** and **`the_repository` removal series** have **unresolved regressions** in interactive rebase. Follow-ups are expected.
- **`no-ref-delta` capability**: The design discussion remains **open-ended**, with no clear path forward. Peff’s feedback suggests the proxy’s optimizations might need to be **applicable to Git’s native receiving code** to justify inclusion.