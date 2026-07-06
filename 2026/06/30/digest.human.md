Here’s the digest for **June 30, 2026**, covering the Git mailing list’s daily traffic:

---

### **The day in brief**
A busy but focused day on the Git mailing list, with **94 emails across 23 threads**. The standout developments:
1. **`git replay --linearize`’s design debate** heated up, with Johannes Schindelin and Patrick Steinhardt clashing over CLI consistency vs. usability.
2. **Phillip Wood’s 11-patch series** fixing rebase’s dropped-commit handling landed, addressing a long-standing bug and systemic issues in the sequencer.
3. **Patrick Steinhardt’s ODB abstraction work** saw multiple threads converge, with `ps/odb-generalize-prepare` and `ps/odb-drop-whence` advancing toward `next`.
4. **Usability tweaks** (e.g., `git blame -b`’s hash formatting, `git history squash`’s recoverability) sparked productive discussions.

---

### **Notable threads**

#### **`git replay --linearize`: CLI design vs. usability**
The debate over `git replay --linearize`’s interface intensified. Johannes Schindelin rejected Patrick Steinhardt’s proposal to mirror `git rebase`’s `--rebase-merges` syntax, calling it "full of usability warts" and advocating for distinct, intuitive options like `--linearize`. Patrick countered that any deviation from `git rebase` must be **explicitly justified** in commit messages. Toon Claes sided with Schindelin, arguing `--linearize` better describes the predictable flattening behavior. The thread remains unresolved, but the focus has shifted from technical correctness to **project-wide UX consistency**.

**Key takeaway**: The tension between CLI uniformity and usability is now a first-order design question for `git replay`.

---

#### **Rebase’s dropped-commit handling: fixed at last**
Phillip Wood delivered an **11-patch series** overhauling how `git rebase` handles dropped commits. The series fixes a critical bug where notes from dropped commits were incorrectly copied to HEAD, corrupting note history. It also addresses systemic issues like fixup-commit edge cases, final fixup cleanup, and edit/skip command handling. Junio C Hamano queued the series after a minor tooling hiccup (b4 dropped a patch), and the fixes are now slated for `next`. This resolves a years-old pain point and sets the stage for further sequencer improvements.

**Files touched**: `sequencer.c`, `t3400-rebase.sh`, `t5407-post-rewrite-hook.sh`.

---

#### **ODB abstraction: steady progress**
Patrick Steinhardt’s ODB abstraction effort saw two threads advance:
1. **`ps/odb-generalize-prepare`** (3 patches) was approved by Toon Claes and Junio, generalizing the `reprepare()` callback into a `prepare()` API with a flush flag. This enables `git grep` to work with pluggable ODB backends.
2. **`ps/odb-drop-whence`** (6 patches) refactored `struct object_info` to use a `source` field, replacing the coarse `whence` field. Junio and Justin Tobler debated the opt-in `source` parameter design, but the series is now conceptually approved.

**Impact**: These refactorings are foundational for Git’s long-term ODB abstraction goals.

---

#### **`git history squash`: recoverability and UX**
Harald Nordgren’s `git history squash` feature sparked a **user-centric discussion** about recoverability. Matt Hunter and Phillip Wood highlighted that `git reset --hard` may not fully undo operations that move multiple branch refs (e.g., with `--update-refs`). Harald proposed a **human-centered reflog** to track high-level workflows, while Phillip suggested adopting Jujutsu’s operation-logging model. The thread also resolved the `--reedit-message` template format, with Phillip’s cleaner design (omitting `fixup!` noise) now adopted.

**Key takeaway**: Recoverability is a systemic gap in Git’s UX, not just a `history` command issue.

---

#### **`git blame -b`: hash formatting fixed**
René Scharfe submitted a **bugfix patch** for `git blame -b`, addressing a long-standing annoyance where the command reserved an extra hex digit for an unused caret marker. The patch dynamically calculates mark columns, aligning hash lengths with `core.abbrev`. Junio and Laszlo Ersek confirmed the fix, which will ship in the next release.

**Files touched**: `blame.c`.

---

### **In brief**
- **`git refs` subcommands merged**: Patrick Steinhardt’s `delete`, `update`, `create`, and `rename` subcommands for `git refs` graduated to `next` after a final typo fix.
- **Memory leaks plugged**: Jeff King fixed leaks in `git format-patch` and the test harness, with Patrick Steinhardt proposing LSan for CI.
- **Test modernization**: Bryan B. Lima’s patch replacing raw `test -f` calls with descriptive helpers was queued.
- **Config case-sensitivity**: Rishav Dewan’s patch fixing case-sensitivity in old-style `[section.subsection]` headers awaits review from Johannes Schindelin.
- **Rust build cleanup**: Jan Palus removed a spurious Makefile dependency in the Rustification effort.

---

### **On the radar**
- **`git replay --linearize`’s interface**: Still unresolved; expect a follow-up patch or RFC.
- **`USE_NSEC`’s future**: Jeff King and Patrick Steinhardt debated runtime configuration vs. build-time knobs.
- **`git history squash` recoverability**: Harald’s reflog proposal may inspire broader UX work.

---

**Volume**: Heavy (94 emails), but dominated by focused technical discussions and patch series.
**Tone**: Collaborative, with productive debates on design trade-offs (e.g., CLI consistency vs. usability).
**Next steps**: Watch for `git replay --linearize`’s next iteration and Phillip Wood’s rebase fixes landing in `next`.