# Git Mailing List Digest - 2026/04/30 (Thursday)

**The day in brief.** A moderately busy Thursday with 46 emails across 13 threads, featuring the completion of several significant technical efforts. The standout developments include Taylor Blau's finalized incremental MIDX repacking series, Ezekiel Newren's xdiff refactoring reaching merge readiness, and a new feature for diff statistics in log formats. Meanwhile, discussions continue around HTTP authentication fixes and a `git fetch --deepen` regression.

## Notable threads

### **Incremental MIDX repacking finalized**  
Taylor Blau completes his 16-patch series implementing incremental MIDX/bitmap-based repacking for large repositories. The final version introduces two operational modes: geometric compaction (merging layers when conditions are met) and append-only (preserving all prior layers). Key improvements in this iteration include replacing custom pipe handling with `pipe_command()` to eliminate deadlock potential, adding a `--base` option for custom layer specification, and converting manual string arrays to `strvec` for safer memory management. The series has received thorough review from both object storage (Elijah Newren) and process safety (Jeff King) experts, marking a major milestone in Git's scalability for large repositories.

### **xdiff refactoring ready for merge**  
Ezekiel Newren's xdiff refactoring series reaches completion after incorporating all of Phillip Wood's review feedback. The v6 patches finalize type safety and clarity improvements in xdiff's record cleanup logic while maintaining all non-regressing improvements from previous versions. Notable changes include explicit action execution logic in the final patch, improved limit calculation clarity, and replacement of nested ternary operators with explicit if/else ladders. Phillip Wood has verified the implementation addresses his concerns, particularly around INVESTIGATE handling and action execution flow. The thread demonstrates a model review cycle with substantive technical engagement, now awaiting final integration by Junio.

### **New diff statistics placeholders**  
A new feature patch introduces pretty-format placeholders for diff statistics in Git logs. The change adds four new placeholders (`%(diff-stat:files)`, `%(diff-stat:insertions)`, etc.) with short aliases (`%aF`, `%aA`, `%aR`), reusing Git's existing diffstat machinery while caching results to avoid performance overhead. The implementation includes thorough documentation updates and test coverage verifying behavior matches `--shortstat` output across various scenarios. This enables including change statistics in custom log formats without parsing `--shortstat` output manually, addressing a long-standing workflow need.

### **HTTP authentication fixes complete**  
Matthew John Cheetham adds a documentation patch completing the HTTP authentication bugfix series. The patch explicitly documents the three valid values for `http.emptyAuth` (`auto`, `true`, `false`) with clear descriptions of each mode's authentication behavior, matching the technical implementation from earlier patches. With this addition, the series addressing Kerberos (SPNEGO) authentication issues when using `http.emptyAuth=auto` is now complete - the technical fixes are in place, test coverage exists, and documentation accurately reflects the behavior.

### **`git fetch --deepen` regression discussion**  
The discussion around a `git fetch --deepen` regression in Git 2.54.0 continues, with Mikael Magnusson arguing strongly that the command should be a no-op in non-shallow repositories. He contends that allowing `--deepen` to reset the shallow boundary would lead to nonsensical behavior where a fully unshallow repository could revert to being shallow, contradicting user expectations. The thread is moving toward consensus that the 2.53.0 behavior (no-op in non-shallow repos) is correct regardless of documentation wording, though no maintainer response or proposed fix has appeared yet.

## In brief

**`git format-rev` bugfixes** -- Kristoffer Haugsbakk addresses a CI-caught compiler warning about potential uninitialized variable use in the new experimental `git format-rev` command, along with an error message correction pointed out by a reviewer.

**GitHub Actions updates** -- Johannes Schindelin completes a 6-patch series updating GitHub Actions dependencies across all workflows, including updates to setup-msbuild, artifact handling, checkout, and Git for Windows SDK setup actions to maintain compatibility with GitHub's platform changes.

**git-gui repository handling** -- Shroom Moo's v3 patch for git-gui's bare repository handling addresses an edge case with `--separate-git-dir` configurations, though Mark Levedahl raises deeper concerns about complex repository/worktree relationships that may require further consideration.

**Windows large object fixes** -- Torsten Bögershausen notes Johannes Schindelin's patch series addressing Windows platform limitations with large objects (>4GB) makes an existing GitHub pull request obsolete, though the review doesn't engage with technical details.

**`git rebase -i` behavior clarified** -- Phillip Wood confirms that sequential processing of reworded commits in interactive rebase is the intended behavior, not a regression, allowing inspection of changes for each commit being edited.

## On the radar

**Reftable backend refactoring** -- Karthik Nayak's refs backend consolidation series awaits a revision to address error handling feedback from Toon Claes, with the core architectural changes already reviewed and approved.

**Git Rev News draft** -- Christian Couder shares the draft of Git Rev News edition 134 for community review before its May 2 publication, inviting contributions to improve coverage of recent mailing list activity.