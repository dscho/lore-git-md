# Git Mailing List Digest — 2026/01/03

## The day in brief

A moderately busy day with 19 emails across 6 threads, dominated by the conclusion of a long-running `git status` push-tracking feature and several bugfix discussions. The push-tracking series reached its final form after 14 iterations, while test fixes and encoding issues also drew attention.

## Notable threads

### **`git status` push-tracking feature finalized**

After 14 iterations spanning two weeks, Harald Nordgren's feature to show push-tracking branch divergence in `git status` output has reached its final form (v14). The series now properly handles custom refspec transformations through a full resolution chain that maps local branches through push refspecs to determine remote destinations, then back through fetch refspecs to find tracking branches. Phillip Wood provided key technical guidance on the refspec implementation, while Junio Hamano had earlier approved the technical direction.

The implementation shows both upstream and push tracking comparisons when they differ, with output like:
```
On branch feature
Your branch is ahead of 'upstream/main' by 1 commit.
Your branch is ahead of 'origin/feature' by 1 commit.
  (use "git push" to publish your local commits)
```

With 210 lines of new test coverage and all edge cases addressed, this appears ready for inclusion pending Phillip Wood's final sign-off on the refspec implementation details.

### **Windows test fix sparks filesystem discussion**

A minimal fix for a racy test in the difftool test suite (`t7800-difftool.sh`) evolved into a deeper discussion about Windows filesystem behavior. The original patch modified test content to ensure different file sizes (17 bytes instead of 12) to work around Windows' unreliable stat detection for same-size files. While Johannes Schindelin reviewed and approved the test fix, Phillip Wood and Paul Tarjan's follow-up discussion revealed this only addresses the symptom - the underlying issue with difftool's temporary index and same-size modifications on Windows remains unresolved for scripted use cases.

### **`git status -z` path formatting inconsistency reported**

Artur Pyrogovskyi reported and Pushkar Singh confirmed an inconsistency in `git status` output formatting when combining `-z` with `status.relativePaths`. Jeff King (Peff) analyzed the behavior, identifying two distinct bugs: porcelain v1 with `-z` intentionally bypasses config (historical behavior), while porcelain v2 with `-z` and `--short -z` exhibit inconsistent behavior. The discussion now awaits input from status.c experts to clarify intended semantics before proceeding with fixes.

## In brief

**Reftable iterator fix** -- Pushkar Singh reviewed Tsahi Elkayam's patch fixing undefined behavior in the indexed table reference iterator by properly validating `value_type` before accessing union members.

**Format-patch merge commit warnings** -- A refined RFC patch proposes adding warnings about skipped merge commits in `git format-patch`, now only showing when no patches are generated and at least one merge was skipped.

**Gitignore` encoding improvements** -- A new patch addresses encoding issues in `.gitignore` handling, particularly improving support for UTF-16LE files with BOMs commonly created by Windows PowerShell.

## On the radar

**Refspec edge cases** -- The push-tracking feature's final refinement around custom refspec transformations may warrant close attention as it gets its final review.