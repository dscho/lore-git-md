# Here’s the daily digest for June 16, 2026:

---

### **The day in brief**
A busy day on the Git mailing list (94 emails, 26 threads) with **security-hardened features landing**, **architectural debates heating up**, and **long-running efforts reaching milestones**. The standout developments: Pablo Sabater’s `git cat-file --batch-command` series cleared its final technical hurdle with Junio’s ack, while Patrick Steinhardt’s `git refs` consolidation sparked a design debate about atomicity and CLI consistency. Meanwhile, **Windows large-object support** and **SHA-256 interoperability** saw critical progress, and a **controversial `Assisted-by:` trailer proposal** divided contributors.

---

### **Notable threads**

#### **`git cat-file --batch-command` lands after 12 iterations**
Pablo Sabater’s **security-hardened** series implementing `git cat-file --batch-command` for remote object metadata queries received **Junio’s ack** for its final patch, resolving the last type-safety concern (a `size_t`/`unsigned long` mismatch) as an "evil merge" during integration. The series, now **ready for `next`**, replaces static allow-lists with **dynamic capability-based validation** and includes fixes for memory leaks in the transport helper. All core reviewers (Patrick Steinhardt, Jonathan Tan, Christian Couder, Taylor Blau) have approved the approach, which enables querying object sizes without full downloads—a boon for large-repository workflows.

#### **`git refs` consolidation sparks atomicity debate**
Patrick Steinhardt’s **four-patch series** adding `delete`, `update`, and `rename` subcommands to `git refs` faced **substantive pushback** from Junio over **atomic creation semantics**. The maintainer argued that the current `update` subcommand’s implicit creation of references lacks safety guarantees, proposing either a dedicated `create` subcommand or sentinel values (e.g., `""` for `<oldoid>`) to enforce "must not exist" semantics. The discussion highlights tensions between **discoverability** (unified `git refs` interface) and **safety** (explicit atomic operations). A v2 is likely, with implications for reftable and other ref backends.

#### **Windows large-object support inches closer**
Johannes Schindelin’s **8-patch series** converting `unsigned long` to `size_t` for object sizes on Windows (where the two types differ) received **Junio’s ack** for its delta-handling patch. The series, now **rebased and conflict-free**, addresses 4GB+ object support by systematically updating object-file, ODB, and hash algorithm code paths. All production changes have been validated in Git for Windows, and the series is **poised for `next`** pending final review. Key files: `object-file.c`, `sha1dc_git.c`, `odb/source-*`.

#### **SHA-256 interoperability hits a milestone**
Brian M. Carlson announced that **SHA-1/SHA-256 interoperability** now passes its test suite, with core functionality (shallow clones, submodules, partial clones) working. However, **critical limitations** remain: packfile URIs and promisor remotes are unsupported due to architectural constraints (e.g., unmapped objects in disconnected packs). Junio probed these gaps, particularly the **security implications** of trusting server-provided object mappings. The 200+ patch series is **not targeted for Git 3.0**, but the milestone marks progress toward a long-term goal.

#### **`Assisted-by:` trailer proposal divides contributors**
A **controversial RFC** from Marius Spix proposed adopting an `Assisted-by:` trailer for AI-assisted commits, mirroring the Linux kernel’s recent policy. Kristoffer Haugsbakk **pushed back hard**, arguing that Git has historically rejected project-specific trailers (citing a 2013 `Fixes:` rejection) and that the proposal would force Git to make policy decisions (e.g., auto-propagating "taint" during merges). The debate touches on **Git’s role in attribution standards** and whether it should normalize `--trailer` behavior to match `--signoff`. No consensus yet, but the thread reflects broader tensions around AI in open-source workflows.

---

### **In brief**
- **`git replay --linearize` merged to `next`**: Toon Claes’s series, which flattens merge commits into linear history, graduated after addressing all review feedback (root commit handling, test coverage, documentation). The feature provides a server-side alternative to `git rebase --no-rebase-merges`.
- **`git history reword` leak fixed**: Kaartic Sivaraam’s patch plugged a memory leak in `git history reword` when handling encoded commit messages. The fix adds a missing `repo_unuse_commit_buffer` call, with follow-up work identified for test coverage of encoded commits.
- **`git rebase --squash` faces architectural challenges**: Harald Nordgren’s proposal to add `--squash` to `git rebase` (folding commit ranges while preserving messages) was critiqued by Phillip Wood for **efficiency issues** (repeated conflict resolution) and **UX gaps** (no interactive message editing). Patrick Steinhardt endorsed a `git history squash` alternative, shifting the discussion toward his experimental command.
- **`git update-ref --rename` superseded**: Junio’s standalone `--rename` proposal for `git update-ref` was **deprecated in favor of `git refs rename`**, with Patrick Steinhardt agreeing to implement it under the unified `git refs` interface. The decision reflects a broader push to consolidate ref-related commands.
- **`git branch --delete-merged` nears completion**: Harald Nordgren’s 15-iteration series adding `--delete-merged` for safe branch cleanup received **final review feedback** from Phillip Wood on test design and API consistency. The series, which includes `--dry-run` support and per-branch opt-outs, is **one v16 away from merging**.
- **Documentation updates**: Kristoffer Haugsbakk’s series clarifying trailer conventions in `SubmittingPatches` advanced, with a **debate over non-identity trailers** (e.g., `Closes:`) still unresolved. Meanwhile, Patrick Steinhardt’s **b4 configuration standardization** (shallow threading, cover templates) received final sign-off and is **ready for `next`**.

---

### **On the radar**
- **`git history squash`**: Patrick Steinhardt’s endorsement of this alternative to `git rebase --squash` could shift the feature’s implementation path. Watch for design discussions around range selection and conflict handling.
- **Reflog handling in `git refs rename`**: Junio’s question about whether to create reflogs for renamed references (when the source lacks one) may require a `--create-reflog` option or default behavior change.
- **Test modernization conflicts**: Junio noted a `grep` vs. `test_grep` conflict in a bugfix series, highlighting the ongoing tension between legacy test patterns and modernization efforts.
- **ODB abstraction follow-ups**: Patrick Steinhardt’s **11-patch fix series** for refdb lifecycle issues (memory leaks, duplicate creation) remains blocked on the `ps/setup-centralize-odb-creation` topic. The ODB work is critical for reftable and other storage backends.