Here’s the digest for **2026/07/20**, a day of **moderate traffic (54 emails, 17 threads)** with a mix of **feature finalization, bugfix refinement, and procedural debates**. The standout themes: **Git’s source reorganization debate intensifies**, a **new `git history squash` command nears readiness**, and **long-running refactoring efforts reach key milestones**.

---

### The day in brief
A **busy but focused** day on the Git mailing list, with **two threads dominating attention**: Patrick Steinhardt’s controversial `lib/` directory reorganization (now with **brian m. carlson’s endorsement**) and Harald Nordgren’s **`git history squash` command (v10, functionally complete)**. Meanwhile, **Tian Yuchen’s `the_repository` removal series (v8) lands**, and **Junio’s meta-commentary on review culture** sparks a tooling debate. The day’s tone is **collaborative but contentious**—technical progress is steady, but **structural changes and workflow questions** continue to divide contributors.

---

### Notable threads

#### **1. Git’s source tree reorganization: the `lib/` debate heats up**
**Headline**: *Patrick Steinhardt’s RFC to move `libgit.a` sources into a `lib/` directory gains a high-profile supporter—but critics remain unmoved.*
**What’s new**: **brian m. carlson** (a long-time contributor) endorsed the reorganization in a **surface-level reply**, calling the current root-directory clutter "odd" and framing the change as a long-overdue cleanup. His support echoes **Johannes Schindelin’s combative defense** from earlier in the day, which dismissed **SZEDER Gábor’s objections** as "short-sighted" and drew a parallel to historical resistance to Git itself. The pro-reorganization camp now includes **Steinhardt, Schindelin, carlson, and Oswald Buddenhagen**, while critics (**Gábor, Junio C Hamano, Phillip Wood**) continue to emphasize **workflow disruption** (e.g., `git log --follow` pain, rebasing challenges) and question whether the benefits justify the costs.

**Key details**:
- **Files affected**: ~700 files (core subsystems like `object.c`, `refs.c`, `strbuf.c`) moved to `lib/`.
- **Build system**: Updates to `Makefile`, `meson.build`, `CMakeLists.txt`, and CI workflows.
- **Debate status**: **Polarized**. Proponents argue the reorganization enforces coding conventions (e.g., no `the_repository`/`die()` in library code) and improves discoverability; critics counter that the disruption is **ongoing** (not one-time) and that tooling improvements (e.g., `git show` hints for renamed files) are speculative.
- **Next steps**: No consensus yet. **Kaartic Sivaraam’s `git show` hint proposal** (suggesting renamed paths) remains a potential middle ground, but **Dscho’s latest reply doubles down on the reorganization’s necessity** without addressing practical concerns.

**Why it matters**: This is **Git’s most ambitious structural change in years**, with implications for **downstream forks, tooling, and contributor workflows**. The debate’s intensity reflects broader tensions between **codebase hygiene and stability**.

---

#### **2. `git history squash` reaches v10: ready for `next`?**
**Headline**: *Harald Nordgren’s `git history squash` command—designed to fold commit ranges into a single commit—posts its **tenth iteration**, addressing all prior feedback and earning Junio’s "Will replace" sign-off.*
**What’s new**: The v10 series is **functionally complete**, with fixes for:
- **Reflog message accuracy**: Now records the full revision expression (e.g., `squash: updating start..HEAD ^keep`) instead of truncating to the first argument.
- **Rev-list option sanitization**: Preserves the `boundary` walk invariant while enforcing `--reverse --topo-order`.
- **Documentation clarity**: Tightens descriptions of `amend!` and `--reedit-message` behavior.
The series is **technically sound**, with **Junio’s "Will replace" sign-off** on v7 and **no remaining blockers**. The only unresolved discussion—whether `--reedit-message` should be the default—is a **minor usability tweak** and not a blocker.

**Key details**:
- **Use case**: Fold a commit range (including merges) into its oldest commit **without the repeated conflict stops of `git rebase`**.
- **Behavior**: Rejects operations with refs pointing to commits inside the squashed range (with advice to use `--update-refs=head`), and groups `fixup!`/`squash!`/`amend!` commits under their targets in the `--reedit-message` template.
- **Test coverage**: 776 lines in `t/t3455-history-squash.sh`, covering edge cases like interior merges, ref handling, and `amend!` message selection.
- **Architectural vision**: Junio and **D. Ben Knoble** endorse using `git history squash` as a **faster backend for `git rebase --autosquatch`** when combined with `git replay`, avoiding rebase’s repeated conflict resolution.

**Why it matters**: This is a **foundational piece of Git’s future history-editing infrastructure**, with potential to **dramatically improve performance** for large-scale rebases. The series’ readiness signals a shift from design debate to integration.

---

#### **3. `the_repository` removal: Tian Yuchen’s v8 series lands**
**Headline**: *Tian Yuchen’s **four-patch series (v8)** to migrate `trust_executable_bit` and `has_symlinks` into `struct repo_config_values`—part of the broader `the_repository` removal effort—**addresses all prior feedback** and appears ready for `next`.*
**What’s new**: The v8 iteration resolves:
- **Build-breaking issue**: SZEDER Gábor’s report of an unused `repo` parameter in `ce_mode_from_stat()` (fixed by marking it `UNUSED` in patch 2/4, then removing the annotation in patch 3/4 when the parameter is used).
- **Stylistic feedback**: Junio’s suggestions to reformat the ternary expression in `repo_trust_executable_bit()` and remove defensive NULL checks.
The series is **mechanical and uncontroversial**, with **no behavioral changes**. It establishes a **template for future configuration migrations** and aligns with the project’s goal of eliminating global state.

**Key details**:
- **Files touched**: `read-cache.c`, `environment.c/h`, `apply.c`, `compat/mingw.c/h`, and related headers.
- **New symbols**: `repo_has_symlinks()` and `repo_trust_executable_bit()` getters.
- **Mingw compatibility**: Uses a compile-time macro (`platform_has_symlinks()`) and static function (`mingw_platform_has_symlinks()`) to eliminate mutable global state.

**Why it matters**: This is a **critical milestone** in the `the_repository` removal effort, demonstrating **incremental progress** while avoiding disruptive changes. The series’ success may accelerate similar migrations.

---

#### **4. Rebase dropped-commit notes: Phillip Wood’s v3 series approved for `next`**
**Headline**: *Phillip Wood’s **nine-patch series** to fix the sequencer’s incorrect copying of notes from dropped commits **earns Junio’s approval** and is marked ready for `next`—despite an **unresolved regression in interactive rebase**.*
**What’s new**: Junio confirmed the series is **"ready to go"** pending typofix squashing, even though **Uwe Kleine-König reported** that the fix **fails for interactive rebase (`-i`)**. The regression affects **user-initiated drops** (e.g., `git rebase --skip` or `--continue` without committing), where notes are still incorrectly copied. Phillip clarified that the series only handles commits dropped by `--empty=drop` (non-interactive default), not the interactive backend’s `--empty=ask` behavior (which drops on `--continue` but keeps on `--skip`). A **follow-up series is expected** to address the interactive rebase gap.

**Key details**:
- **Files touched**: `sequencer.c`, test scripts (`t3400-rebase.sh`, `t3404-rebase-interactive.sh`, `t5407-post-rewrite-hook.sh`).
- **New symbols**: `enum pick_result` (replaces integer return values in `do_pick_commit()`).
- **Behavior change**: Notes and post-rewrite hooks no longer receive information about dropped commits (for `--empty=drop` only).

**Why it matters**: This is a **long-overdue bugfix** for a subtle but **data-corrupting issue** in rebase. The **interactive rebase regression** highlights the complexity of Git’s sequencer backends, but Junio’s approval signals confidence in the incremental approach.

---

#### **5. Review culture debate: Junio vs. Oswald on Gerrit vs. mailing lists**
**Headline**: *Junio C Hamano’s **meta-commentary on mailing-list review culture**—distinguishing between `Acked-by` (low-confidence) and `Reviewed-by` (substantive)—sparks a **procedural counterpoint** from Oswald Buddenhagen, who argues that **Gerrit’s multi-level scoring system already captures this nuance**.*
**What’s new**: The exchange is **purely meta-level**, with no impact on in-flight patches. Key points:
- **Junio**: Mailing-list replies are **more nuanced** than Gerrit’s binary +1 system, as they can convey **low-confidence acknowledgments** (`Acked-by`) vs. **substantive reviews** (`Reviewed-by`).
- **Oswald**: Gerrit’s **-2 to +2 scale** already encodes this nuance, and the mailing list’s workflow is **less efficient** for low-value verdicts.
- **Context**: The discussion arose from Oswald’s **surface-level ack** of Phillip Wood’s rebase series, which Junio framed as an `Acked-by`.

**Why it matters**: This is a **recurring theme** in Git’s development: **tooling vs. culture**. The debate reflects broader questions about **how to scale review** without losing the project’s collaborative ethos.

---

### In brief
- **`git repo info` path keys**: K Jayatheerth’s GSoC series adding path-related keys (e.g., `path.objects`, `path.hooks`) **gains maintainer feedback** on `path.grafts` (unconditional path reporting is intentional) and `path.index` (should return empty in bare repos). **Lucas Seiki Oshiro** raised both issues.
- **`git bisect --reset-when-found`**: Harald Nordgren’s **v3 series** (auto-reset after bisect) is **marked for `next`** after addressing platform-compatibility concerns. The option is now parameterized (`--reset-when-found[=<where>]`) with values `original` or `found`.
- **`git repack --drop-filtered`**: Siddharth Shrimali’s RFC for partial-clone space reclamation **acknowledges a blocking flaw** in `--write-bitmap-index` validation (fix planned for v2). **Christian Couder** reviewed the CLI design.
- **Swift userdiff driver**: Shlok Kulshreshtha’s **v2 patch** (adding Swift support to `git diff`) **addresses all review feedback** but remains in `seen` pending validation from Swift users. **Johannes Sixt** retracted an earlier optimization suggestion.
- **`git last-modified` Bloom filters**: Toon Claes’s performance optimization series **resolves the `--show-trees` correctness bug** (filtering uncovered paths with `match_pathspec()`). **Jeff King** and **Taylor Blau** endorsed the fix.
- **Documentation synopsis style**: Jean-Noël Avila’s **four-patch series** (converting `imap-send`, `format-patch`, `send-email`, `request-pull` to synopsis style) **awaits a reroll** to fix minor stylistic inconsistencies (backtick usage, dash count in AsciiDoc blocks).
- **Push tracking after remote rename**: Harald Nordgren’s **two-patch series** (fixing `git status` push display after remote rename) **needs a revised commit message** to clarify the URL-based lookup design. **Junio** requested the change.
- **`git config --global` and `include.*`**: The **bug report** is resolved: the behavior is **intentional**, and `lbmk` should use `git var GIT_AUTHOR_IDENT` instead. **Jeff King** and **Junio** closed the design discussion.
- **`contrib/fast-import/import-zips.py`**: Junio **declined to accept** Chris Packham’s Python 3 update, favoring **removal of unmaintained scripts** from `contrib/`.

---

### On the radar
- **`no-ref-delta` capability**: Taylor Blau’s **pack protocol extension** (for write-through proxies) remains under review, with **Jeff King** now acknowledging its potential optimizations but still questioning its **narrow applicability**.
- **Trace2 recursion crash**: Derrick Stolee’s **bugfix** (removing `xsnprintf()` from trace2) and Taylor Blau’s **alternative** (disabling `libintl_vsnprintf()` redirection for `xsnprintf()`) are **converging on a two-part solution**, but **Junio** suggested implementing the latter in a **MinGW-specific file**.
- **Coverity unchecked return values**: Johannes Schindelin’s **11-patch series** (hardening Git against Coverity warnings) **addressed Junio’s correctness concerns** (fifth and ninth patches) and awaits resolution of **Patrick Steinhardt’s design questions** (tenth and eleventh patches).