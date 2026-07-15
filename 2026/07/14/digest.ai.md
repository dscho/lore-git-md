The day in brief
2026/07/14 was a **heavy but focused** day on the Git mailing list, with **135 emails across 25 threads**. The standout developments were **two major series landing in `next`**—Pablo Sabater’s **`remote-object-info`** security-hardened feature for `git cat-file --batch-command` and Harald Nordgren’s **`--delete-merged`** branch-cleanup command—while **design discussions** on **packfile URI races**, **ODB abstraction**, and **`git history squash`** integration with `git rebase` hinted at future architectural shifts. A **new contributor’s test modernization** also graduated to `next`, and **Coverity-driven hardening patches** from Johannes Schindelin addressed latent bugs across HTTP, reftable, and bisect subsystems.

---

## Notable threads

### `remote-object-info` lands in `next` (Pablo Sabater)
Pablo Sabater’s **17-iteration, security-hardened series** adding `remote-object-info` support to `git cat-file --batch-command` is now **technically complete and merged to `next`**. The feature lets users query object metadata (currently just size) from remote repositories without downloading full objects, with strict input validation, dynamic capability-based placeholder filtering, and **699 lines of new tests**. Junio reverted the v16 merge due to a **cover-letter attribution mismatch** (incorrectly crediting Eric Ju and Calvin Wan), but the v17 resend fixed this and addressed the last **bounds-checking concern** in the response parser. The series is a **major step forward for remote object inspection**, with only a **follow-up on callback-type consistency** deferred.

Key technical details:
- **Dynamic capability-based validation**: Runtime filtering of format placeholders against server-advertised capabilities (e.g., `object-info=size`), directly addressing segfault and `die()` issues.
- **Memory safety**: Fixed two memory leaks in transport helpers and eliminated a static variable in `fetch-pack.c` that risked undefined behavior.
- **Test coverage**: **699 lines** of new tests in `t/t1017-cat-file-remote-object-info.sh`, covering all transport types (`git://`, `file://`, `http://`) and edge cases.
- **Security**: Strict protocol v2 enforcement, input size limits (10,000 objects max, 8KB URL length), and bounds-checking in response parsing.

---

### `--delete-merged` branch cleanup graduates (Harald Nordgren)
Harald Nordgren’s **19-iteration series** introducing `git branch --delete-merged` is now **implementation-complete and merged to `next`**. The command safely deletes local branches merged into their upstream while preserving branches used as upstreams for unmerged branches via **refined stacked-branch protection (abort-and-clear)**. The series includes **per-branch opt-out** (`branch.<name>.deleteMerged=false`), **`--dry-run` preview**, and **repeatable `--delete-merged` arguments** to widen the upstream match. Junio requested that the **stacked-branch protection be extended to `git branch -d` for consistency**, which Harald agreed to implement as a follow-up.

Key technical details:
- **Stacked-branch protection**: Fails to delete a branch if it is used as an upstream for an unmerged branch, but **clears stale upstream configs** for kept (merged) branches whose own upstream is being deleted.
- **Per-branch opt-out**: `branch.<name>.deleteMerged=false` exempts specific branches from automated cleanup while allowing explicit deletion via `git branch -d`.
- **`--dry-run`**: Prints "Would delete branch <name>" without modifying refs or config, with explicit checks to avoid side effects.
- **Test coverage**: Comprehensive integration tests covering edge cases like self-referential upstreams, missing upstreams, and CLI/config interactions.

---

### `git history squash` and the future of `git rebase` (Harald Nordgren, D. Ben Knoble, Junio C Hamano)
A **forward-looking design discussion** emerged around integrating `git history squash` with `git rebase --autosquash`. D. Ben Knoble speculated that combining `git history squash` with `git replay` could emulate `git rebase --autosquash` **non-interactively and faster** by avoiding repeated conflict resolution. Junio C Hamano **fully endorsed this vision**, calling it "an ideal future" and confirming that the ongoing `git history` work is already moving the project in this direction. The discussion frames `git history squash` as a **foundational piece of a larger refactoring effort** to modernize Git’s history-editing infrastructure, with the eventual goal of replacing the internals of `git rebase -i`. No immediate code changes are proposed, but the architectural shift is now a **project-level aspiration**.

Key technical details:
- **Performance**: Avoiding repeated conflict resolution in `git rebase` by using `git replay` for non-interactive operations.
- **Feature compatibility**: Ensuring `git history squash` handles all scenarios currently supported by `git rebase --autosquash`, including conflicts and out-of-range fixup targets.
- **Long-term goal**: Replacing the internals of `git rebase -i` with a modular, conflict-free pipeline.

---

### Packfile URI races: design trade-offs (Ted Nyman, Junio C Hamano, Jeff King)
Ted Nyman’s **bugfix series for packfile URI download races** sparked a **substantive design discussion** about resumable downloads and coordination mechanisms. The initial design used **unique tempfiles** to prevent corruption but abandoned resumable downloads for packfile URIs, which Junio and Jeff King argued was a **regression**. Ted now plans to revise the design to **preserve predictable partial-pack paths** while preventing concurrent corruption, addressing both **handoff scenarios** (one process taking over a stalled download) and **stale-file cleanup** (leftover `.pack.temp` files from crashed processes). The discussion also acknowledged a **similar concurrency vulnerability in dumb HTTP**, suggesting a **unified solution** may be needed for both features.

Key technical details:
- **Resumable downloads**: The current design sacrifices resumption for packfile URIs, which Junio and Peff argue is unacceptable.
- **Coordination mechanisms**: Proposals include lockfiles, Peff’s write-only file descriptor approach, or piping to `index-pack --stdin`.
- **Dumb HTTP**: Shares the same concurrency vulnerability, suggesting a unified solution is needed.

---

### ODB abstraction: object filters and callback consistency (Patrick Steinhardt, Taylor Blau, Jeff King)
Patrick Steinhardt’s **ODB abstraction series** (introducing object filters to `odb_for_each_object()`) is **ready for integration**, with all prior feedback addressed. The series enables `git cat-file --batch-all-objects` to work with pluggable backends while eliminating redundant index traversals. A **follow-up discussion** emerged about **callback-type consistency** between `traverse_bitmap_commit_list()` and `traverse_commit_list()`, with Taylor Blau proposing to **split the callback type** used by `for_each_bitmapped_object()` from `show_reachable_fn` to preserve abortable behavior for bitmapped object iteration while aligning the non-bitmap traversal path with `void`-returning callbacks. Jeff King endorsed the idea as a "nice cleanup" but cautioned that its safety can only be confirmed by implementation. The follow-up is **planned as a two-patch series on top of v3**.

Key technical details:
- **Object filters**: Enables `git cat-file --batch-all-objects` to work with pluggable backends.
- **Callback-type split**: Preserves abortable behavior for bitmapped object iteration while aligning non-bitmap traversal with `void`-returning callbacks.
- **Performance**: Eliminates redundant index traversals in `odb_for_each_object()`.

---

## In brief

### Test modernization graduates (Shlok Kulshreshtha)
Shlok Kulshreshtha’s **test modernization series** for `t/t1100-commit-tree-options.sh` is now **queued in `next`**. The patches convert legacy `test_expect_success` blocks to the modern quoted-body style and relocate a here-doc into the setup test, following the pattern established in `t7001`. The changes are purely stylistic, with no behavior or coverage changes. Junio and Patrick Steinhardt reviewed the series, focusing on **surface-level improvements** (commit message tense, formatting), and confirmed it is ready for integration.

### Coverity-driven hardening (Johannes Schindelin)
Johannes Schindelin posted an **11-patch series** addressing unchecked return values flagged by Coverity. The patches add error checks to functions that can fail (e.g., `curl_easy_duphandle`, `deflateInit`, `repo_parse_commit`, `lseek`, `dup`, `strbuf_getline_lf`, `launch_editor`, `reftable_table_init_ref_iterator`, and `rename`) across HTTP, config, reftable, last-modified, compat/pread, transport-helper, and bisect subsystems. The fixes are **mechanical and uncontroversial**, hardening existing code paths against silent failures without introducing new behavior.

Key patches:
- **HTTP transport**: `get_active_slot()` now checks `curl_easy_duphandle()` for NULL to prevent undefined behavior in `curl_easy_setopt()`.
- **Reftable backend**: `block_writer_init()` now checks `deflateInit()` return value to prevent silent data corruption.
- **Bisect subsystem**: `get_terms()` and `read_bisect_terms()` now check `strbuf_getline_lf()` return value to handle truncated `BISECT_TERMS` files.

### `git show-branch` slab conversion (Gatla Vishweshwar Reddy, Junio C Hamano, Jeff King)
Gatla Vishweshwar Reddy’s **slab conversion for `git show-branch`** (replacing `object.flags` with a commit-slab) received **substantive feedback** from Junio and Jeff King. The current design uses a fixed `uint64_t[1]` slab element, which Junio argued does not fully exploit the slab’s flexibility. He suggested using a **variable-length bitset** (prior art exists) to handle an arbitrary number of branches and questioned the placement of the `UNINTERESTING` bit in the slab. Jeff King endorsed Junio’s critique and pointed to a **2025 series** introducing a variable-length bitset slab as a model for a v3. The patch is **not yet ready for integration** and will require a redesign.

### `git last-modified` performance inquiry (Gusted)
Gusted reported that `git last-modified -z -t --max-depth=0` is **2.5× slower** (66 ms vs 26 ms) than an equivalent `git log` pipeline when run against the ziglang/zig repository. The inquiry is purely diagnostic, with no patch or design discussion yet. The thread is awaiting further analysis to determine whether the performance gap is expected or whether the command can be optimized.

---

## On the radar

### `no-ref-delta` capability (Taylor Blau, Jeff King)
Taylor Blau’s **`no-ref-delta` capability** series (enabling receivers to request packs containing only `OFS_DELTA` entries) remains **under review** after Jeff King raised **fundamental concerns** about the motivation and design. The use case is now tied to a **write-through proxy**, but Peff questioned whether the feature’s abandonment of resumable downloads is justified. The discussion is ongoing, with no clear resolution yet.

### `fetch.submoduleErrors` (Paulius Zaleckas, Junio C Hamano)
Paulius Zaleckas’s **`fetch.submoduleErrors`** series (configurable submodule fetch error handling) is **under review**, with Junio requesting **mechanical refactoring** of config value parsing and option forwarding logic. The series is **not yet ready for integration**, but the design (allowing users to choose between `fail` and `warn` modes) is uncontroversial.

### Fake-linear mainline proposal (Richard Fine)
Richard Fine’s **RFC for persistent shallow fetch options and a "fake-linear mainline" mechanism** sparked interest but remains **speculative**. The proposal targets large repositories with long-running branches and merge-heavy histories, but reviewers may suggest **existing tools** (e.g., `--filter=tree:0`, `git replace`) before endorsing new features. The discussion is ongoing, with no clear path forward yet.