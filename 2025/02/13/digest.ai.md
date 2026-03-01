# Git Mailing List Digest — 2025/02/13

**The day in brief.** A busy Thursday with 58 emails across 19 threads, dominated by performance investigations in the reftable backend and ongoing patch series reviews. Key developments include a major reftable performance regression report, resolution of the `rebase --update-refs=interactive` design, and several build system improvements. The reftable discussion consumed nearly half the day's traffic as contributors worked to diagnose a 30x slowdown in mass ref operations.

## Notable threads

### Reftable backend shows 30x slowdown in mass ref operations

Brian M. Carlson reported a severe performance regression where creating 50,000 refs took nearly a minute with the reftable backend compared to 5.5 seconds with the traditional files backend. Patrick Steinhardt quickly reproduced the issue and identified tombstone records from prior deletions as the primary culprit, preventing auto-compaction while still requiring expensive conflict checks. The discussion evolved into a deep technical investigation with contributions from Jeff King and Christian Couder, revealing multiple optimization opportunities:

1. Skipping redundant `repo_get_oid()` calls in `update-ref` (already yielding a 3x improvement)
2. Reconsidering ambiguous refname warnings that add overhead
3. Potential iterator infrastructure changes to enable batched conflict checks

The thread represents a significant performance investigation that may lead to both immediate fixes and longer-term architectural improvements to the reftable backend.

### Rebase's interactive ref updating reaches design consensus

The long-running discussion about `git rebase --update-refs=interactive` reached resolution with agreement to:
- Keep the simple CLI interface unchanged
- Add `rebase.updateRefs=interactive` config for complex workflows
- Reserve `--update-refs` argument syntax for future pattern filtering

Phillip Wood and Ivan Shapovalov converged on this hybrid approach after exploring various branch management strategies. The design now cleanly separates configuration of interactive behavior from potential future filtering capabilities while preserving backward compatibility.

### Merge-recursive optimizations approved with caveats

Elijah Newren approved one of two performance patches for merge-recursive's string list handling, accepting the O(n²) to O(n log n) optimization for `process_renames()` while discouraging similar changes to `get_unmerged()`. The discussion revealed maintainers' strategic view of merge-recursive as legacy code not worth significant optimization effort, with attention better directed toward merge-ort. The approved change improves a hot path while setting clear boundaries for further work in this subsystem.

### Windows compatibility patches for reftable headers

Johannes Schindelin raised build integration concerns about Patrick Steinhardt's reftable decoupling series, specifically objecting to duplicate filenames in mingw/ and msvc/ directories that complicate Git-for-Windows maintenance. While technically sound for mainline Git, the current header organization creates practical problems for Windows builds that will need resolution before merging. This represents the final outstanding issue for an otherwise ready series.

## In brief

**Documentation standardization** M Hickford's patch to consistently use "title" rather than "subject" for commit message first lines in `git-commit.txt` was approved, with Junio Hamano suggesting similar updates for `git-rebase.txt`.

**Build system fixes** Two mechanical patches addressed Makefile syntax issues with zlib configuration and added Meson support for credential-wincred and credential-libsecret helpers.

**Mailmap handling** Jacob Keller reported a segmentation fault in `git check-mailmap` when processing full mailmap entries with email-only lookups, sparking discussion about expected behavior.

**Pickaxe documentation** Junio approved documentation fixes clarifying that `-G` matches changed lines (not just patches containing the regex) while suggesting these cleanups land separately from the long-options feature work.

**Platform compatibility** Zejun Zhao's series addressing `-Wsign-comparison` warnings in `apply.c` received detailed review feedback about type consistency between related fields.

**Pre-push hook limitations** Jayce Cao highlighted inefficiencies in identifying new commits during push operations, with Brian M. Carlson noting fundamental security limitations of the hook approach itself.

## On the radar

The **reftable performance investigation** will continue next week as Patrick Steinhardt works on iterator infrastructure changes to enable batched conflict checks. The 30x slowdown has revealed multiple optimization opportunities that may yield significant improvements for large repositories.