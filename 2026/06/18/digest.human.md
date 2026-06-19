## The day in brief.

*2026-06-18* was a busy but focused day on the Git mailing list, with **98 emails across 28 threads**. The standout developments: **`git history squash` landed in `next`**, resolving a long-running design debate; **`git branch --delete-merged` reached v16**, addressing all prior feedback; and **ODB abstraction work advanced** despite lingering refdb issues. Documentation updates (trailers, `SubmittingPatches`) and regression fixes (Windows keychain, zlib) also featured prominently. The tone was collaborative, with most threads converging toward resolution.

---

### Notable threads

### `git history squash` queued in `next`

Harald Nordgren’s four-patch series implementing `git history squash`—a tool to fold a commit range into its oldest commit while preserving descendants—was **merged to `next`** after Junio C Hamano applied minor test tweaks. The design pivoted from a `rebase --squash` option to a standalone `git history` subcommand, addressing efficiency concerns (single three-way merge vs. repeated rebase stops) and aligning with Patrick Steinhardt’s `git history` roadmap. The `--reedit-message` option now seeds the editor with *all* folded-in messages, not just the oldest, matching `git rebase -i` behavior. Phillip Wood’s earlier objections were fully resolved, and the series is now poised for `master`.

---

### `git branch --delete-merged` reaches v16

Harald Nordgren’s **16-iteration series** adding `git branch --delete-merged` (safe automated branch cleanup) is now **technically complete**. The v16 patches address all prior feedback, including:
- A **flags-based API** for `delete_merged_branches()` (replacing boolean parameters).
- **Stacked-branch safety** (omitted as unnecessary per Junio’s feedback).
- **Per-branch opt-out** via `branch.<name>.deleteMerged` config.
- **`--dry-run`** preview functionality.
The series is ready for final review, with no known blocking issues. Phillip Wood’s high-weight feedback has been incorporated, and the implementation now matches project API consistency expectations.

---

### ODB abstraction work advances despite refdb blocker

Patrick Steinhardt’s **17-part ODB abstraction series (v3)** completed its mechanical changes, exposing deeper **refdb architectural flaws** (memory leaks, duplicate refdb creation, initialization ordering). Junio C Hamano acknowledged the series as **mechanically sound** and ready for integration once the blocking refdb issues are resolved. Jeff King (Peff) flagged a minor type conflict in patch 15/17 (resolved via evil merge), while Justin Tobler provided surface-level mechanical reviews. The series represents the **final large-scale ODB restructuring** Patrick plans to undertake; future work will be incremental.

---

### Documentation updates: trailers and `SubmittingPatches`

Kristoffer Haugsbakk’s **five-part series** updating `SubmittingPatches` for trailer conventions was **merged to `master`**, with a post-merge follow-up addressing AsciiDoc anchor placement. The series standardizes:
- **Trailer principles** (e.g., `Helped-by`, `Co-authored-by`).
- **Linux-specific trailer avoidance** (`Fixes:`, `Link:`).
- **`Based-on-patch-by`** trailer (with DCO requirements).
- **Chronological ordering** (e.g., `Signed-off-by` last).
Separately, Michael Montalbo and Junio refined wording in `SubmittingPatches` to clarify design critique expectations, emphasizing that contributors should **defend high-level design before implementing**.

---

### Regression fixes: Windows keychain, zlib, and HTTP auth

- **Windows keychain build regression**: Toon Claes’s patch fixing a link error in `contrib/credential/osxkeychain` (when Rust support is enabled) was **merged to `master`**. The fix aligns the Makefile with Meson’s handling of Rust dependencies.
- **zlib clamping bug**: Johannes Schindelin’s patch correcting a `size_t` clamping error in `git-zlib.c` (affecting >4GB buffers) was **approved by Junio** as an "obvious fix."
- **HTTP auth test failure**: Todd Zullinger confirmed that **curl 8.21.0-rc3** resolves the regression in `t5563.18`, eliminating the need for Git-side changes.

---

### In brief
- **`fetch.followRemoteHEAD` global config**: Matt Hunter’s seven-patch series adding a global fallback for `remote.<name>.followRemoteHEAD` was **merged-ready** after Junio accepted the final design resolution (warning for invalid values).
- **Ref-filter performance regression**: Tamir Duberstein’s fix for a 25x slowdown in `git branch --remotes` (introduced in Git 2.51.0) was **merged to `maint` and `next`**.
- **Reference backend refactoring**: Patrick Steinhardt’s eight-patch series modernizing ref backend initialization **resolved its last design question** (recursive refdb initialization) after Jeff King’s critique. The series is now uncontroversial.
- **Interactive typo correction**: A patch adding interactive prompts for mistyped Git commands (e.g., `git comit`) was **questioned by Justin Tobler** for overlapping with `help.autoCorrect=prompt`. The author may need to justify the default behavior change.
- **Line-log extensions**: Karthik Nayak’s seven-patch series adding range-scoped `--stat`, `--check`, and `-G` to `git log -L` received **Junio’s approval** for patch 5/7, with minor documentation tweaks requested.

---

### On the radar
- **Pinned references**: Erik Östlund’s RFC for "pinned references" (ref + expected OID) remains in early discussion. The strawman syntax (`refs/tags/v1.2.3?oid=a1b2c3d4`) and use cases (deployment, CI/CD) need validation.
- **Zsh completion bug**: Lutz Lengemann’s patch fixing `git -C <path> <command>` completion in zsh **needs iteration** to address D. Ben Knoble’s feedback on variable propagation and chained `-C` options.
- **`ignore_case` libification**: Justin Tobler and Tian Yuchen’s series moving `ignore_case` into `struct repo_config_values` is **ready for integration** after addressing Junio’s feedback on getter design.