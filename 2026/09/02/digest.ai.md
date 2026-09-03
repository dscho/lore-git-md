# Git mailing list daily digest for 2026/09/02

## The day in brief
The Git mailing list saw active development across several fronts today. Key highlights include Junio C Hamano’s final sign-off on the `--missing-only` feature for `git rev-list`, a critical flaw identified in Elijah Newren’s shallow clone push optimization, and the rejection of a `hooks.allowNoVerify` configuration proposal. The day also featured substantive discussions on ODB refactoring, CI modernization, and argument parsing improvements.

## Notable threads

### GitLab’s `--missing-only` for `git rev-list` approved and ready for `next`
**What changed**: Junio C Hamano provided final sign-off on Siddharth Asthana’s `--missing-only` option for `git rev-list`, which enables output of only missing object IDs without post-processing. The feature supports GitLab’s Gitaly workflow for single-pass transaction packing.

**Why it matters**: This feature provides a script-friendly way to list only missing objects (without `?` prefix) while preserving existing `--missing=` formatting options. It integrates cleanly with rev-list’s output controls and is now approved for integration.

**Key technical details**:
- Files: `builtin/rev-list.c`, `Documentation/rev-list-options.adoc`, `t/t6202-rev-list-missing.sh`
- New flag: `--missing-only` (filters output to missing objects only)
- Output format: one OID per line (no `?` prefix), or `path=`/`type=` fields if `--missing=print-info` is used
- Incompatible options: `--count` and `--disk-usage` are explicitly rejected

**Today’s delta**: [2026/09/02/21-26-03] Junio encouraged an optional commit message tweak to better "sell" the patch, marking the feature as ready for `next`.

---

### Critical flaw identified in shallow clone push optimization
**What changed**: Derrick Stolee identified a critical flaw in Elijah Newren’s patch series improving `git push` performance from shallow clones. The patch can over-exclude objects needed by other refs in a multi-ref push when one ref is rejected for shallow-update reasons, risking repository corruption.

**Why it matters**: The flaw undermines the patch’s safety guarantees, making it unsuitable for merging in its current form. The discussion now centers on whether to adopt Stolee’s negotiation-based approach or refine Newren’s original patch.

**Key technical details**:
- Files: `send-pack.c`, `Documentation/config/push.txt`, `t/t5538-push-shallow.sh`
- New config: `push.shallowExcludeBoundary` (boolean, default `true`)
- Failure mode: Multi-ref pushes where one ref is rejected for shallow-update reasons but others depend on excluded objects

**Today’s deltas**:
- [2026/09/02/18-23-55] Stolee identified the over-exclusion flaw, providing a test case demonstrating the corruption risk.
- [2026/09/02/19-05-50] Stolee reinforced the flaw as a critical correctness issue, questioning the default assumptions of Newren’s patch.
- [2026/09/02/20-57-00] Newren refuted the corruption concern, clarifying that `receive-pack` enforces connectivity checks, making the over-exclusion issue a false-positive failure rather than corruption.

---

### `hooks.allowNoVerify` configuration proposal rejected
**What changed**: Junio C Hamano rejected Alessio Attilio’s proposal to introduce a `hooks.allowNoVerify` configuration option that would prevent accidental bypass of hooks via the `--no-verify` flag. The rejection was reinforced by brian m. carlson’s philosophical opposition.

**Why it matters**: The rejection highlights Git’s design principles around user-controlled hooks and the project’s resistance to introducing configuration layers that restrict CLI overrides. The discussion effectively closes the door on the current design.

**Key technical details**:
- New config key: `hooks.allowNoVerify` (string, accepts `true`/`warn`/`false`; defaults to `true`)
- Commands affected: `commit`, `push`, `merge`, `rebase`, `am`
- Emergency overrides: `GIT_ALLOW_NO_VERIFY=1` environment variable and `-c hooks.allowNoVerify=true` CLI flag

**Today’s deltas**:
- [2026/09/02/19-21-01] Junio rejected the feature, citing concerns about configuration complexity creep and the narrow scope of the feature.
- [2026/09/02/22-14-41] Carlsson opposed the feature on philosophical grounds, arguing that `--no-verify` is a deliberate user choice and that Git should not introduce configuration layers to restrict it.

---

### ODB refactoring continues with v2 of 13-patch series
**What changed**: Patrick Steinhardt posted v2 of his 13-patch series refactoring the object database (ODB) subsystem to remove ad-hoc source linking. The series continues the `the_repository` removal effort and addresses Junio’s feedback from v1.

**Why it matters**: This refactoring simplifies the ODB subsystem by ensuring the sources list contains only alternates, paving the way for future performance improvements in alternate handling. The series is a key part of the ongoing ODB abstraction effort.

**Key technical details**:
- Files touched: `odb.c`/`odb.h`, `submodule-config.c`, `builtin/grep.c`, `tmp-objdir.c`/`tmp-objdir.h`, test helpers
- Symbols removed: ODB source registration functions, `submodule_source_paths` string list
- New behavior: None; the series is purely refactoring

**Today’s deltas**:
- [2026/09/02/13-34-49] Steinhardt posted v2 of the series, incorporating Junio’s feedback on patch 1/13.
- [2026/09/02/10-23-26] Steinhardt accepted Junio’s suggestion to encapsulate the repository dependency within `cache-tree.c`.

---

### CI modernization: Asciidoctor version bump and Rust support
**What changed**: Jeff King posted a two-patch series to bump the Asciidoctor version tested in GitLab CI from 1.5.8 to a system-provided version, addressing an outdated version pin. Johannes Schindelin also posted a patch series enabling Rust compilation in Git’s GitHub Actions Windows CI jobs.

**Why it matters**: These changes modernize Git’s CI infrastructure, aligning it with user environments and unblocking the broader Rustification effort. The Asciidoctor update eliminates an eight-year-old version pin, while the Rust support ensures Cargo produces a static library compatible with MinGW’s `ld.exe`.

**Key technical details**:
- Files touched: `.github/workflows/main.yml`, `ci/lib.sh`, `ci/install-dependencies.sh`, `config.mak.uname`, `Makefile`
- New behavior: CI workflows use GitHub’s pre-installed Rust toolchain, configured to target the GCC ABI

**Today’s deltas**:
- [2026/09/02/07-11-13] Peff posted a two-patch series to bump the Asciidoctor version.
- [2026/09/02/06-25-48] Junio flagged a discrepancy in the Rust target triple mapping for UCRT64.

---

### New `parse-options` sub-API for early argument scanning
**What changed**: Christian Couder posted a six-patch series introducing a new `parse-options` sub-API (`early_scan_options()`) to fix argument-parsing bugs where hand-rolled early scans fail to account for option values that appear as separate arguments.

**Why it matters**: The new API addresses a recurring class of bugs in commands like `git bisect`, `git rev-parse`, and `git fast-import`, where early scans misinterpret option values as special flags like `--`. The series is well-motivated and thoroughly tested but faces design skepticism from Junio.

**Key technical details**:
- New functions: `early_scan_options()`, `early_scan_options_from_options()`
- Files touched: `parse-options.h`, `parse-options.c`, `builtin/bisect.c`, `builtin/rev-parse.c`, `fast-import.c`, test files
- Bugs fixed: Misparsing of `--` and special flags when option values appear as separate arguments

**Today’s deltas**:
- [2026/09/02/16-10-41] Couder posted the six-patch series.
- [2026/09/02/18-52-23] Junio raised substantive design concerns, questioning the API’s long-term viability and the practical value of the `git bisect` and `git rev-parse` fixes.

---

### Bugfixes and usability improvements
**What changed**: Several bugfixes and usability improvements were posted today, including:
- A memory leak fix in the submodule error path by Jeff King.
- A NULL-pointer dereference fix in `git history split` by zkd18cjb@mail.ustc.edu.cn.
- A buffer overflow fix in `match_pathspec_with_flags()` by Yannik Tausch.
- An edge case fix in `common_prefix_len()` by Yannik Tausch.
- An inconsistency in newline handling between `git commit` and `git merge` reported by Simon Cheng.

**Why it matters**: These fixes address real-world issues, from crashes to subtle misbehavior, improving Git’s robustness and user experience. The memory leak fix is particularly notable for its thorough exploration of the `parsed_object_pool` API’s asymmetry.

**Key technical details**:
- Files touched: `builtin/history.c`, `dir.c`, `t/unit-tests/u-dir.c`, `t/t6132-pathspec-exclude.sh`
- Subsystems: pathspec-matching, submodule handling, commit message processing

**Today’s deltas**:
- [2026/09/02/05-51-17] Peff posted a two-patch bugfix series for the submodule memory leak.
- [2026/09/02/12-07-36] zkd18cjb@mail.ustc.edu.cn posted a bugfix for the `git history split` crash.
- [2026/09/02/12-22-39] Yannik Tausch posted a bugfix for the pathspec buffer overflow.
- [2026/09/02/13-04-09] Yannik Tausch posted a bugfix for the `common_prefix_len()` edge case.
- [2026/09/02/14-04-37] Simon Cheng reported the newline handling inconsistency.

---

## In brief
- **[2026/09/02/04-35-59]** Siddharth Shrimali posted a final GSoC 2026 submission report for the disk space recovery project, confirming the project is feature-complete.
- **[2026/09/02/11-30-45]** Yuvraj Singh Chauhan posted a two-patch series moving the global `grafts_keep_true_parents` configuration variable into the repository-specific `repo_config_values` struct.
- **[2026/09/02/13-20-42]** Kaartic Sivaraam volunteered to co-mentor a project and/or serve as an org admin for Git’s Outreachy December 2026 cohort.
- **[2026/09/02/14-42-19]** Karthik Nayak strengthened the commit message’s rationale for the `receive-report` hook, addressing Junio’s consistency concern.
- **[2026/09/02/15-07-23]** Thomas Bachem agreed to implement Phillip Wood’s preferred approach for fixing the `rerere` lock race condition: disabling background maintenance during rebase.
- **[2026/09/02/17-04-31]** Kristoffer Haugsbakk and Julia Evans finalized two purely stylistic tweaks to the `gitdatamodel(7)` documentation.
- **[2026/09/02/19-14-34]** Junio highlighted a fundamental consistency concern with the `receive-report` hook: its ability to report a state divergent from the actual repository state.
- **[2026/09/02/19-52-30]** Junio accepted the iteration of Harald Nordgren’s `git checkout -m` autostash conflict handling series as "good to me."
- **[2026/09/02/21-05-11]** Junio requested a final reroll of the `dk/use-nsec-runtime` series to consolidate `NO_NSEC` build-time differences.
- **[2026/09/02/22-40-54]** Junio acknowledged Karthik Nayak’s feedback on the `git checkout` libification series, resolving a correctness issue in the third patch.