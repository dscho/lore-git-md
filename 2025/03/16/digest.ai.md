# Git Mailing List Digest — 2025/03/16

## The day in brief

A moderately active Sunday with 39 emails across 20 threads, featuring several technical discussions about assertion safety, cryptographic random number generation, and merge machinery cleanups. Notable developments include a finalized series for safer assertions in Git's codebase and an in-depth discussion about directory rename tracking limitations in `git log --follow`.

## Notable threads

### **Safer assertion handling introduced**

Elijah Newren's 3-part series introducing safer alternatives to `assert()` reached its final form today. The patches:
1. Add a new `BUG_IF_NOT()` macro that remains active in production builds
2. Implement CI detection of assertions with potential side effects using a clever (now public domain) compiler trick
3. Convert 9 flagged assertions across merge machinery and object storage subsystems

The series addresses a subtle hazard where assertions containing function calls could be silently removed in production builds. With licensing concerns resolved and technical validation from multiple reviewers, this appears ready for merging.

### **CSPRNG backend discussion concludes**

A technical discussion about cryptographic random number generation backends reached consensus today. The thread established that:
- Linux's `arc4random` is just a `getrandom()` wrapper with no security benefits
- BSD's implementation (using ChaCha20) remains the preferred choice on BSD systems
- Git should default to `getrandom` on Linux where available

Junio Hamano proposed documentation updates to clarify this, while noting the meson build system still needs correction. The discussion revealed an interesting historical detail - glibc briefly included a true `arc4random` in 2022 before reverting to syscall wrapping after kernel maintainer objections.

### **Directory rename tracking limitations analyzed**

Johannes Schindelin and Junio Hamano dug into why `git log --follow` fails to track directory renames in subtree-merged histories. The analysis revealed fundamental limitations in Git's revision walking logic:

1. Current single-path queue access (`q->queue[0]`) can't handle merges where paths rename differently in parallel histories
2. A proper solution would require per-ancestor path tracking during revision walks
3. Directory-level following would emerge naturally from a more general multiple-path solution

Hamano noted this isn't a superficial fix - it would require architectural changes to core revision walking behavior. The discussion positions directory-following as part of a larger path-tracking challenge in Git's revision subsystem.

## In brief

**gitk improvements**: A v3 patch adds prefetch ref hiding to gitk's preferences, following the pattern for remote ref hiding. Johannes Sixt also reviewed a v3 patch fixing gitk's external diff handling for renamed files.

**Documentation fixes**: Han Jiang corrected an ancestry graph example in `rev-list-options.adoc`, while another patch updated outdated information about `git restore -p` pathspec support (since Git 2.35).

**Windows-specific fixes**: Johannes Schindelin confirmed a keyboard unresponsiveness bug in `git add -p`'s edit mode is fixed for Git for Windows v2.49.0.

**Merge-recursive cleanups**: Elijah Newren sent a 5-patch series of preparatory refactorings including test comment fixes, include cleanup, and merge-ort improvements.

**SMTP auth handling**: Zheng Yuting's v4 series refactoring SMTP error handling in `git-send-email.perl` was properly rebased and restructured per Junio's feedback.

**Build system fixes**: A patch corrected Perl detection in Meson builds when documentation is enabled but Perl bindings are disabled.

## On the radar

The assertion safety series appears ready for merging after resolving licensing questions. The CSPRNG discussion may prompt follow-up patches to adjust meson configuration. The directory rename tracking analysis suggests deeper architectural work may be needed in Git's revision walking code.