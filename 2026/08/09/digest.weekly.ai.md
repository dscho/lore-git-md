# The Git Project Weekly Digest
**2026/08/03 -- 2026/08/09**

## The period in brief
This was a **high-volume, eventful week** (471 emails across 7 days) marked by **major feature integrations**, **contentious design debates**, and **critical regressions**. The standout developments: **`git branch --delete-merged` landed in `next`** after 25 iterations, **`git history squash` was ejected from `next`** due to fundamental flaws, and **ODB abstraction work advanced** with two series merged. The week also saw **procedural debates** about review quality and AI-assisted contributions, **security concerns** in `gitk` and `git push --porcelain`, and **performance optimizations** for merge-base queries and I/O. Readers who skipped the daily digests should not miss: **the `git history squash` ejection**, **the ODB abstraction milestones**, and **the `paint_down_to_common()` 100–1000× speedup**.

---

## Key developments

### `git branch --delete-merged` lands in `next`
Harald Nordgren’s **25-iteration series** adding `git branch --delete-merged`—a safe, automated branch cleanup command—**graduated to `next`** after resolving all technical and procedural blockers. The feature provides **order-independent stacked-branch protection**, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. The final design simplifies stacked-branch protection to clear upstream configs only for kept branches, avoiding the complexity of protecting entire hierarchies. Junio C Hamano and Phillip Wood confirmed the series is **implementation-complete**, with only a follow-up planned to extend the protection to `git branch -d`. The series is a **major usability improvement** for topic-branch workflows, offering a predictable alternative to manual branch cleanup.

**Key participants**: Harald Nordgren, Junio C Hamano, Phillip Wood.
**Status**: **Merged to `next`**; expected to graduate to `master` in the current cycle.

---

### `git history squash` ejected from `next` after fundamental flaws exposed
Phillip Wood’s review of Harald Nordgren’s `git history squash`—a command to collapse commit ranges into a single commit—**exposed critical correctness flaws** in reachability logic, UX inconsistencies, and ref safety, leading to its **ejection from `next`**. The issues include:
- **Incorrect handling of `UNINTERESTING` commits** (e.g., `git history squash ^A B` where `A` is `UNINTERESTING`).
- **Failure to handle root commits** (commits with no parents).
- **Multi-tip histories** (e.g., `git history squash ^A C D` in `-A-B-C \ D`).
- **Inconsistent ref safety** (refuses to squash commits pointed to by tags or remote-tracking refs, unlike other `git history` commands).

The ejection sparked a **broader discussion about review quality and integration pipeline policy**, with Junio proposing a **temporary moratorium on accepting new topics into `seen`** unless they receive substantive review. The series remains **stalled in `seen`** pending Phillip’s fixups, but the procedural debate may resurface in future "What's cooking" reports.

**Key participants**: Harald Nordgren, Phillip Wood, Junio C Hamano, Matt Hunter.
**Status**: **Ejected from `next`**; fixups in progress.

---

### ODB abstraction work advances with two series merged
The **pluggable ODB effort** saw two major milestones:
1. **On-disk structure pluggability**: Patrick Steinhardt’s **six-patch series** decoupling on-disk ODB structure creation from repository setup **graduated to `master`**. The series delegates backend-specific directory creation (e.g., `objects/pack/`, `objects/info/`) to the ODB backend via a `create_on_disk` callback, fixing a latent bug with alternates and loose-object maps. This is the **first major ODB abstraction milestone** since the reftable backend.
2. **Packfile writes in `receive-pack`**: Justin Tobler’s **seven-patch series** extending the ODB transaction system to support pluggable packfile writes in `git receive-pack` **advanced to v2**. The series introduces `odb_transaction_write_pack()` and refactors `receive-pack` to eliminate global state, enabling backend-agnostic packfile processing. The v2 addresses all feedback from v1, including ergonomic improvements and backend-specific optimizations.

**Key participants**: Patrick Steinhardt, Justin Tobler, Junio C Hamano, Peff.
**Status**:
- **On-disk pluggability**: **Merged to `master`**.
- **Packfile writes**: **Under active review (v2)**.

---

### `paint_down_to_common()` optimization queued
Kristofer Karlsson and Tian Yuchen’s **nine-patch series** optimizing `paint_down_to_common()`—terminating early when one side of a merge-base query exhausts its commit queue—**achieved 100–1000× speedups** for asymmetric queries (e.g., repositories with import grafts or shallow histories). The series introduces **Trace2 instrumentation** to report step counts, deterministic test assertions for all commit-graph modes, and a **topological ceiling** for v1 commit-graphs. Junio C Hamano confirmed the series is **procedurally unblocked** and queued in his tree, with review bandwidth as the remaining bottleneck. The optimization is particularly impactful for large repositories like the Linux kernel.

**Key participants**: Kristofer Karlsson, Tian Yuchen, Elijah Newren, Junio C Hamano.
**Status**: **Queued in Junio’s tree**; awaiting additional reviewers.

---

### `git repack --drop-filtered` nears merge
Siddharth Shrimali’s **seven-patch series** adding `--drop-filtered` to `git repack`—safely reclaiming disk space in partial clones by removing locally cached promisor blobs exceeding a size threshold—**addressed all prior feedback** and is ready for integration. The series includes **safety guards** (merge/rebase/cherry-pick checks, index validation) and **protocol robustness** (client `die()`s on under-length or over-length server responses). Junio’s latest review identified **three mechanical issues** (SYNOPSIS mismatch, missing newline, test script style), which are trivial to fix. The feature is a **performance-critical improvement** for partial clones in large-scale hosting environments.

**Key participants**: Siddharth Shrimali, Junio C Hamano, Siddharth Asthana.
**Status**: **Feature-complete (v4)**; needs mechanical fixes before merging.

---

### `git cat-file --batch-command` `%(objecttype)` support ready for integration
Pablo Sabater’s GSoC series extending `git cat-file --batch-command` to support the `%(objecttype)` placeholder in remote-object-info queries **reached v6** and is ready for integration. The series adds end-to-end support for object type metadata, the last piece guaranteed to match between client and server post-fetch. Key architectural changes include replacing `struct object_info` with a dedicated `struct fetch_object_info_results` and unifying the default format string to `%(objectname) %(objecttype) %(objectsize)`. Junio and Peff endorsed all patches as ready to merge, with Peff’s refactoring (removing object-info fields from `git_transport_options`) called a "sanity improvement." The series is a **foundational step** for tools like `git-annex` and `git-lfs`.

**Key participants**: Pablo Sabater, Junio C Hamano, Peff.
**Status**: **Ready for integration (v6)**.

---

### Security concerns in `gitk` and `git push --porcelain`
Two security issues surfaced this week:
1. **`gitk` custom commands**: Junio C Hamano identified a **command injection vulnerability** in Tim Wiederhake’s patch adding user-configurable commands to `gitk`’s context menus. Placeholders (e.g., `%t` for commit title) are interpolated into shell commands without proper quoting, enabling arbitrary command execution via malicious commit titles. The patch remains **stalled pending a fix**.
2. **`git push --porcelain`**: Xavier Morel reported that `git push --delete --porcelain` outputs human-readable errors (e.g., "remote ref does not exist") instead of the documented machine-readable format. Junio proposed a fix using `!` as an error prefix, and Xavier outlined an implementation strategy. The issue is a **script-breaking inconsistency** in the porcelain protocol.

**Key participants**: Junio C Hamano, Tim Wiederhake, Xavier Morel.
**Status**:
- **`gitk`**: **Stalled pending fix**.
- **`git push --porcelain`**: **Fix proposed**; implementation in progress.

---

## In brief

**`git bisect --reset-when-found`** -- Harald Nordgren’s series adding `--reset-when-found[=<where>]` to `git bisect` (automatically resetting the working tree after identifying the first bad commit) **graduated to `next`**. The feature targets automated workflows and interactive debugging.

**`git add --resolved`** -- Junio’s series adding `--resolved` to `git add` (staging only paths whose conflict markers have been removed) **addressed all review feedback** and is ready for integration. The series consolidates duplicate conflict-marker detection logic and adds helpers for index removal with flags.

**`git fast-import` libification** -- Christian Couder’s **12-patch series** refactoring `git fast-import` to use the parse-options API and reduce global variables **received Junio’s approval** and is queued in `seen`. The series removes support for five undocumented command-line options and shrinks the codebase by 38 lines.

**`git history` and `git replay` silently drop notes** -- D. Ben Knoble reported that `git replay --onto=...` and `git history fixup` silently drop Git notes during history rewriting. Phillip Wood proposed a **pre-exec copy approach** to preserve notes, but no patch has been posted yet.

**`git rebase -x` with `git commit --amend` loses notes** -- D. Ben Knoble reported that `git rebase -x` combined with `git commit --amend` fails to rewrite notes. Phillip Wood diagnosed the root cause (OID-mapping issue) and proposed a **pre-exec copy fix**, but no patch has been posted yet.

**`writev()` wrapper revival** -- Patrick Steinhardt’s **six-patch series** reintroducing a `writev()`-like wrapper for performance **achieved design convergence**. The wrapper writes only the first `iovec` in the loop, addressing forward-compatibility concerns. The series delivers **1.37× latency improvement** in `git-upload-pack` and **4% wall-clock reduction** in `git fast-import`.

**`git last-modified` Bloom filter optimization** -- Toon Claes’s **four-patch series** optimizing `git last-modified` with Bloom filters **achieved a 3.7× speedup** by reusing filters. The series is stalled on a maintainability concern about `bloom_filter_settings` management.

**`git maintenance` geometric repack with promisor packs** -- Taylor Blau’s **two-patch bugfix series** for `git maintenance`’s geometric repack task with promisor packs **ensures the task properly evaluates both ordinary and promisor pack progressions**. The series is ready for review.

**`git diff` bash completion** -- Junio’s **three-patch series** fixing bash completion for `git diff` **aligns path completion with `git status` and `git add`**, especially when invoked with `git -C <directory>`. The series is ready for integration.

**`git config` editor failure** -- Johannes Schindelin’s **11-patch series** fixing unchecked return values (e.g., `curl_easy_duphandle`, `deflateInit`, `repo_parse_commit`) **adds missing error checks** to harden Git against silent failures. The series is complete and ready for final review.

**`git fsmonitor` macOS cookie flush** -- Tamir Duberstein’s patch fixing a macOS-specific performance regression in Git’s fsmonitor daemon **addresses a cookie wait mechanism issue**. The patch is narrowly scoped and under review.

**`git regex` memory leak on macOS** -- Chungmin Lee’s patch fixing a memory leak in Git’s macOS builds when the system regex engine (TRE) processes invalid UTF-8 sequences **raises maintainability concerns** about the current workaround. The discussion remains open.

**`fetch_if_missing` in `struct repository`** -- Tian Yuchen’s patch moving `fetch_if_missing` into `struct repository` **remains blocked on design questions** about repository-less invocations and a null pointer dereference risk in `builtin/index-pack.c`.

**`git whatchanged` deprecation** -- Junio’s procedural nudge reminded users to find replacements for `git whatchanged` before reporting continued use.

**`git rebase` commit-graph regression** -- Philippe Blain reported a fatal error when submodule pointers change, bisected to `bb5da75d61`. No fix has been posted yet.

**`git fetch` feature request** -- Douglas Puchalski proposed decoupling default fetch refspecs from explicit branch mappings. Junio suggested extending `--refmap` to support a `matching`-like mode.

**Interactive patch selection** -- Junio’s proposal to unify `git add -p`/`git checkout -p` workflows sparked discussion about **triage actions** (stage/discard/defer). Christian Couder suggested leveraging `git history` for advanced use cases.

**`USE_NSEC` runtime config** -- D. Ben Knoble’s series converting the build-time `USE_NSEC` macro to a runtime option (`core.useNanosec`) **identified a performance regression** in the hot path. Junio proposed caching the value in `repo_settings` for constant-time access.

**Sparse index use-after-free** -- Shlok Kulshreshtha’s patch fixing a use-after-free in `get_oid_with_context_1()` when resolving relative paths against a sparse index **added test coverage** and is ready for final review.

**Coccinelle performance regression** -- SZEDER Gábor’s benchmarks showed Coccinelle 1.3.1 is **4.5× slower** than 1.1.1, challenging the rationale for bumping the Ubuntu image for the static-analysis job.

**`Signed-off-by:` critique** -- Дилян Палаузов questioned the legal and practical validity of the `Signed-off-by:` trailer, arguing it is redundant and creates an uneven playing field. No responses yet.

**`git rebase` vs. `git merge` `-X` inconsistency** -- zrzut01@poczta.fm reported that `git rebase -X <invalid-option>` silently ignores the invalid option, while `git merge -X <invalid-option>` errors out. No maintainer response yet.

---

## Looking ahead
The next week is likely to see:
- **Rerolls for stalled topics**: `git history squash` (correctness fixes), `git last-modified` Bloom filter optimization (maintainability concerns), `fetch_if_missing` libification (architectural questions).
- **New patches for confirmed bugs**: `git push --porcelain` inconsistency, `git rebase -x` notes loss, `gitk` command injection vulnerability.
- **Integration of ready series**: `git repack --drop-filtered`, `git cat-file --batch-command` `%(objecttype)`, `git add --resolved`, `git bisect --reset-when-found`.
- **ODB abstraction follow-ups**: Justin Tobler’s packfile writes series (v2), Patrick Steinhardt’s ongoing work.
- **Performance discussions**: `writev()` wrapper revival, Coccinelle performance regression.

The **Git 3.0 preparation** (`bc/restrict-hex-to-lowercase`) and **pluggable ODB effort** remain key long-term themes.