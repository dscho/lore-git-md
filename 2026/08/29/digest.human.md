# Git mailing list daily digest for 2026/08/29

## The day in brief

The Git mailing list saw key developments in two major patch series: the `dk/use-nsec-runtime` series converting `USE_NSEC` to a runtime option is now marked ready for `master`, while a four-patch bugfix series addressing geometric repacking race conditions has been integrated into Junio’s `next` branch. Jeff King provided final endorsements for the repacking fixes, and René Scharfe proposed technical alternatives to improve compile-time safety in a refactoring helper.

## Notable threads

### Runtime nanosecond precision configuration ready for `master`

The `dk/use-nsec-runtime` series, which replaces the build-time `USE_NSEC` macro with a runtime configuration option (`core.useNanosec`), is now explicitly described as "well-crafted, uncontroversial, and ready to advance to `master`" by author D. Ben Knoble. The series allows users to enable nanosecond precision on supported filesystems without recompiling Git, addressing "racy Git" problems. The v5 update incorporates all prior review feedback, including unconditional initialization of the `use_nanosec` field in `struct repo_config_values` and simplified `#ifndef NO_NSEC` guards in `is_racy_stat()`. The series is applied under its topic name and awaits graduation to `master`.

The three patches are self-contained: the first fixes a Meson build-system issue (`MAN_BASE_URL` configuration), the second reorders assignments in `repo_config_values_init()` for clarity, and the third introduces `core.useNanosec` and removes `USE_NSEC`. A trivial conflict with `ty/repo-config-cleanups` (which removes comments in `struct repo_config_values`) remains but is easily resolved. Performance benchmarks confirm the v2 fix for the v1 regression (~41× slowdown in `git diff`) is effective, with v5 restoring baseline performance (21 ms vs 20 ms).

### Geometric repacking race condition fixes integrated

A four-patch bugfix series addressing a race condition in Git’s geometric repacking mechanism has been integrated into Junio’s `next` branch. The series fixes crashes in `git replay` and missing-object errors in other operations (`git merge-tree`, `git diff`, `git rev-list`) caused by multi-pack-index (MIDX) references to removed packfiles. Jeff King (Peff) provided final endorsements for the core recovery logic in `odb/source-packed.c`, confirming the gating on `OBJECT_INFO_SECOND_READ` and iteration over `m->num_packs + m->num_packs_in_base` is correct and optimal. Peff also endorsed the removal of `OBJECT_INFO_QUICK` from `git mktree --batch` as aligned with the original intent of commit `817b0f602710`.

The series is well-motivated by production evidence and includes thorough test coverage. The core fix (patch 4/4) implements a tri-state return from `midx_fill_entry()` (`MIDX_FILL_MISS`, `MIDX_FILL_HIT`, `MIDX_FILL_OWNER_UNAVAILABLE`) and gates recovery on `SECOND_READ` to optimize performance for `QUICK` callers. The function has been renamed to `midx_fill_entry()` to reflect its new semantics. The only remaining loose end is a minor suggestion to improve error message clarity, but this is not a blocker.

### Compile-time safety proposals for refactoring helper

René Scharfe proposed two technical alternatives to improve compile-time safety for the `die_for_incompatible_opts()` helper, which replaces the fixed-arity `die_for_incompatible_optN()` family. The proposals address Jeff King’s concern that the varargs helper sacrifices compile-time protection for flexibility. The first approach combines `EOF` and `NULL` sentinels via a macro to prevent direct calls to the internal function, while the second introduces a type-safe compound literal (`struct used_option`) to enforce correct argument patterns at compile time. Neither proposal requires changes to existing call sites, and both preserve the series’ goal of handling more than four options.

The discussion remains focused on internal API safety, with no impact on user-visible behavior. Junio C Hamano acknowledged the trade-off but retained the `EOF` sentinel for backward compatibility. René’s contributions provide actionable paths forward for Junio to consider, though the series remains an early draft.

## In brief

- **`gitremote-helpers` documentation fix**: Lorenz Leutgeb documented the `if-asked` value for the `option pushcert` setting in `gitremote-helpers.adoc`, aligning the documentation with the existing implementation (commit `30261094b1f7fdcba3b7a1`).
- **Checkout refactoring follow-up**: Junio C Hamano updated `post_checkout_hook()` to accept a `struct repository *` parameter, making it reusable in non-global contexts. The fix aligns with the series’ goal of extracting reusable logic from built-in commands.