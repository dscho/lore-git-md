Here is the digest for **2026/07/12 (Sunday)**, covering 51 emails across 15 active threads.

---

### The day in brief
A quiet Sunday on the Git mailing list, with **51 emails** spread across **15 threads**. The most consequential development: **Pablo Sabater’s `remote-object-info` security-hardened feature series (v16) received final review confirmation** and is now **ready for merging to `next`**, marking a major milestone in remote object metadata querying. Elsewhere, **Tian Yuchen’s libification series (v10)** advanced with resolved design debates, while **Mirko Faina and Pablo Sabater converged on a four-level wrapping design** for `git log --graph` indentation, clearing a late-stage usability blocker. A **Cygwin IPv6 URL regression fix** remains contested over UNC path compatibility, and a **new fsmonitor crash** was reported in builds without Rust support.

---

### Notable threads

#### **`remote-object-info` security-hardened feature ready for merging**
**Headline:** Chandra Pratap’s **final review confirmation** marks Pablo Sabater’s **16-patch series** as **ready for Junio’s merge to `next`**. The series adds a new `remote-object-info` command to `git cat-file --batch-command`, enabling clients to query object metadata (e.g., size) from remote repositories without downloading full objects. Key security improvements in v16 include:
- **Dynamic capability-based placeholder validation**: Runtime filtering of format placeholders (e.g., `%(objectsize)`) against server-advertised capabilities, preventing crashes or injection risks.
- **Memory safety**: Fixed leaks in `transport-helper.c` and improved loop counter hygiene.
- **Protocol compliance**: Strict v2 enforcement, input size limits (10,000 objects max), and silent continuation for unsupported fields (matching Git’s established patterns).
- **Test coverage**: **699 lines** of new tests in `t/t1017-cat-file-remote-object-info.sh`, verifying edge cases across all transport types.

The series is **complete and well-documented**, with all technical issues resolved. The only open question—whether to fail explicitly or silently when metadata is missing—has been documented and does not block integration. This is a **significant step forward** for remote object querying, with potential applications in partial clone workflows and repository analysis tools.

---

#### **`git log --graph` indentation: four-level wrapping design emerges as consensus**
**Headline:** Mirko Faina and Pablo Sabater **converged on a four-level wrapping design** for visual root indentation in `git log --graph`, resolving a late-stage usability debate. The design caps indentation at four levels before resetting to the left margin, creating a visually abrupt transition that reinforces the unrelatedness of commits and avoids symmetry-induced ambiguity in edge cases (e.g., when the number of visual roots is a multiple of 3 or 4).

**Key developments:**
- Mirko initially proposed a **two-level wrap** (resetting after two levels) to avoid excessive indentation in simple cases, but Pablo countered with a **four-level wrap** to eliminate ambiguity.
- Both designs address prior objections to the original cascading indentation (e.g., Chandra Pratap’s concerns about visual mimicry of merge topology).
- The underlying **lookahead buffer infrastructure** (approved by Junio) remains unchanged; this is purely a usability refinement.
- The `--no-graph-indent` opt-out flag will be retained to disable indentation in non-oneline formats (e.g., `--format=fuller`).

The series is now **technically complete** and awaits maintainer input on the indentation style. Patch 4/4 in the v9 series may require minor adjustments, but no further architectural changes are expected. This clears the last major blocker for a feature that improves clarity in rebase-heavy workflows.

---

#### **Libification effort advances: `repo_config_values` migration (v10)**
**Headline:** Tian Yuchen’s **nine-patch series** migrating global configuration variables into `struct repo_config_values` reached **v10**, addressing all prior feedback. The series is now **technically complete** and **CI-clean**, with key improvements:
- **Destructor signature alignment**: `repo_config_values_clear()` now takes a `struct repo_config_values*` (resolving Pablo Sabater’s submodule leak concern).
- **Memory safety**: Fixed leaks in `core_pager_config()` and `git_apply_config()`, and cached redundant calls to `repo_config_values()` for performance.
- **Behavioral change**: `repo_config_values()` now asserts `repo == the_repository` via `BUG()`, enforcing a constraint that previously relied on caller discipline (documented in v11).

**Open questions:**
- **RFC questions**: Whether migrating program paths (editor, pager, askpass) is worthwhile (Tian argues yes for per-repo toolchain flexibility).
- **Follow-up work**: Cleanup of `repo_config_values` comments and deferred enum relocations (e.g., `push_default`).

The series is **ready for `next`** and represents a solid incremental step in the `the_repository` removal effort. Reviewers should focus on the updated destructor signature and cached getter calls.

---

#### **Cygwin IPv6 URL regression fix contested**
**Headline:** Ramsay Jones’s **Cygwin IPv6 URL regression fix** hit a snag when Torsten Bögershausen objected to removing a conditional in `connect.c`, citing **UNC path breakage on Git for Windows**. The regression, introduced in the 2019 security releases, misinterprets IPv6-style SCP URLs (e.g., `[::1]:repo`) as invalid paths because `[` is treated as a potential Windows drive letter.

**Key points:**
- Torsten argues the `#ifdef DUMMY` block Ramsay proposes to remove was introduced in 2018 to enable UNC path handling (`file://server/share/repo`) on Git for Windows.
- Ramsay’s earlier suggestion to replace `#ifdef DUMMY` with `#ifdef GIT_WINDOWS_NATIVE` remains a possible resolution.
- The fix restores test coverage in `t5500-fetch-pack.sh` and `t5601-clone.sh`, which have been skipped since 2024.

The thread’s next step is likely a **v2** that preserves UNC logic while addressing Cygwin’s IPv6 parsing. This is a **platform-specific regression** with no broader impact.

---

#### **New fsmonitor crash reported in builds without Rust support**
**Headline:** Đoàn Trần Công Danh reported a **`BUG_vfl` assertion crash** in `compat/fsmonitor/fsm-listen-linux.c` when Git is built with `NO_RUST=Yes`. The crash occurs during test `t7527-builtin-fsmonitor.sh` (test 43) and involves a "double remove of watch" during a rename operation. The issue is reproducible on `next`, `seen`, and the upcoming Git 2.55.0 release.

**Key details:**
- The crash is tied to the Linux-specific fsmonitor listener code and does not affect Windows/macOS backends.
- Recent Rust-related work in fsmonitor may be a contributing factor, though the crash occurs in C code.
- This is a **serious regression** in a performance-critical feature for large repositories.

No fix has been proposed yet, but the test failure provides a clear reproduction case. Given the proximity to the Git 2.55.0 release, this warrants prompt attention.

---

### In brief

#### **`git-subtree` safeguard series**
Ian Jackson confirmed he will post **v2** of the `git-subtree` safeguard series (detecting Rust rewrite usage) in a few days, incorporating test style improvements. The series is procedurally ready for inclusion post-v2.55.

#### **`GIT_NO_LAZY_FETCH=fromAccepted` security debate**
Christian Couder and brian m. carlson remain at odds over the security implications of the `fromAccepted` mode. Christian argues it is more restrictive than the status quo (`GIT_NO_LAZY_FETCH=0`), while brian warns it expands the attack surface. No resolution yet, but Christian committed to improving documentation in v2.

#### **Commit date display discussion**
Oswald Buddenhagen proposed defaulting to **committer date** (with author name) in `git log`, arguing it better reflects when a commit "started to matter to users." The thread remains exploratory, with no consensus on changing the default or introducing a configuration option.

#### **Documentation contributor guidance series**
Junio’s **six-patch documentation series** clarifying contributor guidance saw minor editorial tweaks in response to feedback. Key changes:
- Revised commit-message template: **"Implementation (The Changes)"** replaces "Command (The Instruction)" to clarify the distinction from "Design (The Approach)."
- Agreed to reflow an awkward paragraph in the integration branches description.
- No substantive objections to the series’ core goals (e.g., trailer semantics, `seen` branch disambiguation).

#### **Sequencer `--empty` fix for autosquash**
Farid Zakaria’s **bugfix** for `git rebase --autosquash` (respecting `--empty` for autosquash-induced empty commits) is ready for integration but awaits Phillip Wood’s `pw/rebase-drop-notes-with-commit` topic to graduate to `next`. The patch is technically sound and addresses a real-world workflow pain point (e.g., reverting commits via `fixup!`).

#### **Build system fix for `ps/libgit-in-subdir`**
Ramsay Jones’s **one-line Makefile fix** (adding `lib/` prefix to `pack-revindex.sp`) was queued in `seen` by Junio. The fix addresses a sparse compilation error in Patrick Steinhardt’s `ps/libgit-in-subdir` topic.

#### **Debug-commit mechanism RFC**
Simon Richter proposed a **standardized debug-commit mechanism** (e.g., `drop!` prefix) for automatic removal during interactive rebase. Junio noted this could be implemented today via a `GIT_SEQUENCE_EDITOR` script, but no patches or deeper discussion have emerged.

---

### On the radar
- **Fsmonitor crash**: The newly reported `BUG_vfl` crash in builds without Rust support needs investigation, especially given the upcoming Git 2.55.0 release.
- **`git log --graph` indentation**: Maintainers must rule on the four-level vs. two-level wrapping design before the series can proceed to final review.
- **Libification series**: Tian Yuchen’s v10 is ready for `next`; reviewers should focus on the updated destructor signature and cached getter calls.
- **Cygwin IPv6 fix**: A v2 is needed to address Torsten Bögershausen’s UNC path compatibility concern.