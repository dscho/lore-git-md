# Here is the digest for **2026/07/12**:

---

### The day in brief
A Sunday with moderate traffic (51 emails across 15 threads), dominated by **two major series nearing completion**—the `remote-object-info` security-hardened feature (now ready for `next`) and the `git log --graph` visual root indentation (stalled on a usability decision). A **Cygwin IPv6 regression fix** sparked platform-compatibility debate, while a **libification series** saw active review. Off-topic noise included a partnership proposal and a bug report about a crash in fsmonitor when built without Rust.

---

### Notable threads

#### **`remote-object-info` security-hardened feature ready for `next`**
The 16-patch series adding `git cat-file --batch-command remote-object-info` (allowing clients to query object metadata from remotes without downloading full objects) received **final review confirmation** from Chandra Pratap. All technical issues—including a critical refactoring flaw in v15, memory leaks, and dynamic capability-based placeholder validation—are resolved. The series is **complete, well-documented, and memory-safe**, with **699 lines of new tests** covering edge cases across all transport types. Junio is now expected to merge it to `next`.

#### **`git log --graph` visual root indentation: usability decision pending**
The v9 series introducing cascading indentation for excluded parents in `git log --graph` remains stalled on a **late-stage usability choice** between two bounded wrapping designs:
- **Two-level wrap** (Mirko Faina’s proposal): Resets indentation after two levels, avoiding excessive indentation in simple cases.
- **Four-level wrap** (Pablo Sabater’s counter-proposal, now endorsed by Mirko): Resets after four levels to eliminate symmetry-induced ambiguity in edge cases.

Both designs address prior objections (e.g., Chandra Pratap’s concerns about visual ambiguity) and require no changes to the approved lookahead buffer infrastructure. The maintainers must rule on the indentation style and whether to retain the `--no-graph-indent` opt-out flag before finalizing patch 4/4. The series is otherwise **technically complete and ready for `next`**.

#### **Libification series: `repo_config_values` migration advances**
Tian Yuchen’s v10 series migrating nine global configuration variables into `struct repo_config_values` saw **active review** from Pablo Sabater and Junio. Key developments:
- **Destructor signature resolved**: `repo_config_values_clear()` now takes a `struct repo_config_values*` (enabling submodule support and avoiding initialization-order crashes).
- **Memory safety**: Fixed leaks in `git_apply_config()` and redundant calls to `repo_config_values()` across patches 4–9.
- **Behavioral change**: `repo_config_values()` now asserts `repo == the_repository` (documented in v11).
- **Missing cleanups**: Junio flagged redundant global variables (`askpass_program`) and requested `FREE_AND_NULL()` in `git_apply_config()`.

The series is **CI-clean and memory-leak-free**, with v11 expected to address the remaining nits. It represents a solid step in the `the_repository` removal effort.

#### **Cygwin IPv6 regression fix: platform-compatibility debate**
Ramsay Jones’s patch to fix a five-year-old regression in IPv6-style SCP URL handling on Cygwin (e.g., `[::1]:repo`) sparked a **substantive review** from Torsten Bögershausen. The core dispute: whether to remove a conditional in `connect.c` that Torsten argues is critical for Git for Windows’ UNC path support. Ramsay’s earlier suggestion to replace `#ifdef DUMMY` with `#ifdef GIT_WINDOWS_NATIVE` remains a possible resolution. The patch is **technically sound** but blocked on this compatibility concern.

#### **`GIT_NO_LAZY_FETCH=fromAccepted`: security debate unresolved**
Christian Couder’s series extending `GIT_NO_LAZY_FETCH` to support a `fromAccepted` mode (allowing lazy fetching only from mutually agreed promisor remotes) faced a **high-weight security objection** from brian m. carlson. Christian countered that the mode is more restrictive than the existing `GIT_NO_LAZY_FETCH=0` (which disables all protections) and proposed future improvements (e.g., server-side trusted repository marking). The objection remains unresolved, but Christian committed to improving documentation in v2.

---

### In brief

**`git-subtree` safeguard** -- Ian Jackson confirmed he will post v2 of the safeguard series (detecting Rust rewrite usage) after travel, with no technical changes beyond test style improvements. Junio reaffirmed the "Expecting a reroll" status.

**`git repo info` prefix querying** -- K Jayatheerth abandoned the feature for `git repo info` (a plumbing command), pivoting to a different GSoC task ("histograms patch") after Junio’s design critique.

**Build system fix** -- Ramsay Jones’s one-line fix for a sparse error in `pack-revindex.c` (due to the `lib/` directory reorganization) was queued in `seen`. Johannes Schindelin and Junio confirmed the fix is correct and should be squashed into the original commit if the branch is re-rolled.

**Documentation clarifications** -- Junio’s six-patch series clarifying contributor guidance saw **editorial tweaks** in response to feedback:
- Revised commit-message template: **"Implementation (The Changes)"** replaces "Command (The Instruction)".
- Accepted `====` for AsciiDoc section markers and reflowed an awkward paragraph in `MyFirstContribution.adoc`.
- No substantive objections to the series’ core goals (e.g., disambiguating the `seen` branch, trailer semantics).

**Sequencer bugfix** -- Farid Zakaria’s patch making `git rebase --autosquash` respect `--empty` for emptied commits is **ready for integration** but must wait for Phillip Wood’s `pw/rebase-drop-notes-with-commit` topic to graduate to `next`.

**fsmonitor crash** -- Đoàn Trần Công Danh reported a `BUG_vfl` crash in `fsm-listen-linux.c` when Git is built without Rust (`NO_RUST=Yes`). The crash occurs during a rename operation and is reproducible on `next`, `seen`, and the upcoming Git 2.55.0 release. No fix yet.

**Debug-commit RFC** -- Simon Richter proposed a standardized `drop!` prefix for debug commits to be automatically removed during interactive rebase. Junio noted the feature could be implemented today via a `GIT_SEQUENCE_EDITOR` script, but no patches or deeper discussion emerged.

---

### On the radar
- **`git log --graph` indentation decision**: Maintainers must choose between the two-level and four-level wrapping designs.
- **fsmonitor crash**: A regression in a performance-critical feature, warranting prompt attention before Git 2.55.0.
- **`GIT_NO_LAZY_FETCH=fromAccepted`**: Security objection remains unresolved; v2 expected with improved documentation.