# Git Mailing List Digest - 2025/12/31

**The day in brief.** A moderately active New Year's Eve with 10 emails across 7 threads, featuring performance investigations, fsmonitor improvements, and test reliability fixes. The standout items are the Linux fsmonitor backend reaching v4 and an ongoing debate about `git pack-refs` performance compared to JGit.

## Notable threads

**Linux fsmonitor backend reaches v4** -- The long-running effort to bring filesystem monitoring to Linux via inotify has reached its fourth iteration, with Paul Tarjan addressing all previous feedback except for two small memory leaks. This implementation matches the existing Windows and macOS backends, enabling faster status operations by watching filesystem events rather than scanning the working directory. The patch includes recursive directory watching, special handling for renames, and detection of remote filesystems. With comprehensive test coverage and multiple rounds of review, this appears ready for merging once the remaining leaks are addressed.

**Pack-refs performance investigation continues** -- Martin Fick challenges the NFS latency explanation for Git's poor `pack-refs` performance compared to JGit, noting that JGit handles the same environment with 37 pack files much faster (20s vs Git's 5+ minutes). The discussion has shifted focus to potential write-path inefficiencies in Git's implementation, though no concrete optimization proposals have emerged yet. This thread continues to refine understanding of the performance gap between Git's and JGit's ref handling, with the reftable backend remaining a potential architectural solution.

**Format-patch merge commit warnings debated** -- Dominique Martinet proposes adding warnings when `git format-patch` skips merge commits, addressing a user experience issue reported by `jj` users. Junio Hamano responds that warnings should be context-sensitive - helpful when no commits are formatted but potentially noisy in merge-heavy workflows. This RFC thread now has two clear positions to reconcile: making merge-skipping more visible versus avoiding spam in common scenarios.

## In brief

**Fsmonitor daemon memory leak fix** -- Paul Tarjan sends v2 of a patch fixing a 40-byte per-request leak in the fsmonitor daemon's client handling code, properly cleaning up a khash table used for pathname deduplication.

**Nixpkgs test suite fix** -- Adam Dinwoodie advocates for including Brian Lyles' minimal fix for `gui--askyesno` test failures in nixpkgs builds, adding the command to the expected-failure list as a stopgap measure for Git v2.52.0.

**Difftool test race condition** -- Johannes Schindelin fixes a flaky Windows CI failure in the difftool test suite by modifying test content to ensure reliable change detection, avoiding platform-specific filesystem quirks.

**Fsmonitor test reliability** -- A new patch introduces a `retry_grep()` helper to address race conditions in fsmonitor event tests, replacing the previous approach of using `test-tool fsmonitor-client query` as a delay mechanism.