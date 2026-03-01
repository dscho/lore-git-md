# Git Mailing List Digest - 2025/02/18

**The day in brief.** A busy day with 103 emails across 24 threads, featuring significant progress on multiple fronts. Key highlights include the final approval of Patrick Steinhardt's reftable decoupling series and Christian Couder's promisor-remote capability, along with ongoing discussions about build system improvements and documentation fixes. The day also saw some heated exchanges around contributor expectations.

## Notable threads

**Reftable library decoupling complete**  
Patrick Steinhardt's 18-patch series to fully separate the reftable library from Git core dependencies received final approval from Junio Hamano after six iterations. The changes enable standalone use by projects like libgit2 by systematically replacing Git-specific utilities with reftable-native implementations. The v6 version addressed Windows build system conflicts through careful header reorganization while maintaining all functionality. This architectural change represents a significant step in Git's modularization efforts.

**Promisor-remote capability approved**  
Christian Couder's v5 patch series introducing protocol v2 capability for promisor remote advertisement was accepted after resolving cross-topic patch sharing complications from earlier versions. The implementation allows servers to advertise promisor remotes during clone/fetch operations, with configurable client acceptance policies ("KnownName" and "KnownUrl"). Junio noted the series is now ready to be queued, marking progress on the Large Object Promisors (LOP) feature that aims to improve large blob handling through dedicated object storage.

**Meson build system expansion**  
Patrick Steinhardt continued advancing Meson support with a 12-patch series wiring up contrib/ components including credential helpers, git-contacts, Coccinelle, and gitk. The changes follow established patterns for build system unification while handling platform-specific requirements. CI integration now verifies credential helpers across Windows (wincred), Linux (libsecret), and macOS (osxkeychain). The series leaves only git-gui, sparse, and perf tests as remaining omissions from Meson support.

**Documentation conflicts emerge**  
A routine documentation fix for `git-send-email` escalated into a heated exchange about contributor expectations. After Matěj Cepl corrected a typo in the man page (`--no-signed-off-cc` → `--no-signed-off-by-cc`), style feedback about commit message formatting led to accusations of gatekeeping and an announcement of contributor disengagement. The incident highlights ongoing tensions between maintaining project standards and welcoming new contributors.

**HTTP authentication regression reported**  
Peter Georg identified a regression in Git 2.46+ where `git-http-push` fails to properly use credentials from netrc files due to proactive authentication changes. The bug manifests differently based on `http.proactiveAuth` settings, either dropping credentials after initial use or ignoring netrc entirely. Brian m. carlson acknowledged the issue but noted fundamental incompatibilities between netrc's limitations and proactiveAuth's design goals, suggesting git-credential-netrc as a workaround while a proper solution is investigated.

## In brief

The OS version advertisement feature series received final approval after switching to dash-separated format ("git/1.8.3.1-Linux"). Perl version requirements were standardized to 5.26.0 across build systems with documentation refinements for Meson edge cases. `git merge-tree --stdin` improvements finalized with deadlock fixes, config scope tightening, and documentation clarifications about exit status behavior. A pickaxe (-G/-S) bugfix will now reject empty strings with proper errors rather than crashing. Documentation terminology was standardized between `git-commit.txt` and `git-rebase.txt` regarding commit message "subjects" versus "titles".

## On the radar

The discussion about `format.from` configuration affecting `git show --format=email` continues, with Junio Hamano maintaining that these settings were designed specifically for `format-patch` despite some config sharing in the code. The submodule merge behavior thread explores documentation placement challenges for behavior affecting all merge machinery users (rebase, cherry-pick etc.), not just `git merge`. Performance optimization work on refname availability checks shows promising 1.15x-5x speedups but faces review questions about iterator lifecycle semantics.