# Git Mailing List Digest - 2025/12/31

**The day in brief.** A moderately active New Year's Eve with 10 emails across 7 threads, featuring ongoing performance investigations, fsmonitor improvements, and test stability fixes. The most notable developments include a new Linux fsmonitor backend reaching maturity and continued discussion about merge commit handling in `git format-patch`.

## Notable threads

**Linux fsmonitor backend reaches v4** -- The proposed Linux implementation of Git's filesystem monitoring (fsmonitor) using inotify has progressed to its fourth iteration, addressing all previously identified issues except for two small memory leaks. This brings Linux implementation to parity with existing Windows and macOS backends, enabling faster status operations through filesystem event watching rather than full directory scans. The implementation handles recursive directory watching, directory renames, and remote filesystem detection, with extensive stress testing confirming its robustness. The remaining leaks (512-byte during startup and 40-byte in IPC handling) are the final blockers before this long-running effort can be considered ready for merging.

**Memory leak fixes for fsmonitor daemon** -- A related thread sees progress on fixing those memory leaks, with Paul Tarjan's v2 patch properly addressing a 40-byte per-request leak in the fsmonitor daemon's client handling code by correcting khash table cleanup. René Scharfe's review confirmed the technical approach while raising questions about cleanup during interruption scenarios, leading to a refined v2 that focuses specifically on the clear memory leak without overclaiming about interruption handling. The changes are minimal but important for the daemon's stability.

**`git format-patch` merge commit warnings** -- Dominique Martinet proposes an RFC patch to make `git format-patch` warn when skipping merge commits, addressing user confusion in merge-heavy workflows like those used with `jj`. Junio Hamano's response suggests the warnings should be context-sensitive - helpful when no commits are formatted at all, but potentially noisy in common workflows that intentionally include merge-heavy ranges. This sets up an interesting design question for future iterations: how to make the behavior visible when needed without annoying established users.

## In brief

**Nixpkgs test fix** -- Adam Dinwoodie advocates for including Brian Lyles' minimal fix for `gui--askyesno` test failures in nixpkgs builds as a stopgap measure while longer-term solutions are discussed. The patch simply adds the command to an expected-failure list to unblock builds for Git v2.52.0.

**Racy difftool test fix** -- A Windows-specific test flakiness in the difftool suite is addressed by modifying test content to ensure reliable change detection, avoiding platform-specific filesystem quirks around same-size file replacements.

**Fsmonitor test stability** -- Race conditions in fsmonitor event tests are fixed by introducing a `retry_grep()` helper that waits for events to appear in trace files, replacing a less reliable delay mechanism.

## On the radar

**`pack-refs` performance investigation** -- Martin Fick continues pushing back on NFS latency as the sole explanation for Git's poor `pack-refs` performance compared to JGit, suggesting write-path inefficiencies may be a factor. This long-running investigation remains active but without clear optimization paths yet.