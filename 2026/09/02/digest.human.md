# Git mailing list daily digest for 2026/09/02

## The day in brief
The Git mailing list saw active discussion on several fronts today. A **`--missing-only` option for `git rev-list`** received final maintainer sign-off, signaling readiness for integration. **GitLab CI documentation jobs** were modernized with a shift to system-provided Asciidoctor versions. **Shallow clone push performance** sparked debate over correctness and false-positive failures. **Outreachy participation** gained momentum with new mentor volunteers. **`git imap-send`** gained a `--draft` option to mark uploaded messages as IMAP drafts. **`git rerere`** saw a proposed fix for lock contention during rebase. **`parse-options`** introduced a new sub-API for early argument scanning, though maintainer feedback questioned its design. **`hooks.allowNoVerify`** was rejected as too narrowly scoped.

## Notable threads

### `--missing-only` for `git rev-list` (2026/04/19/08-48-39)
**What changed?**
Junio C Hamano provided final sign-off on Siddharth Asthana’s `--missing-only` option for `git rev-list`, which outputs only missing object IDs without post-processing. The maintainer encouraged an optional commit message tweak to better "sell" the patch.

### Why it matters

This feature directly supports GitLab’s Gitaly partial clone workflows by enabling efficient identification of missing objects in a single-pass transaction packing operation. The implementation is complete, with all review feedback incorporated, and the topic is queued for `next`.

### Key technical details

- Files: `builtin/rev-list.c`, `Documentation/rev-list-options.adoc`, `t/t6202-rev-list-missing.sh`
- New flag: `--missing-only` (filters output to missing objects only)
- Output format: one OID per line (no `?` prefix), or `path=`/`type=` fields if `--missing=print-info` is used
- Incompatible options: `--count` and `--disk-usage` are explicitly rejected

---

### GitLab CI documentation jobs (2026/09/01/08-16-11)
**What changed?**
Jeff King (Peff) posted a two-patch series to modernize GitLab CI’s Asciidoctor installation. The series replaces the manual `gem install asciidoctor -v 1.5.8` with `apt-get install asciidoctor`, using the Ubuntu image’s default version (2.0.20). This eliminates an outdated version pin (1.5.8, from 2018) and simplifies CI configuration.

### Why it matters

The change aligns CI with user environments, where newer Asciidoctor versions are standard, and reduces maintenance overhead. The series is uncontroversial and ready for merging, with no technical objections.

### Key technical details

- Files: `ci/install-dependencies.sh`
- Behavior: replaces manual `gem install` with system-provided Asciidoctor
- Root cause: outdated version pin (1.5.8) and transitive dependency removal
- Verification: GitHub Actions test run using Asciidoctor 2.0.20

---

### Shallow clone push performance (2026/08/21/06-55-51)
**What changed?**
Derrick Stolee identified a critical flaw in Elijah Newren’s patch to improve `git push` performance from shallow clones. The patch could over-exclude objects needed by other refs in a multi-ref push when one ref is rejected for shallow-update reasons, risking repository corruption. Stolee proposed an alternative: repurposing `push.negotiate` for shallow repos.

### Why it matters

The discussion highlights a fundamental tension between performance and correctness. Newren’s patch assumes the server has the shallow graft, which is usually true but can lead to false-positive failures. Stolee’s alternative avoids this but may resend more data. The thread is now focused on whether to adopt Stolee’s approach or refine Newren’s patch.

### Key technical details

- Files: `send-pack.c`
- New config: `push.shallowExcludeBoundary` (boolean, default `true`)
- Failure mode: multi-ref pushes where one ref is rejected for shallow-update reasons but others depend on excluded objects
- Alternative: `push.negotiate=shallow` mode

---

### Outreachy December 2026 cohort (2026/08/26/10-55-19)
**What changed?**
Kaartic Sivaraam volunteered to co-mentor a project and/or serve as an org admin for Git’s Outreachy December 2026 cohort. Pablo (last name not provided) also expressed interest in co-mentoring.

### Why it matters

The thread is gaining momentum, with three confirmed volunteers (Christian Couder, Usman Akinyemi, Kaartic Sivaraam) and a potential fourth (Pablo). The discussion is now poised to shift toward finalizing project ideas and confirming roles before the September 11 deadline.

### Key technical details

- Administrative thread; no code changes
- Goal: gather volunteers and project ideas for Outreachy participation

---

### `git imap-send --draft` (2026/09/01/14-21-47)
**What changed?**
Wolfgang Faust posted a patch adding a `--draft` option to `git imap-send` to mark uploaded messages with the IMAP `\Draft` flag. Junio C Hamano raised two points: whether email clients would misbehave if they encounter messages marked with the `\Draft` flag, and a preference for failing fast rather than warning when `--draft` is used with older libcurl versions. Wolfgang agreed to fail fast.

### Why it matters

The feature addresses a long-standing usability gap: `git imap-send` is documented for uploading draft emails but lacked a way to signal draft status to IMAP clients. The patch is well-motivated and uncontroversial, with clear documentation and graceful fallback.

### Key technical details

- Files: `Documentation/git-imap-send.txt`, `git-curl-compat.h`, `imap-send.c`
- New CLI option: `--[no-]draft` (defaults to `--no-draft`)
- New symbol: compatibility macro for `CURLOPT_UPLOAD_FLAGS` and `CURLULFLAG_DRAFT`
- Behavior: marks uploaded messages with the IMAP `\Draft` flag

---

### `git rerere` lock contention (2026/09/02/08-31-37)
**What changed?**
Thomas Bachem posted a patch introducing a `rerere.lockTimeout` config and `RERERE_SKIP_LOCKED` flag to handle lock contention between background maintenance and rebase. Phillip Wood opposed the timeout-based fix, advocating for disabling background maintenance during rebase via config overrides and fixing the sequencer’s error handling. Thomas agreed to implement Phillip’s preferred approach.

### Why it matters

The issue stems from the geometric maintenance strategy introduced in Git 2.54, which triggers `git rerere gc` after every commit. This creates unavoidable lock contention with the sequencer’s own `rerere` operations. The revised approach addresses the root cause by preventing the background task from running during rebase.

### Key technical details

- Files: `rerere.c`, `sequencer.c`, `t/t4200-rerere.sh`, `t/t7900-maintenance.sh`
- New config: `rerere.lockTimeout` (milliseconds, default 1000)
- New flag: `RERERE_SKIP_LOCKED` for `setup_rerere()`
- Revised approach: disable `gc.auto` and `maintenance.auto` during rebase

---

### `parse-options` early argument scanning (2026/09/02/16-10-41)
**What changed?**
Christian Couder posted a six-patch series introducing a new `parse-options` sub-API (`early_scan_options()`) to fix argument-parsing bugs where hand-rolled early scans fail to account for option values that appear as separate arguments. Junio C Hamano raised substantive design concerns, questioning the long-term viability of the new API and the practical value of the `git bisect` and `git rev-parse` fixes.

### Why it matters

The series targets a recurring class of bugs in commands like `git bisect`, `git rev-parse`, and `git fast-import`. Junio’s feedback suggests the current design may create technical debt, and the discussion is now focused on whether to rework the API to align more closely with `parse_options()` or proceed as a separate, minimal abstraction.

### Key technical details

- Files: `parse-options.h`, `parse-options.c`, `builtin/bisect.c`, `builtin/rev-parse.c`, `fast-import.c`, test files
- New functions: `early_scan_options()`, `early_scan_options_from_options()`, `parse_options_takes_argument()`, `find_early_scan_option()`
- New types: `struct early_scan_option`, `early_scan_fn`, `enum early_scan_flags`
- New macros: `EARLY_SCAN_SKIP_VALUE`, `EARLY_SCAN_WANT`, `EARLY_SCAN_WANT_VALUE`, `EARLY_SCAN_END`

---

### `hooks.allowNoVerify` (2026/09/02/16-17-42)
**What changed?**
Junio C Hamano rejected Alessio Attilio’s `hooks.allowNoVerify` feature, citing concerns about configuration complexity creep and the narrow scope of the feature (hooks-only). brian m. carlson reinforced the rejection, arguing that `--no-verify` is a deliberate user choice and that Git should not introduce configuration layers to restrict it.

### Why it matters

The feature aimed to prevent accidental bypass of hooks via `--no-verify` in managed environments or CI workflows. The rejection highlights a philosophical divide: whether Git should provide guardrails for workflow discipline or defer to user education. The discussion is effectively closed unless a fundamentally different approach is presented.

### Key technical details

- New config key: `hooks.allowNoVerify` (string, accepts `true`/`warn`/`false`; defaults to `true`)
- Commands affected: `commit`, `push`, `merge`, `rebase`, `am`
- Emergency overrides: `GIT_ALLOW_NO_VERIFY=1` environment variable and `-c hooks.allowNoVerify=true` CLI flag

---

## In brief
- **`--missing-only` for `git rev-list`**: Final sign-off received; queued for `next`. (2026/09/02/21-26-03)
- **GSoC 2026 – Improving disk space recovery for partial clones**: Final submission report posted; project is feature-complete for GSoC scope. (2026/09/02/04-35-59)
- **`git checkout -m` autostash conflict handling**: v4 posted and accepted; ready for merging. (2026/09/02/18-29-01, 2026/09/02/19-52-30)
- **`USE_NSEC` runtime config**: Follow-up discussion on `#ifndef NO_NSEC` guard; maintainer requested a final reroll. (2026/09/02/07-26-46, 2026/09/02/21-05-11)
- **Silent corruption in `git diff` on Windows**: D. Ben Knoble clarified that the `text` attribute (not the `diff` filter) triggers line-ending normalization, corrupting binaries. (2026/09/02/12-14-19)
- **`receive-report` hook**: Karthik Nayak strengthened the commit message’s rationale for decoupling reported status from actual repository state. (2026/09/02/14-42-19)
- **`gitdatamodel(7)` man page**: Final stylistic tweaks agreed upon; series merged to `next`. (2026/09/02/17-08-01)
- **`git push` from shallow clones**: Elijah Newren refuted Stolee’s corruption concern, clarifying that `receive-pack` enforces connectivity checks. (2026/09/02/20-57-00)
- **`git commit` vs. `git merge` newline handling**: Simon Cheng reported an inconsistency in newline handling between the two commands. (2026/09/02/14-04-37)
- **ODB ad-hoc source linking**: Patrick Steinhardt posted v2 of a 13-patch series refactoring the ODB subsystem to remove ad-hoc source linking. (2026/09/02/13-34-49)
- **Rust compilation support in GitHub Actions**: Junio C Hamano flagged a discrepancy in the Rust target triple mapping for UCRT64. (2026/09/02/06-25-48)
- **Submodule memory leak**: Jeff King posted a two-patch bugfix series to plug a memory leak in the submodule error path. (2026/09/02/05-51-17)
- **Pathspec buffer overflow**: Yannik Tausch posted a bugfix patch addressing a buffer overflow in `match_pathspec_with_flags()` when handling negative pathspecs. (2026/09/02/12-22-39)
- **Common prefix calculation**: Yannik Tausch posted a bugfix patch fixing an edge case in `common_prefix_len()` where an exclude pathspec appearing first in the list incorrectly returns zero as the common prefix length. (2026/09/02/13-04-09)
- **`grafts_keep_true_parents`**: Yuvraj Singh Chauhan posted a two-patch series moving the global `grafts_keep_true_parents` configuration variable into the repository-specific `repo_config_values` struct. (2026/09/02/11-30-45)
- **`git history` NULL tree guard**: zkd18cjb@mail.ustc.edu.cn posted a bugfix patch preventing a NULL-pointer dereference crash in `write_ondisk_index()` when the parent commit’s tree object is missing. (2026/09/02/12-07-36)
- **Git Rev News edition 138**: Christian Couder announced the publication of Git Rev News edition 138. (2026/09/02/19-27-05)