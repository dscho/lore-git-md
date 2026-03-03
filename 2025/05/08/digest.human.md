Here's the Git mailing list daily digest for May 8, 2025:

## The day in brief
May 8 saw lively discussion across 25 active threads with 87 emails. The day was dominated by technical refinements to several major features, including final polishing of the `git send-email` OAuth documentation, continued debate about `git-blame-tree` integration, and performance optimizations for packed-refs and promisor packs. Junio's "What's cooking" report provided a comprehensive snapshot of current development activity.

## Notable threads

### Stricter FQDN validation and OAuth docs for send-email
Aditya Garg's series improving `git send-email` reached its final form after seven iterations, with Junio Hamano queuing v7 for merging. The changes include RFC1035-compliant domain validation and comprehensive OAuth2.0 documentation for Gmail and Outlook. The thread included extensive discussion about documentation formatting standards, particularly around backtick usage for paths versus commands. The series demonstrates Git's attention to both technical correctness (with a refined regex for domain validation) and user experience (clear OAuth configuration examples).

### Debate continues on git-blame-tree integration
The discussion about whether to integrate directory-level blame functionality into `git blame` or keep it as a separate command saw significant activity. Marc Branchaud strongly advocated for integration based on discoverability concerns, while Junio Hamano clarified key behavioral constraints - notably that `git blame path/to/file` must maintain its current line-by-line annotation behavior. The thread explored UI design questions including recursion defaults and the relationship to `git log -1`. D. Ben Knoble suggested a potential middle ground following the `git diff-tree` plumbing/public split pattern.

### Memory optimizations for packed-refs
A completed series optimizing memory usage in the packed-refs backend received final review attention. Jeff King and Junio Hamano discussed subtle mmap strategy considerations, particularly around when to use direct mmap/verify/unmap versus temporary snapshots during fsck operations. The exchange highlighted Git's careful attention to memory management patterns, even in performance optimization work. The series had already demonstrated significant improvements - reducing processing time from 67.6s to 18.8s for a 2.7GB promisor pack.

### Performance boost for promisor pack handling
Han Young contributed a performance optimization for `git repack` that skips enumeration of objects in kept packs when repacking promisor objects. The change reduced processing time by 72% (from 67.6s to 18.8s) in test cases while also producing smaller output packs. Junio requested review from Christian Couder as a promisor area expert, indicating the patch is nearing readiness for integration. The optimization builds on earlier work by Tomáš Trnka and demonstrates Git's ongoing focus on large-repository performance.

### Stash import/export as refs
A v5 series from Lidong Yan introduced the ability to import and export stashes as refs, enabling stash transfer between repositories via push/fetch/bundle. The implementation uses commit chains with two parents - one for the stash sequence and one for the stash data. The series includes thorough test coverage and addresses memory handling issues found in earlier iterations. This solves a long-standing workflow limitation where stashes couldn't be shared between repositories.

## In brief

**Commit message attribution debate** -- Jeff King weighed in on the bracketed annotation vs formal trailer discussion, pragmatically accepting the bracketed style despite theoretical advantages to trailers.

**gitk SHA-256 support** -- Johannes Sixt reviewed Takashi Iwai's implementation, focusing on code quality issues like variable handling and test coverage.

**Memory leak fixes** -- Multiple patches addressed leaks: in email header decoding (Lidong Yan), reftable array reallocation (though René Scharfe noted the macro may be unused), and commit-graph pack handling.

**Build system improvements** -- Ramsay Jones' series fixed path handling across Make, Meson and Autoconf builds, including an important sysinfo detection fix validated on Solaris.

**Sparse index integration** -- Derrick Stolee's series making `git add -p/-i` and `git apply` sparse-index aware received positive review from Junio, with tests showing 96.7% runtime improvements.

**Windows CI fixes** -- Junio confirmed combining Meson Release mode and `--vsenv` flag fixes for Visual Studio builds in the maintenance branch.

## On the radar

**Autoconf future** -- Eli Schwartz raised questions about autoconf's ongoing maintenance value as Meson adoption grows, though Solaris validation of recent fixes shows some need remains.

**Backup strategies** -- The thread exploring Git bundle-based backups uncovered a timestamp vulnerability with `--since` and is now evaluating state-tracking alternatives to time-based approaches.