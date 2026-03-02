# Git Mailing List Digest — 2025/03/16

## The day in brief  

A moderately active Sunday with 39 emails across 20 threads, featuring steady progress on several fronts. Notable developments include finalization of safer assertion handling in Git's core, resolution of a Windows-specific keyboard lockup bug, and continued discussion about directory rename tracking limitations. The day saw multiple patch series reach their final iterations, with several likely headed for inclusion.

## Notable threads  

### **Safer assertion handling reaches consensus**  
Elijah Newren's 3-part series introducing `BUG_IF_NOT()` as a safer alternative to `assert()` has cleared its final hurdles. After resolving licensing concerns about the CI detection mechanism (a clever compiler trick now confirmed public domain), the series adds runtime-checked assertions for merge machinery, object storage, and parallel checkout. The patches systematically replace risky assertions containing function calls, ensuring they remain active in production builds. Junio Hamano and brian m. carlson have reviewed the approach, which will help prevent subtle bugs from disappearing in release builds.

### **Windows keyboard lockup fix confirmed**  
Johannes Schindelin identified the root cause of a Git for Windows issue where `git add --patch`'s "e" edit option could render the keyboard unresponsive. The fix is slated for v2.49.0, delayed from Friday per the project's no-Friday-release policy. This resolves a workflow-halting bug that required terminating Git Bash when editing patches, particularly affecting users of the interactive staging interface.

### **Directory rename tracking limitations examined**  
A deep dive into Git's path-filtered log behavior revealed fundamental constraints in handling subtree merges. Johannes Schindelin and Junio Hamano analyzed why `--follow` fails to track directory renames across merge points, identifying core architectural limitations in `tree-diff.c`. The discussion suggests proper support would require restructuring revision walking to maintain per-ancestry path tracking—a non-trivial change with potential performance implications. This technical exploration clarifies why Git currently silently ignores directory arguments to `--follow`.

### **SMTP error handling refined**  
Zheng Yuting's GSoC work on SMTP authentication reached v4, now properly split into two clean patches after rebasing issues. The series improves `git-send-email.perl`'s error handling by distinguishing temporary (4xx) from permanent (5xx) SMTP errors and consolidating exception capture. The changes make the authentication flow more robust while maintaining the existing structure reviewers requested. With the version control issues resolved, this iteration appears ready for final review.

## In brief  

Gitk gains a prefetch ref hiding option in v3, addressing visual clutter in repositories with maintenance tasks. Documentation updates correct outdated notes about `git restore -p` pathspec support (since Git 2.35) and fix an ancestry graph example. A Meson build fix ensures Perl is properly required for documentation generation.  

The merge-recursive removal effort saw five preparatory cleanups: removing obsolete includes, fixing test comments, making diff algorithm tests explicit, correcting a strset/strintmap usage, and polishing merge-ort documentation.  

Cygwin regex test anomalies resurfaced in Ramsay Jones' compatibility series, revealing a 12-year-old test case that unexpectedly passes due to platform-specific regex behavior.  

## On the radar  

The CSPRNG backend discussion concluded that Linux should prefer `getrandom` over `arc4random`, as glibc's implementation provides no benefits over direct syscalls. This consensus may prompt follow-up patches to adjust build system defaults.  

Johannes Sixt is considering dropping a UI label from git-gui's comment character support after reaching agreement on the core functionality. This minor interface tweak appears to be the last open question in that thread.