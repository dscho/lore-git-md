Here's the daily digest for April 4, 2026:

## The day in brief

April 4 saw moderate traffic with 69 emails across 15 threads, dominated by ongoing test modernization work and several feature refinements. The most notable developments include Johannes Schindelin's comprehensive test suite updates for bare repository handling and Adrian Ratiu's parallel hook execution series reaching merge readiness. A critical `git diff --no-index` NULL dereference bug also surfaced, prompting discussion about deeper object database fixes.

## Notable threads

### fsmonitor split-index interaction analysis

Johannes Schindelin provided a deep analysis of an edge case where fsmonitor interacts with split-index and `index.skipHash`, arguing that Paul Tarjan's current patch addresses symptoms rather than root causes. The issue occurs when Scalar sets `index.skipHash=true`, leading to a null shared index OID that fsmonitor's bitmap references. Schindelin identified three potential fixes, suggesting the fsmonitor series should drop this patch and address the core index.skipHash limitation separately. This shifts focus from fsmonitor-specific work to broader index handling concerns.

### Parallel hook execution reaches maturity

Adrian Ratiu's parallel hook execution series (v6) appears ready for merging after addressing all technical feedback. The comprehensive implementation now includes:
- Global `hook.jobs` defaults with `-1` for CPU core count
- Per-hook `hook.<name>.parallel` flags
- CLI `-j/--jobs` override precedence
- Per-event `hook.<event>.jobs` settings
- Per-event `hook.<event>.enabled` master switches

The series has maintainer approval and only minor philosophical questions remain about namespace organization. The final version includes Patrick Steinhardt's meson.build fix and demonstrates careful attention to backwards compatibility, particularly for the pre-push hook's output handling.

### Test suite modernization for bare repositories

Johannes Schindelin continued his 17-patch series updating Git's test suite to explicitly handle bare repositories rather than relying on implicit discovery. The changes systematically convert tests to use `--git-dir` for bare repos and `-C` for non-bare, preparing for a potential future `safe.bareRepository=explicit` default. The v2 iteration addresses Junio Hamano's feedback about `ext::` URL handling in t5509 by using `--git-dir=.` after `-C`. Schindelin emphasizes this work has standalone value as test hygiene improvement regardless of any eventual default change.

### `git diff --no-index` NULL dereference bug

A segmentation fault in `git diff --no-index` with large files prompted discussion about proper handling of repository-less operations. The crash occurs when accessing `the_repository->objects` (NULL in this mode) during large file handling. Initial fixes proposed checking `INDEX_WRITE_OBJECT`, but Jeff King suggested a more comprehensive solution at the object database transaction level to gracefully handle NULL sources. The thread awaits input from object-database experts, particularly Justin Tobler, on whether to implement a targeted fix or broader architectural change.

### `the_repository` removal in refs backend

Shreyansh Paliwal's refactoring series (v5) continues converting the refs subsystem to explicitly pass repository context rather than using globals. The current patches address lock timeouts, hash algorithm usage, and reftable backend changes. The series demonstrates incremental progress in René Scharfe's long-running effort, with the author responsive to review feedback about remaining static variables and future work needed for full multi-repo support.

## In brief

**Documentation standardization** -- Jean-Noël Avila submitted four patches converting `git-difftool`, `git-range-diff`, `git-shortlog`, and `git-describe` manuals to synopsis-style AsciiDoc format, continuing the long-running documentation cleanup effort.

**Graph rendering improvements** -- Pablo Sabater's v2 patch improves `git log --graph` output for commits with excluded parents by introducing a placeholder mechanism that prevents misleading vertical alignment of unrelated lineages.

**`git stash` argument parsing** -- Deveshi Dwivedi's patch makes `git stash` automatically assume "push" when unambiguous push-specific options (`-m`, `--staged`, etc.) are present, improving usability while maintaining backwards compatibility.

**`git switch` detach behavior** -- Thibaud CANALE proposed making `git switch`'s commit hash behavior configurable to match `git checkout`'s default detach mode via `checkout.switchDetach`, though questions remain about scripting impacts.

**Backfill sparse-checkout fix** -- Trieu Huynh corrected the backfill command's sparse-checkout integration by properly initializing the sparse flag to allow config auto-detection, receiving final approval from Derrick Stolee.

## On the radar

**C23 const-correctness** -- Jeff King followed up on his completed const-correctness series with minor documentation and indentation fixes for the new CONST_OUTPARAM macro.

**April Fools' aftermath** -- A late addition to the hash algorithm joke thread provided meta-commentary comparing the exchange to web development culture, serving as a lighthearted postscript to the concluded discussion.