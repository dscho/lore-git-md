# Git Mailing List Digest - 2025/05/12

## The day in brief

A busy day with 126 emails across 28 threads, featuring significant discussions around deprecation infrastructure, submodule handling, and performance optimizations. Key highlights include the completion of the `git whatchanged` deprecation series, a new `--dry-run` option for `merge-tree`, and ongoing debates about Change-ID implementation and `git stash` behavior in submodule contexts.

## Notable threads

### `git whatchanged` deprecation completed

Junio Hamano's 6-patch series implementing the full deprecation lifecycle for `git whatchanged` reached its final form after incorporating feedback from Elijah Newren and Patrick Steinhardt. The series follows the established pattern from the earlier `pack-redundant` deprecation, including:

- A reusable `you_still_use_that()` helper function for deprecation warnings
- Documentation updates across 8 files replacing `whatchanged` references
- Test modernization replacing `whatchanged` with `log --raw`
- Conditional compilation removal via `WITH_BREAKING_CHANGES`
- BreakingChanges.adoc entry documenting the removal

The series demonstrates Git's methodical approach to command deprecation, with careful attention to documentation hygiene and test coverage. Only minor documentation tweaks remain before merge.

### Change-ID standardization debate continues

The Change-ID discussion saw substantive technical exchanges about distributed implementation approaches. Junio Hamano proposed making predecessor pointers optional during transfer (similar to promisor objects) to optimize bandwidth, while Martin von Zweigbergk clarified Jujutsu's working implementation uses random Change-IDs. 

Brian m. carlson raised security concerns about random IDs, suggesting deterministic SHA-256 hashes instead to prevent potential credential leakage through commit metadata. The thread remains active with fundamental questions about Change-ID generation and transfer semantics still unresolved.

### Stash branch display fix for submodules

K Jayatheerth identified and fixed a bug where `git stash list` incorrectly showed submodule branch names instead of superproject branches in stash entries. The root cause was buffer reuse in `refs_resolve_ref_unsafe()` during submodule operations. The solution uses `xstrdup()` to preserve the superproject branch name, with the patch receiving positive review from Junio Hamano pending test coverage and minor code quality fixes.

## In brief

**SHA-256 support for gitk** -- Takashi Iwai and Johannes Sixt continued refining the implementation, with discussions about hash length preferences and remaining hardcoded SHA-1 assumptions in several procedures.

**Scalar maintenance configuration** -- Derrick Stolee and Junio Hamano finalized the `scalar reconfigure` interface, simplifying from a tri-state model to OPT_BOOL with "keep" as default behavior.

**send-email FQDN detection** -- Aditya Garg's platform-specific approach (`--fqdn` for Linux, `-f` for macOS) gained approval after extensive discussion about cross-platform compatibility and semantic clarity.

**Reftable performance fix** -- A patch from shejialuo addressed a 1.62x performance regression in reftable block reading by adding early type checking to avoid unnecessary decoding.

**json-writer documentation** -- GSoC participant Lucas Seiki Oshiro added comprehensive API documentation, receiving detailed style feedback from Patrick Steinhardt and Karthik Nayak.

**contrib directory cleanup** -- Patrick Steinhardt's systematic removal of unmaintained contrib components progressed to v3, though Phillip Wood argued for keeping `git-contacts` due to its role in contributor onboarding.

## On the radar

**Stash default behavior change** -- Junio Hamano expressed reservations about making `--index` the default for `git stash apply`, citing personal workflow disruptions from similar aliases. The discussion may influence whether this change proceeds under `WITH_BREAKING_CHANGES`.

**Submodule configuration safety** -- K Jayatheerth proposed automatic unique naming for conflicting submodule paths instead of `--force` overwrites, aligning with Junio Hamano's preference for safer defaults.

**Packed-refs empty file handling** -- Consensus emerged to treat empty packed-refs files as warnings (not errors) in fsck, respecting historical compatibility while allowing sites to enforce stricter checks.