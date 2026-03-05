Here's the daily digest for September 10, 2025:

## The day in brief
A busy day with 104 emails across 26 threads, dominated by major technical discussions around Rust integration, command deprecation, and atomic reference updates. The Rust infrastructure series saw significant refinement, while several documentation and bugfix efforts reached completion. Junio's "What's cooking" report provided a comprehensive snapshot of in-flight topics.

## Notable threads

### Rust infrastructure advances
Patrick Steinhardt's Rust integration series (v4) saw extensive discussion, now expanded to 9 patches. The series makes Rust mandatory by Git 3.0, introducing meson build system integration and converting the varint subsystem as a test case. Key changes include explicit integer width handling (addressing type safety concerns) and a Cargo-first build approach. Junio provided feedback on documentation wording around the mandatory transition timeline and LTS maintenance implications. The thread revealed ongoing tensions between thorough platform testing (particularly Windows) and accelerated adoption timelines.

### Command deprecation framework nears completion
The command deprecation series (kh/you-still-use-whatchanged-fix) is nearly merge-ready after addressing a memory leak in the alias shadowing implementation. Jeff King proposed restructuring to use `handle_alias()` directly rather than creating redundant wrapper functions. The series updates deprecation messaging and adds infrastructure for shadowing deprecated builtins like `whatchanged`. Junio confirmed the series will be held until the leak fix is finalized, marking one of the last steps before integration.

### Atomic reference updates design converges
The `git replay` atomic reference updates discussion reached consensus on interface design after extensive review. Siddharth Asthana simplified the proposal to make atomic updates the default behavior, with `--output-commands` preserving pipeline compatibility and `--allow-partial` handling non-atomic cases. This resolves earlier debates about flag naming (`--update-refs` vs `--update`) while addressing performance concerns raised by Patrick Steinhardt. Documentation updates will reflect the new default behavior ahead of merging.

### Signature handling comes to fast-import
Christian Couder proposed a two-patch series adding `--signed-commits` to `git fast-import`, mirroring existing `fast-export` capabilities. The first patch refactors shared signature mode handling into gpg-interface.c, while the second implements the new option with comprehensive test coverage. Junio's review focused on implementation details in `import_signature()` and questioned whether tag support should follow in a subsequent patch. The series enables tools like git-filter-repo to better manage signatures during repository operations.

## In brief

**Submodule path encoding fix** -- Adrian Ratiu addressed compiler warnings about unused variables in the submodule path encoding series, with fixes planned for v3.

**Subtree regression fix finalized** -- Colin Stagner's fix for squashed subtree merges under prefixes completed review with all feedback incorporated, ready for maint-2.44.

**git-checkout documentation overhaul** -- Julia Evans' v4 series refining `git-checkout.adoc` completed review, making the command's dual nature (branch switching vs file restoration) clearer while maintaining technical precision.

**Default branch name transition** -- Phillip Wood's series switching from 'master' to 'main' advanced to v3 with test suite updates and breaking-changes documentation.

**Clar test framework update** -- The unit test framework received updates including improved platform support and better error reporting via `cl_invoke()`.

**ODB wrapper removal** -- Patrick Steinhardt removed deprecated object database wrapper functions as part of the ongoing `the_repository` elimination effort.

## On the radar

**Rust rollout strategy** -- The timeline for default Rust enablement remains under discussion, with Windows support as the key outstanding blocker.

**SHA-1/SHA-256 interoperability** -- Brian m. carlson's large patch series (93 patches) may need to be deferred from Git 3.0 due to scope, with Patrick Steinhardt offering assistance.

**Private branch proposal** -- An informal suggestion for private branch functionality generated initial discussion about whether this belongs in core Git versus platform implementations.