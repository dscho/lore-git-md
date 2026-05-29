# Here's the daily digest for 2026/05/28:

### The day in brief
A moderately busy day with 48 emails across 20 threads, featuring several bugfixes nearing completion, platform-specific discussions, and a feature proposal sparking debate. Key highlights include progress on Windows signal handling, a resolved `git mv --index-only` discussion, and multiple patch series reaching final iterations.

### Notable threads

### Windows signal handling alignment

Johannes Schindelin revealed that Git for Windows already implements nuanced SIGTERM/SIGKILL differentiation through `atexit()` handlers, diverging from both native Windows behavior and the current patch's approach. Junio C Hamano accepted Schindelin's plan to upstream this more sophisticated solution, putting the original patch on hold. This thread demonstrates how platform-specific implementations sometimes evolve independently before being reconciled with mainline Git.

### `git mv --index-only` workflow concluded

After thorough discussion, Junio definitively closed the `git mv --index-only` proposal by demonstrating that index/working-tree divergence is intentional Git behavior. While technically feasible, the niche use case didn't justify core implementation when existing commands (`git rm --cached` + `git update-index`) achieve the same end state. The thread showcased Git's conservative approach to adding specialized flags.

### External notes command support

Siddh Raman Pant pinged their v3 series adding configuration-driven external notes display, featuring timeout handling and extensive tests. The comprehensive 6-patch implementation has evolved through multiple iterations but awaits maintainer attention. This represents a significant feature addition that could enable richer notes integration while maintaining security through process isolation.

### Config file locking timeouts

The discussion around Jörg Thalheim's config locking timeout series took a philosophical turn as Johannes Schindelin and Junio agreed numeric config values should ideally include units, though acknowledged existing conventions make this challenging to change. The substantive debate about process-wide versus per-repo timeout behavior remains unresolved, awaiting further implementation refinement.

### Line-log integration finalized

Michael Montalbo's line-log integration series reached completion with a v3 addressing the last test syntax nit. The changes unify `-L` with Git's standard diff pipeline while carefully bounding supported formats. A follow-up discussion about test helper documentation standards emerged, with Montalbo committing to address this project-wide need separately.

### In brief

**`git describe` pattern matching fix** -- Jacob Keller corrected `--match`/`--exclude` pattern forwarding when using `--contains --all`, aligning implementation with documented behavior.

**Deprecated commit list removal** -- Kristoffer Haugsbakk completed the removal of long-deprecated functions from commit.h after verifying no remaining callers, with acknowledgment from original author Patrick Steinhardt.

**macOS linker warning resolution** -- Consensus formed around using `-fno-common` compiler flag to suppress Xcode 16.3+ warnings about pkt-line buffer alignment, avoiding source modifications.

**HTTP pack index leak fix** -- Lorenzo Pegorari's v2 patch now addresses both memory leaks and redundant tempfile cleanup in HTTP pack handling based on Jeff King's review.

**Windows pathspec test fix** -- Kristofer Karlsson's patch to skip backslash pattern tests on Windows was queued, though Junio raised deeper questions about pathspec handling semantics.

**Worktree hook directory bug** -- A new bug report demonstrated how directory changes in secondary worktree hooks cause `git status` to incorrectly report all files as deleted.

### On the radar

**Delta-islands with path-walk** -- Taylor Blau's series enabling `--delta-islands` with `--path-walk` awaits performance metrics from Derrick Stolee's testing before final approval.

**Real-time collaboration proposal** -- Christian Couder's survey of alternative approaches (Jujutsu, Pijul, CRDTs) has shifted discussion toward integration patterns rather than core Git changes.