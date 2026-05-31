# Git Mailing List Digest — 2026/05/30 (Saturday)

**The day in brief.** A moderately active Saturday with 19 emails across 10 threads, featuring documentation improvements, configuration refactoring, and a few bugfix discussions. Notable items include Junio's review of Kristoffer Haugsbakk's `git replay` documentation series and Tian Yuchen's ongoing work to eliminate `the_repository` global variable.

---

## Notable threads

### `git replay` documentation finalized

Kristoffer Haugsbakk's 4-patch series standardizing `git replay` configuration documentation received maintainer approval from Junio C Hamano. The work clarifies the relationship between `replay.refAction` and `--ref-action` options, converting bullet lists to nested definition lists and improving cross-references between man pages. Junio noted minor usability tradeoffs in moving some details from config to command documentation but confirmed the technical approach as correct. The series exemplifies Git's documentation maintenance workflow — precise improvements reviewed for convention compliance — and is now ready for merging.

### `git describe` pattern matching fix hits edge case

Jacob Keller's bugfix for `git describe`'s handling of `--exclude` and `--match` with `--contains --all` encountered an unexpected CI failure. The test expected "branch_A" but got "remotes/origin/remote_branch_A", suggesting exclude patterns aren't properly applied to remote branches in `--all` mode. This edge case wasn't caught by initial test coverage. Junio flagged the issue, which may require revisiting how exclude patterns are processed for remote refs. The original patch had aligned implementation with documented behavior; this appears to be a refinement case rather than a fundamental flaw.

### Ancient timestamp documentation added

Following a bug report about inconsistent parsing of small Unix timestamps, Junio C Hamano provided a patch documenting the `@` prefix workaround for very old dates (like Unix epoch 0). The change explains in `date-formats.adoc` that the prefix forces raw timestamp interpretation, necessary for values below 100,000,000 to avoid YYYYMMDD confusion. Test cases demonstrate the difference between prefixed and unprefixed small timestamps. The patch cleanly addresses the documentation gap while leaving the underlying parsing inconsistency as a known issue.

### `trust_executable_bit` refactoring reviewed

Tian Yuchen's series moving `trust_executable_bit` configuration into `repo_config_values` (part of the `the_repository` removal effort) received substantive feedback from Junio. He questioned the overhead of unconditionally calling `repo_config_values()` in hot paths like `ce_mode_from_stat()`, and suggested migrating related `has_symlinks` setting simultaneously for design cohesion. Christian Couder earlier noted a struct field ordering inconsistency. The series aims to eliminate this global variable by making it repository-scoped, with patches cleaning up declarations, relocating functions, and updating call sites to pass state explicitly.

---

## In brief

**`git log --follow` merge handling** — Miklos Vajna proposes refined logic for following renames across merge commits, particularly subtree merges where files are added in one parent but renamed in another. The tweak would follow the parent with meaningful changes when a path is untouched or just added in others.

**`git history` signature handling** — Alix Brunet questions whether the new `git history` command will preserve commit signatures like `git rebase -S` does, noting current behavior drops them during operation. The query touches on consistency between related history-rewriting commands.

**Off-topic spam** — A gardening/DIY article pitch mistakenly sent to the Git list was promptly ignored by all participants.

---

## On the radar

**Reftable compaction** — Patrick Steinhardt's recent fix for a reftable edge case remains in `next`, having graduated from `seen` in the last "What's cooking" report. The change prevents silent ref drops when tables share deletion tombstones.

**Rustification** — Ezekiel Newren's effort to introduce Rust code continues to generate discussion about platform support concerns, particularly for NonStop. Randall S. Becker remains the primary voice raising these compatibility issues.