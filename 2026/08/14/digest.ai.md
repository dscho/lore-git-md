# Git mailing list daily digest for 2026/08/14

## The day in brief
The Git mailing list saw **substantive design discussions** on ODB abstraction, source tree reorganization, and filesystem-level optimizations, alongside **performance regression fixes**, **new features**, and **documentation updates**. Key developments include a **data-driven counterproposal** for Git’s source tree reorganization, **ODB transaction refinements**, and **proposals for filesystem-level copy-on-write** in `git worktree add`. Several patches advanced toward integration, including a **MIDX incremental-write fix**, a **packfile performance regression fix**, and **bash completion improvements**.

---

## Notable threads

### Source tree reorganization: data-driven counterproposal
**What changed?**
Michael Montalbo introduced a **dual-signal analysis** combining commit history patterns (weighted by commit size) and call-graph relationships (derived from `cscope`) to identify cohesive subsystems for incremental reorganization. The script groups 163 of Git’s 231 `libgit.a` source files into 14 directories (e.g., `transport/`, `index/`, `revision/`) with cohesion scores higher than existing directories like `refs/`. Boundary cases (e.g., `shallow.c` grouped with `transport/` despite its `revision:` commit label) are flagged as review targets.

**Why it matters**
This proposal advances the **deadlocked debate** over Patrick Steinhardt’s RFC patch series to reorganize Git’s source tree by moving all `libgit.a` components into a new `lib/` directory. The current `lib/` approach is criticized as an arbitrary intermediate layer that fails to improve discoverability. Montalbo’s **mechanical, reproducible methodology** avoids this arbitrariness while enabling incremental progress. The proposal sidesteps the **newcomer discoverability** framing, instead appealing to **project consistency**: the same signals that flagged `odb/` before it was carved out now identify other cohesive units.

**Technical details**
- **Methodology**: Dual-signal analysis combining (1) **commit history** (weighted modal "area:" labels, e.g., `diff:` or `refs:`) and (2) **call-graph relationships** (using `cscope` to identify function calls between files).
- **Proposed directories**: 14 subsystems (`odb/`, `refs/`, `pack/`, `diff/`, `merge/`, `revision/`, `index/`, `setup/`, `convert/`, `transport/`, `notes/`, `submodule/`, `archive/`, `sequencer/`).
- **Ungrouped files**: 68 files (e.g., `strbuf.c`, `hashmap.c`, `alloc.c`) lack strong historical or call-graph signals, suggesting they may belong in a shared utility layer.
- **Script**: A 200-line Python script processes Git’s history, `Makefile`, and `cscope` output. The `CHARTER` block (14 lines) maps area names to directories and is the only manual input.

**Open questions**
- Whether the community will engage with the methodology’s **technical rigor** and **actionable output** (e.g., boundary cases).
- How this proposal interacts with Junio C Hamano’s **subsystem-by-subsystem alternative** and Patrick Steinhardt’s **`lib/` directory approach**.

---

### ODB abstraction: transaction refinements and design gaps
**What changed?**
The ODB abstraction effort saw **substantive design discussions** and **interface refinements** in two threads:
1. **`odb_transaction_write_pack()`**: Patrick Steinhardt identified a **design gap** in the new interface: the `get_unpack_limit()` helper assumes a `git receive-pack` context but the interface is intended to be generic. Justin Tobler proposed using the existing `ODB_TRANSACTION_RECEIVE` flag (and future `ODB_TRANSACTION_FETCH`) to enable context-specific configuration resolution (e.g., `fetch.unpackLimit` for `git fetch-pack`).
2. **Packfile performance regression**: Jeff King (Peff) clarified that Git already maintains a hashmap for tracking packfiles (introduced in `ec48540fe8`), which prevents duplicates during "reprepare" operations. This reframes the fast-path optimization in Johannes Schindelin’s patch as a targeted optimization to avoid the O(N) scan *even when uniqueness is already guaranteed*. Peff also **cast doubt** on Patrick Steinhardt’s proposed hashmap refactoring, noting the hashmap’s dual role in duplicate detection and avoiding `add_packed_git()` overhead makes it non-trivial to relocate.

**Why it matters**
These discussions address **forward-compatibility** and **architectural trade-offs** in the ODB abstraction effort. The `odb_transaction_write_pack()` interface is a key enabler for pluggable backends, and the design gap could block future adoption if not resolved. Peff’s clarification about the existing hashmap highlights the **complexity of refactoring** low-level ODB internals, where performance optimizations are deeply entangled with correctness guarantees.

**Technical details**
- **`odb_transaction_write_pack()`**: The interface is intended to replace hardcoded `unpack()` logic in `git receive-pack` with a backend-agnostic callback. The `get_unpack_limit()` helper currently reads `transfer.unpackLimit` and `receive.unpackLimit`, but this is insufficient for other use cases (e.g., `git fetch-pack`).
- **Hashmap refactoring**: The existing hashmap in `packfile_store` avoids `add_packed_git()` overhead (allocations, `stat()` calls), making it non-trivial to relocate to `packfile_list` without redesigning packfile loading.

**Open questions**
- Whether the `ODB_TRANSACTION_RECEIVE`/`ODB_TRANSACTION_FETCH` flag approach will be accepted for the `odb_transaction_write_pack()` interface.
- Whether the hashmap refactoring is worth pursuing given its **single-caller context** and the complexity of separating duplicate detection from packfile initialization.

---

### Filesystem-level copy-on-write for `git worktree add`
**What changed?**
Peter Morris proposed leveraging **filesystem-level copy-on-write (CoW)** or block cloning (e.g., ReFS on Windows, Btrfs on Linux, APFS on macOS) for `git worktree add` to avoid physically duplicating data until modification. Junio C Hamano outlined the **core architectural challenge**: the checkout machinery (`checkout_entry()`) lacks visibility into sibling worktrees and operates in isolation. He proposed intercepting `checkout_entry()` to perform CoW based on a mapping of blob objects to cleanly checked-out files across all worktrees, while flagging **TOCTOU races** and side effects on other commands.

**Why it matters**
This proposal targets a **niche but growing use case**: concurrent AI coding agents or CI/CD pipelines that create many worktrees for the same repository. The optimization could reduce SSD wear, save disk space, and speed up worktree creation for large repositories. However, Junio’s feedback highlights the **complexity of refactoring** the checkout machinery, which is a critical path for many Git operations.

**Technical details**
- **Proposed behavior**: Filesystem-level CoW or block cloning during `git worktree add` to defer physical duplication until modification.
- **Architectural challenge**: The checkout machinery (`checkout_entry()`) has no awareness of sibling worktrees, requiring a mapping of blob objects to cleanly checked-out files across all worktrees.
- **TOCTOU races**: The source file might be modified between the mapping step and the CoW operation, requiring verification (e.g., hashing the copied file against the index).
- **Platform-specific APIs**: Windows (`CopyFile`), Linux (`ioctl_ficlone`), macOS (APFS).

**Open questions**
- Whether the **performance and storage benefits** justify the complexity of refactoring the checkout machinery.
- How to handle **TOCTOU races** and side effects on other commands (e.g., `git checkout`, `git reset`).
- Whether this aligns with Git’s **future roadmap** and priorities.

---

### MIDX incremental-write fix
**What changed?**
Patrick Steinhardt reiterated a preference for **direct verification** that the MIDX contains objects from *all* layers (not just the newest one) when using `--base=none`, calling the current bitmap-based test "a bit roundabout." The exchange leaves the **test-coverage question unresolved**, though the core fix remains uncontested.

**Why it matters**
The MIDX incremental-write series (v1) fixes a bug in the MIDX write path when using custom base layers (`--base=none` or `--base=<hash>`). The fix is **technically complete** and ready for integration, but the test-coverage debate highlights a **philosophical difference** in how thoroughly edge cases should be verified. Patrick’s preference for direct verification suggests lingering unease about the test’s thoroughness, though he acknowledges the point is minor.

**Technical details**
- **Bug**: The MIDX write path was not properly handling custom base layers during incremental writes, leading to incorrect reachability closure for bitmaps.
- **Fix**: Threads the `incremental_base` parameter through the write path and corrects pack inclusion logic for custom bases.
- **Test coverage**: The current tests verify bitmap generation, but Patrick prefers direct verification that the MIDX contains objects from *all* layers.

**Open questions**
- Whether the test-coverage question will delay integration or be addressed in a follow-up.

---

### Packfile performance regression fix
**What changed?**
Junio C Hamano **queued Johannes Schindelin’s patch** for the `next` branch, fixing a **quadratic-time performance regression** in `git rev-parse --short HEAD` (used in shell prompts). The regression was introduced in Git v2.53 (commit `589127caa730`) and manifests in Microsoft Git Scalar environments with shared cache repositories, where scheduled fetches deposit packfiles faster than maintenance can repack them. The fix adds a **fast-path optimization** to skip duplicate checks when the packfile is known to be new, eliminating the O(N²) scan.

**Why it matters**
The regression caused real-world slowdowns from 0.4s to 4.5s in Scalar environments, and the fix is **production-critical** for large repositories with many packfiles. Junio’s **substantive maintainability concerns** about the fast-path’s reliance on caller discipline remain unresolved but are deferred for post-merge follow-up.

**Technical details**
- **Root cause**: `packfile_store_add_pack()`’s linear scan of the packfile list before each insertion.
- **Fix**: Skip duplicate checks when the packfile is known to be new, using a `skip_dup_check` parameter in `packfile_list_append()`.
- **Performance**: The fast-path is explicitly faster than an O(1) hashmap lookup, as it avoids the lookup entirely.
- **Test coverage**: New perf test `t/perf/p5303-many-packs.sh` measures `git rev-parse --short HEAD` performance with 10,000 packs.

**Open questions**
- Whether the `skip_dup_check` parameter’s reliance on caller discipline will cause issues in practice (e.g., duplicates degrading non-existence checks).
- Whether the hashmap refactoring proposed by Patrick Steinhardt is worth pursuing given its **single-caller context** and the complexity of separating duplicate detection from packfile initialization.

---

### Bash completion improvements
**What changed?**
Two threads addressed **bash completion improvements**:
1. **`git history` completion**: Elijah Newren confirmed the v3 series looks good, addressing all feedback. The series adds completion for `git history` subcommands (`drop`, `fixup`, `reword`, `split`, and soon `squash`), their options, and positional arguments (revisions and pathspecs).
2. **Large repository performance**: Matthew Hughes reported a **performance regression** in bash completion for `git add` in repositories with hundreds of thousands of tracked files. The root cause is `git ls-files --others --modified --directory` with a wildcard pattern (e.g., `di*`), forcing Git to scan every directory. Hughes proposed a workaround: an environment variable `GIT_COMPLETION_NO_COMPLETE_INDEX` to disable index-based completion.

**Why it matters**
Bash completion is a **critical usability feature** for Git, and these threads address **real-world pain points**:
- The `git history` completion series fills a gap in the command’s usability, aligning it with other Git commands.
- The performance regression affects users with large repositories, where completion can hang for 1–2 seconds.

**Technical details**
- **`git history` completion**: Uses `__git_complete_index_file` for pathspec completion, with subcommand-aware value completion for `--empty` and `--update-refs`.
- **Performance regression**: `git ls-files --others --modified --directory` with a wildcard pattern generates hundreds of thousands of system calls (`getdents64`, `openat`, `fstat`).
- **Workaround**: `GIT_COMPLETION_NO_COMPLETE_INDEX` disables index-based completion, falling back to default bash file completion.

**Open questions**
- Whether the performance regression workaround is acceptable as a default or if a deeper fix (e.g., optimizing `git ls-files`) should be pursued.
- Whether the completion script should cache directory listings or use smarter heuristics to limit filesystem scans.

---

## In brief
- **`uploadpack.lazyFetchTrusted`**: Junio C Hamano provided **substantive reviews** of the series, raising design questions about the `path_allowlist_apply()` API and scalability concerns about holding the list of missing object names in memory. Christian Couder acknowledged the misthreaded v2 patches and agreed to resend with correct threading.
- **`repack --drop-filtered`**: The series was **proposed for `next`** after Christian Couder confirmed all prior feedback was addressed. The feature safely reclaims disk space in partial clones by removing locally cached promisor blobs exceeding a user-specified size threshold.
- **`git format-rev` formatting options**: Kristoffer Haugsbakk’s series to add `--abbrev`, `--color`, and `--date` options to `git format-rev` received **minor feedback** on commit message tense and designated initializers. The series is uncontroversial and on track for v2.
- **`git rev-parse` object name conversion**: Dimitri John Ledkov introduced `repo_oid_to_algop()` to compute object names in another hash algorithm on demand, enabling interoperability between repositories using different object formats (e.g., SHA-1 and SHA-256).
- **`submodule.active` warning**: Tilak Raaz’s GSoC microproject patch to warn on valueless `submodule.active` configuration entries received **feedback on error-handling logic** (fail vs. continue) and test formatting. Junio C Hamano questioned whether the command should fail after emitting the warning.
- **Documentation typo fixes**: Swapnil Saste fixed a typo in `Documentation/SubmittingPatches` ("an incremental updates" → "incremental updates"), and Elijah Newren updated the short help message for `git diff -l` to clarify its effect on rename/copy detection.
- **`chdir_notify` API cleanup**: Colin Hinton’s patch to remove the unused `name` parameter from the `chdir_notify` API was **merged to `next`** after addressing feedback on commit message clarity. The patch is a pure refactoring with no behavioral changes.