# Git mailing list daily digest for 2026/09/01

## The day in brief
GitLab's `--missing-only` workflow was clarified, resolving the last blocker for the `rev-list` feature. Elijah Newren's conflict-resolution safeguards series reached v4 with mechanical renames. Jeff King proposed a ground-up rewrite of trace2 to eliminate `die()`-prone dependencies. A regression in `git stash show` with `--src-prefix`/`--dst-prefix` was fixed. Skybuck Flying reported silent corruption of binary files during checkout due to overly broad `.gitattributes` rules.

## Notable threads

### GitLab's `--missing-only` workflow clarified
**What changed**: Siddharth Asthana clarified that GitLab's Gitaly uses `--missing-only` in a single-pass transaction packing workflow, not iterative onion-peeling, resolving Junio C Hamano's workflow question.

**Problem/goal**: Provide a script-friendly way to list only missing objects (without `?` prefix) while preserving existing `--missing=` formatting options for GitLab's Gitaly workflow.

**Subsystem**: `git rev-list`, partial clone workflows

**Impact**: The `--missing-only` option for `git rev-list` is now ready for final approval and integration, addressing GitLab's need to efficiently identify objects not present in a partial clone during transaction packing.

---
### Conflict-resolution safeguards extended
**What changed**: Elijah Newren posted v4 of the series extending Git's protection against unsafe commit operations (`git commit --amend` and `git commit <paths>`) during conflict resolution to cover `git am`, `git revert`, and all forms of `git rebase`.

**Problem/goal**: Prevent users from accidentally amending the wrong commit or updating individual paths during conflict resolution, which can corrupt repository state.

**Subsystem**: Merge machinery, sequencer internals

**Impact**: The series is technically complete and ready for integration, with all substantive feedback addressed. The only changes since v3 are mechanical renames to clarify intent (e.g., `FROM_REBASE_NOW_EMPTY` instead of `FROM_REBASE_EMPTY`).

---
### Trace2 hardening: ground-up rewrite proposed
**What changed**: Jeff King proposed a "ground-up no-dependency rewrite" of trace2 as the only way to fully eliminate `die()`-prone dependencies, expanding on his earlier architectural critique.

**Problem/goal**: Eliminate all `die()`-triggering helpers from the trace2 API to prevent Git crashes during telemetry operations, even under memory pressure or system call failures.

**Subsystem**: Trace2 (telemetry), memory allocation, MinGW compatibility

**Impact**: The current series blocks direct calls to banned functions but does not address indirect calls via dependencies like `strbuf` or `json-writer.c`. Jeff's proposal would involve creating a new, minimal string-handling interface tailored for trace2 and avoiding generic Git code that could trigger `die()`.

---
### `git stash show` regression fixed
**What changed**: Jeff King and Junio C Hamano posted a two-patch bugfix series addressing a use-after-free regression in `git stash show` with `--src-prefix`/`--dst-prefix` introduced in Git 2.52.0.

**Problem/goal**: Fix a use-after-free bug causing garbage diff headers (e.g., `diff --git Uf.txt Uf.txt`) when `--src-prefix` or `--dst-prefix` is used.

**Subsystem**: Revision parsing, diff generation

**Impact**: The fix introduces a `strvec argv_to_free` field in `struct rev_info` to delay freeing "freed" `argv` elements until `release_revisions()` is called, ensuring references stored in `diff_options` remain valid. The series is ready for `maint`.

---
### Binary file corruption during checkout
**What changed**: Skybuck Flying reported silent corruption of binary files (PNGs, JPEGs) during checkout due to overly broad global `.gitattributes` rules (`* text diff=lfclean`).

**Problem/goal**: Prevent Git from silently corrupting binary files during checkout when global `.gitattributes` rules are overly broad.

**Subsystem**: Checkout, attributes handling

**Impact**: The report highlights a usability issue where users may not realize their global `.gitattributes` rules are corrupting binary files. Randall S. Becker provided platform-specific advice (`core.autocrlf=input`, explicit binary declarations) and warned about filter fragility on binary data.

## In brief

- **[`git replay --linearize` approved]**: Elijah Newren's Reviewed-by on the entire series signals technical completion and readiness for integration.
- **[ODB alternates refactoring]**: Toon Claes approved the v2 cover letter for Patrick Steinhardt's 8-patch ODB alternates refactoring series.
- **[`receive-report` hook v5]**: Karthik Nayak posted v5 of the `receive-report` hook series, addressing all prior feedback and confirming the series is ready for integration.
- **[`git ident` alternative]**: Jeff King proposed extending `git var` instead of introducing `git ident`, arguing it could subsume all `git ident` functionality.
- **[CI Asciidoctor version pin]**: Jeff King noted the CI configuration's Asciidoctor version pin (1.5.8, 2018) is outdated and no longer necessary.
- **[ODB source linking removal]**: Junio C Hamano provided surface-level and substantive reviews of Patrick Steinhardt's 12-patch series removing ad-hoc ODB source linking.
- **[Parallel packfile URIs evicted]**: Patrick Steinhardt requested eviction of `ps/fetch-packfile-uris-parallel` from `next`/`seen` to avoid conflicts with Justin Tobler's upcoming ODB transaction work.