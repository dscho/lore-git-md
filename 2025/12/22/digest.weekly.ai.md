# Git Mailing List Weekly Digest — 2025/12/22 -- 2025/12/28

**The week in brief.** A busy final week of 2025 with 196 emails across 66 threads saw significant progress on several major fronts. The hook subsystem refactoring reached completion after years of work, while debates continued around `git-history`'s branch selection and `status.goalBranch` configuration. Junio Hamano's "What's cooking" report highlighted Windows symlink support and ODB improvements as key pipeline items. Performance discussions around `pack-refs` and batched reference updates revealed deeper architectural considerations, while new contributors began engaging for GSoC 2026 season.

## Key developments

### Hook subsystem modernization completes

Adrian Ratiu's 11-part series refactoring Git's hook infrastructure reached its final form (v6) and received Junio's approval for merging. This multi-year effort standardizes hook execution on a new `hook.h` API, introducing batched stdin processing (500 lines per callback), sideband output preservation, and parallel execution support while maintaining backward compatibility. The changes touch all major hooks including pre-push, pre-receive, and post-rewrite, eliminating direct `find_hook()` calls throughout the codebase. With sign-offs from Ævar Arnfjörð Bjarmason and Emily Shaffer, this foundational work paves the way for future config-based hook management.

### `git status` comparison feature matures

Harald Nordgren's `status.goalBranch` series progressed through nine iterations (v1-v9) to add configurable branch comparison output. The feature shows divergence from a secondary branch (like "upstream/main") alongside normal tracking info, addressing triangular workflow needs. Technical implementation stabilized with comprehensive test coverage, but Junio questioned whether the feature justifies a new config variable rather than leveraging existing tracking. The discussion revealed fundamental tensions between explicit configuration and implicit convention in Git's design philosophy.

### Batched reference error reporting regression

A push error reporting regression (detailed "non-commit object" messages replaced with generics) was traced to batched reference updates introduced in Git 2.51. Jeff King and Karthik Nayak designed a fix attaching error details to individual failed ref updates rather than whole transactions. The solution modifies `ref_transaction_maybe_set_rejected()` to store details in a new `rejection_details` field, addressing issues across push, fetch, and update-ref operations. The discussion highlighted how batched updates inadvertently simplified error reporting that tools had come to depend on.

### `pack-refs` performance investigation

Performance analysis revealed `git pack-refs` suffers from unnecessary object header reads when peeling tags, particularly impacting repositories with millions of refs (like Gerrit servers). Brian m. carlson noted the reftable backend might avoid this overhead by maintaining pre-peeled references, suggesting long-term architectural advantages for high-refcount scenarios. The discussion exposed tradeoffs between Git's current files-based ref storage and reftable's design for scale.

### Standardized commit array handling

René Scharfe completed a 14-part series converting Git's ad-hoc commit pointer arrays to a shared `commit_stack` API. The changes touch core subsystems including revision walking, logging, and bitmap generation, demonstrating significant code reduction (184 insertions vs 247 deletions). The standardized interface now handles commit collections consistently across performance-sensitive areas while maintaining the same memory efficiency as specialized implementations it replaces.

## In brief

**`git-history` branch selection** -- Phillip Wood and D. Ben Knoble challenged Patrick Steinhardt's proposed branch walking strategy, noting workflow friction with stacked branches and questioning performance assumptions about ref distributions.

**PID-based lockfile debugging** -- A feature adding `~pid.lock` companion files for debugging stale locks progressed through v3, with Junio requesting documentation and parameter design refinements.

**SGID bit preservation** -- Hadmut Danisch reported Git clears SGID bits (mode 2770) during operations, breaking container workflows relying on automatic group inheritance, sparking debate about permission handling philosophy.

**Windows MSI packaging** -- A user inquiry highlighted the lack of MSI packages for enterprise deployment via Active Directory Group Policy, revealing a gap in Git for Windows' organizational deployment options.

**New contributor onboarding** -- Three potential GSoC 2026 participants (Andrew Chitester, Deveshi Dwivedi, Ayush Jain) began engaging, directed to microprojects and the Hacking-Git guide by Christian Couder.

**`git branch --show-current`** -- The command's silent ignoring of `-v` was deemed buggy and will now error explicitly after Junio reversed his initial position that the behavior was intentional.

**macOS iconv fixes** -- René Scharfe simplified Homebrew integration for character encoding issues, removing unnecessary component flags while maintaining version-specific activation.

## Looking ahead

The new year will likely see movement on several fronts: Karthik Nayak's batched reference error fixes, continued debate around Rustification and platform support concerns, and potential follow-ups to the completed hook subsystem work. The `status.goalBranch` discussion may pivot toward broader configuration management improvements, while performance optimizations for high-refcount repositories could gain urgency. With GSoC 2026 applications opening soon, expect increased activity around mentorship and project scoping discussions.