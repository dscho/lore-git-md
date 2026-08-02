Here’s the digest for **2026/08/01**, covering the day’s most significant developments in Git’s mailing list activity:

---

### The day in brief
**Saturday, August 1, 2026** was a **high-volume, milestone-heavy day** for Git development, with **12 active threads** spanning **51 emails**. The standout events:
1. **Michael Montalbo’s `diff-provider` RFC v7** landed as a **unified 11-patch series**, proposing a framework for pluggable diff backends (e.g., in-process caches, external processes) while preserving Git’s full diff/blame feature set. The series is **technically complete** and seeks design feedback before splitting into mergeable topics.
2. **Harald Nordgren’s `git history squash` v11** was posted, **resolving all prior feedback** (including the usability debate over default message editing) and marked **"Will replace"** by Junio. The series is **ready for `next`** and positions `squash` as a foundational piece of Git’s future rebase infrastructure.
3. **Ted Nyman’s packfile URI race-condition fix (v6)** received **final approval from Peff**, clearing the last technical hurdle for integration. The series is now **ready for `next`**.
4. **Junio’s `git add --resolved` v3** saw **usability debate** with Peff, but Junio cited existing merge constraints to mitigate risks. The series remains **on track for `next`** pending minor documentation tweaks.

---

### Notable threads

#### **Pluggable diff backends: `diff-provider` RFC v7**
**Headline**: Michael Montalbo’s **11-patch RFC** introduces a **unified abstraction layer** (`diff-provider.h`) for consulting alternate diff hunk sources (e.g., in-process caches, external processes) before falling back to Git’s builtin xdiff. The series ships two example providers:
- **`diff-hunks` store**: A non-authoritative in-process cache at `$GIT_DIR/objects/info/diff-hunks`, keyed by `(old_blob, new_blob, xdl_opts)`, with atomic flush and corruption handling. A warmed store speeds up `log --stat -5000` by **~1.9x** and `blame` by **~1.26x** on `git.git`.
- **`diff.<driver>.process`**: An authoritative external process provider (pkt-line protocol) that answers by object identity alone, enabling pre-fetching or custom diff logic. The process is opt-in per command (like `textconv`) and degrades gracefully to the builtin diff on errors.

**Key details**:
- **Design**: Repository-owned provider chain, built on first consultation, released in `repo_clear()`. Consumers use a unified `diff_request` struct; providers validate hunks for order, overlap, and alignment.
- **Performance**: Default path (no providers) adds **no overhead**; warmed cache delivers **1.26–1.9x speedups**.
- **Scope**: Identity-only requests for now; content-carrying requests (for patch output) deferred to future work.
- **Feedback**: The RFC explicitly seeks **design feedback** on the framework, not a merge of the 11 patches. If accepted, the work will return as separate series for the interface/store, process provider, and content enrichment.

**Status**: **Technically complete**; awaiting design feedback. Junio’s "What’s cooking" may flag this as a **major architectural direction** for Git 3.0.

---

#### **`git history squash` v11: Ready for `next`**
**Headline**: Harald Nordgren’s **4-patch series** adding `git history squash` (fold a commit range into its oldest commit) is **functionally complete** and **resolves all prior feedback**, including the usability debate over default message editing. Junio’s **"Will replace"** sign-off on v7 signals intent to queue for the next release.

**Key changes in v11**:
- **Default message editing**: The command now opens an editor with an autosquash-style template by default (matching `git rebase -i --autosquash`), with `--no-edit` as the opt-out flag. This addresses Phillip Wood’s argument for encouraging commit hygiene.
- **Stricter range validation**: Rejects ranges reaching a root commit or with multiple tips.
- **Ref-handling refinement**: Only local branches pointing inside the range are protected by default; tags/remote refs are left unchanged.

**Design notes**:
- **Performance**: Avoids the repeated conflict stops of `git rebase --autosquash` by using `git replay` under the hood (a **project-level aspiration** endorsed by Junio and D. Ben Knoble).
- **Safety**: Rejects operations with local branches pointing inside the range by default, with advice guiding users to `--update-refs=head`.

**Status**: **Ready for `next`**. Junio’s prior sign-off and Matt Hunter’s explicit v10 sign-off ("v10 looks good to me!") indicate the series is on track for the next release.

---

#### **Packfile URI race-condition fix: Final approval**
**Headline**: Ted Nyman’s **6-patch v6 series** fixing race conditions in Git’s HTTP transport (packfile URI and dumb HTTP) received **final approval from Peff**, clearing the last technical hurdle. The series is now **ready for `next`**.

**Core fix**:
- **Deterministic staging paths**: Uses read-write mode (no append) to ensure concurrent processes write identical bytes at identical offsets.
- **Windows/MINGW compatibility**: Retries open without `O_CREAT` after exclusive creation attempts to handle sharing semantics.
- **HTTP 416 handling**: Gracefully handles completed partial packs, with Peff’s forward-compatibility note about remote file changes mid-transfer (not a regression).

**Key files**: `http.c`, `fetch-pack.c`, `t/t5550-http-fetch-dumb.sh`, `t/t5702-protocol-v2.sh`.
**Status**: **Ready for `next`**. Junio’s intent to graduate the series was confirmed in his call-for-approval email.

---

#### **`git add --resolved`: Usability debate**
**Headline**: Junio’s **4-patch v3 series** adding `git add --resolved` (stage only paths with resolved conflict markers) saw **usability debate** with Peff, who raised concerns about the option creating a **false sense of completeness** by staging only conflict-resolved files while ignoring related changes (e.g., updates to callers of a modified function).

**Junio’s response**:
- Git’s merge machinery (including "ort") **refuses to operate on paths with local modifications**, mitigating the risk. Cited test case `t6423` ("Testcase 11b") as validation.
- The constraint is enforced by `read-tree -m -u` and newer merge code paths (e.g., directory rename handling).

**Status**: **On track for `next`** pending minor documentation tweaks to clarify the feature’s limitations. The debate highlights a subtle workflow pitfall worth addressing in `git-add.txt`.

---

### In brief
- **`git bisect --reset-when-found` v4**: Harald Nordgren’s **2-patch series** adding auto-reset after bisect is **ready for `next`**, with Junio proposing a **refactoring** to simplify the internal state machine (non-blocking).
- **`git repack --drop-filtered` RFC v2**: Siddharth Shrimali’s **7-patch series** for partial clones now **concedes safety guards as optional optimizations** (not correctness measures) after Junio’s feedback. The core functionality (reclaiming promisor blobs exceeding a size threshold) is complete, but the drop log and remote verification remain deferred.
- **`git cat-file --batch-command` `%(objecttype)`**: Pablo Sabater’s **GSoC v2 series** saw **architectural feedback** from Peff, who recommended bypassing `struct object_info` entirely for remote queries. The series is otherwise complete and awaiting Pablo’s response.
- **Hex object IDs in Git 3.0**: brian m. carlson’s **RFC series** to enforce lowercase-only parsing remains **contested**, with Junio invoking the **Robustness Principle** ("be liberal in what you accept") and Peff identifying an edge case where uppercase hex in object payloads creates parallel histories. No maintainer decision yet.
- **`git filter-branch` bugfix**: Grant Moyer’s **standalone patch** fixes a long-standing inversion in commit mapping logic with `--state-branch`. The fix is **self-contained** and includes a test.

---

### On the radar
- **`fetch_if_missing` per-repository**: Tian Yuchen’s **v2 patch** moving the global variable into `struct repository` needs a **v3** to address Junio’s edge case (repository-less invocations). The series aligns with the `the_repository` removal effort.
- **Trailing blank lines**: Thomas Nemeth’s **feature request** for a config option to suppress warnings is **resolved for his use case** via `core.whitespace=-blank-at-eof`, but the stricter interpretation (enforcing exactly one trailing line) remains unaddressed. No active demand for implementation.