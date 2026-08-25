# Git mailing list daily digest for 2026/08/24

## The day in brief
The Git mailing list saw active discussion on several fronts today. Key developments include a maintainer clarification reinforcing the motivation for a global config fix, a design debate over strict vs. permissive OID matching in `git history squash`, and forward-looking discussions about ODB deadlock prevention and AI-generated contributions. The `report` hook series nears graduation with a naming consensus, while a geometric repacking race condition fix faces architectural scrutiny. Junio’s "What’s cooking" report confirmed five ODB/performance topics ready for `next`.

## Notable threads

### Global config listing inconsistency fix
**What changed?**
Junio C Hamano clarified that the discrepancy addressed by Delilah Ashley Wu’s three-patch series (where `git config list --global` only read `$HOME/.gitconfig` despite Git reading both config files) was unintentional, reinforcing the series’ motivation.

### Why it matters

The series fixes a long-standing inconsistency in Git’s config handling, ensuring `--global` operations read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` as documented. The maintainer’s confirmation removes any lingering doubt about the necessity of the fix.

**Key files**: `config.c`, `builtin/config.c`

---

### `git history squash` autosquash marker resolution
**What changed?**
The design debate over OID case-sensitivity in `fixup!`/`squash!`/`amend!` markers narrowed to two options: strict case-sensitive matching (rejecting all uppercase OIDs) or a permissive approach matching `git rebase --autosquash` (accepting all-uppercase/all-lowercase OIDs while rejecting mixed-case). Junio C Hamano argued for strict matching, citing Git’s historical convention of emitting only lowercase hexadecimal OIDs.

### Why it matters

This is the final technical hurdle for Harald Nordgren’s `git history squash` series, which aims to efficiently collapse commit ranges into their oldest commit. The resolution will determine whether the implementation aligns with Git’s existing conventions or prioritizes consistency with `git rebase`.

**Key files**: `builtin/history.c`, `sequencer.c`

---

### Sequencer ODB deadlock prevention
**What changed?**
Phillip Wood proposed two forward-looking approaches to prevent ODB deadlocks during interactive rebase: (1) passing `-c gc.auto=false` to `git commit` (since `run_auto_maintenance()` runs at rebase end anyway), and (2) automatically closing the ODB whenever any Git command is spawned. Junio C Hamano endorsed the latter as a "great approach to study" for long-term refactoring, noting the idea dates back to 2007 discussions.

### Why it matters

The discussion elevates the scope of Johannes Schindelin’s Windows-specific bugfix from a one-off tweak to a potential architectural improvement. The global ODB-release mechanism could eliminate the need for similar fixes across many code paths.

**Key files**: `sequencer.c`

---

### `report` hook naming consensus
**What changed?**
Junio C Hamano and Patrick Steinhardt objected to the hook name `report` as overly generic, suggesting `receive-report` as a more specific alternative. Karthik Nayak’s series is now effectively complete, with all prior feedback addressed and only the naming left to finalize.

### Why it matters

The `report` hook enables server-side status filtering for `git-receive-pack`, addressing GitLab’s need for MVCC (multi-version concurrency control). The naming consensus removes the last blocker before graduation to `next`.

**Key files**: `builtin/receive-pack.c`, `Documentation/githooks.adoc`

---

### Geometric repacking race condition fix
**What changed?**
Jeff King (Peff) and Patrick Steinhardt refined the design for Elijah Newren’s fix to a race condition where a stale multi-pack-index (MIDX) references a removed packfile. Peff argued the fallback logic should only trigger during `SECOND_READ` to optimize performance for `QUICK` callers, while Derrick Stolee suggested geometric repacking adopt a grace period for redundant packfiles (similar to `multi-pack-index expire`) to reduce the race condition’s frequency.

### Why it matters

The fix addresses crashes in `git replay` and missing-object errors in server-side operations. The architectural debate centers on balancing correctness, performance, and operational simplicity, with implications for future race-condition cleanups.

**Key files**: `odb/source-packed.c`, `replay.c`

---

### AI-generated contributions policy
**What changed?**
Oswald Buddenhagen rejected Junio C Hamano’s framing of "contributor trust" as a sufficient safeguard for AI-generated code, arguing that AI tools inherently undermine the ability to verify code provenance—even for trusted contributors. This narrows the policy discussion to a binary choice: maintain the strict prohibition or accept the legal risks of AI-generated contributions.

### Why it matters

The discussion affects Alexey Samsonov’s `utimensat()` series, which was generated by an AI tool (Gemini Flash) and manually verified. The Project Leadership Committee (PLC) or broader community must clarify whether the policy will be amended to reflect a more nuanced stance or upheld as-is.

**Key files**: `SubmittingPatches` (policy document)

---

## In brief
- **`git symbolic-ref` exit codes**: Nikolaus Schuetz proposed documenting the current behavior (exit code 128 without `--quiet`, exit code 1 with `--quiet`) rather than aligning the exit codes, sidestepping the design debate.
- **`git pull --hard`/`--reset`**: Junio C Hamano suggested the desired functionality (fetching upstream and resetting the working tree) belongs in a user-configured alias rather than a built-in command, dismissing both the original proposal and Artur Bieniek’s `reset --pull` alternative.
- **`git organize` directory layout**: Oswald Buddenhagen advocated for nesting subsystems under `lib/` (e.g., `lib/odb/`) to align with Patrick Steinhardt’s `libgit.a` effort, while also criticizing the use of double quotes for cross-subsystem includes.
- **`git format-patch --range-diff-notes`**: Junio C Hamano raised a usability concern about the option semantics, noting that `--no-notes --range-diff-notes` would suppress notes in both patches and range-diffs, contradicting intuitive expectations.
- **ODB abstraction**: Junio’s "What’s cooking" report confirmed five topics ready for `next`: `ps/odb-geometric-repack-loose-threshold`, `ps/odb-pluggable-pack-generation`, `jt/receive-pack-pluggable-writes`, `ps/odb-eagerly-load-alternates`, and `vm/complete-history`. Patrick Steinhardt endorsed merging them.