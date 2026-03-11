# Git Mailing List Digest - March 7, 2026

**The day in brief.** A moderately active day with 39 emails across 15 threads, featuring significant progress on several fronts. Key developments include finalization of configurable partial clone filters, a major line-log improvement series, and ongoing discussions about git-gui maintenance behavior. Junio's "What's cooking" report shows steady progress across multiple topics.

## Notable threads

**Line-log improvements land** -- Michael Montalbo's four-patch series to route `git log -L` output through Git's standard diff pipeline has completed review and been merged. This enables previously unsupported features like `--word-diff`, `--color-moved`, and pickaxe options (`-S`, `-G`) to work with line ranges. The changes maintain backward compatibility while eliminating the hand-rolled `dump_diff_hacky()` function in favor of `builtin_diff()`. Junio expressed enthusiasm for the direction, though some limitations remain (non-patch diff formats still don't work with `-L`).

**Partial clone filters finalized** -- Alan Braithwaite's work on configurable partial clone filters via URL patterns (`clone.<url>.defaultObjectFilter`) has reached its final iteration (v5). The feature allows automatic filter application during clone operations based on URL matching, with command-line overrides. All substantive feedback has been addressed, leaving only minor documentation and test style nits to polish. The implementation uses Git's existing urlmatch infrastructure and maintains clear boundaries by being clone-specific.

**Memory leak investigations continue** -- Jeff King's series fixing mmap-related memory leaks has uncovered a new issue in the object streaming code during final testing. Junio bisected a CI failure to Patrick Steinhardt's earlier commit 595296e ("streaming: rely on object sources to create object stream"), where error handling incorrectly tried to munmap() an unpopulated struct field. Jeff provided a straightforward fix that munmaps the local variable instead. The thread demonstrates Git's rigorous attention to resource management, particularly around error paths.

**git-gui maintenance behavior debated** -- Discussion continues about modernizing git-gui's repository maintenance by integrating with Git's built-in auto-maintenance feature. Johannes Sixt and Junio debated how to interpret `gui.gcwarning=false` - whether it should disable all automatic maintenance (current behavior) or just suppress warnings. They converged on respecting it as a comprehensive opt-out, meaning the patch will need revision to check this config before running maintenance. The resolution balances modernization with backward compatibility.

**Remote group push proposal refined** -- Usman Akinyemi's RFC for remote group support in `git push` (mirroring existing `fetch` functionality) received design feedback from Junio. The maintainer suggested treating single remotes as a one-member group case rather than maintaining separate code paths, simplifying future maintenance. The core concept appears sound but needs implementation refinement to unify the handling logic. This suggests constructive trajectory for the proposal despite the structural changes needed.

## In brief

**Shallow clone bugfix review** -- Samo Pogačnik followed up on his v3 patch fixing `--shallow-since` edge cases, reaching out to domain experts (Patrick Steinhardt, Taylor Blau, Johannes Schindelin) as requested by Junio. The patch has addressed all terminology feedback and awaits specialized review.

**Branch prefixing feature questioned** -- Eric Sunshine raised fundamental concerns about Yoann Valeri's branch name prefixing feature, questioning whether it belongs in core Git and suggesting a more flexible string interpolation design might be preferable.

**Test modernization continues** -- Francesco Paparatto's v4 patch modernizing `t3310-notes-merge-manual-resolve.sh` completed the series, replacing fragile `rev-parse` command substitutions with robust test helpers after thorough review from Eric Sunshine.

**GSoC proposal review** -- Christian Couder provided feedback on Shreyansh Paliwal's GSoC proposal to reduce global state dependencies, correcting technical details about merged commits while affirming the project's alignment with Git's architectural priorities.

**Slow fetch investigation** -- R. Diez provided detailed timing data about slow `git fetch` over SMB/CIFS, showing 23.7 of 24 seconds spent in `git-upload-pack`, with particular delays in reference resolution.

**What's cooking report** -- Junio's status update showed Patrick Steinhardt's ODB abstraction series progressing to 'next', sideband sanitization work being split into near-term and post-3.0 efforts, and various performance improvements in flight.

## On the radar

**Rustification effort** -- While not active today, Ezekiel Newren's work to introduce Rust code into Git remains a long-term effort with ongoing platform support concerns, particularly from Randall S. Becker regarding NonStop compatibility.

**Hook configuration** -- Adrian Ratiu's work on config-based hooks continues to progress in Junio's integration branches, with parallel execution support still under review.