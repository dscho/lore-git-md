Here's the daily digest for March 21, 2026:

## The day in brief

March 21 saw moderate traffic with 38 emails across 16 threads, featuring several notable developments. The HTTP 429 rate limit handling series received final approval and is headed for 'next', while discussions around strbuf optimizations and remote-http segfaults revealed deeper architectural considerations. GSoC proposals continued to generate mentoring discussions.

## Notable threads

### HTTP 429 rate limit handling finalized

Vaidas Pilkauskas's HTTP 429 rate limit retry series received final approval from Taylor Blau and Junio Hamano after six iterations. The implementation adds configurable retry behavior (http.maxRetries, http.retryAfter, http.maxRetryTime) when servers return rate limit responses, using CURLINFO_RETRY_AFTER for curl >=7.66.0. Taylor confirmed the v6 restructuring addressed Jeff King's earlier feedback about parameter passing and error handling. Junio will merge the series to 'next', marking completion of this significant HTTP transport enhancement.

### strbuf_getwholeline() optimization debate continues

The detailed discussion about strbuf_getwholeline() optimizations took several new turns. René Scharfe identified a theoretical memory leak in edge cases, while Jeff King cautioned against extensive rewrites of this stable function. Junio proposed a refactoring that maintains the 7% performance improvement Peff demonstrated while eliminating NULL checks. The thread exemplifies Git's careful balancing of performance and maintainability in core primitives.

### Remote-http segfault exposes hash algorithm issue

Jo Liss reported and K Jayatheerth patched a segfault in git-remote-http when parsing refspecs outside a repository with global fetch configurations. Jeff King traced the root cause to hash algorithm initialization timing, noting the fix needs deeper consideration as Git prepares for SHA-256 default hashes. brian m. carlson emphasized using GIT_HASH_DEFAULT rather than hardcoding SHA-1, highlighting the architectural implications beyond the immediate crash.

### GSoC proposal refinement

Christian Couder and Junio Hamano provided feedback on Francesco Paparatto's GSoC proposal to migrate configuration variables from global to repository-scoped storage. Christian requested more precise documentation of prior work, while Junio suggested simplifying proposal requirements to focus on learning outcomes. The exchange shows Git's mentoring approach for complex refactoring projects.

### fetch --prune-tags behavior questioned

Junio raised important questions about Orgad Shaneh's patch to align `git fetch --prune-tags` behavior with its documentation, suggesting the current behavior might be intentional rather than a bug. He also noted missing test coverage for the pruning aspect of the feature. The discussion now needs historical context to determine whether to change the code or update the documentation.

## In brief

**Bisect terminology consistency** -- Jonas Rebmann's patch to make bisect honor custom terms (like "old/new") in output messages received approval from Junio, pending improved commit message and test coverage.

**git backfill argument validation** -- Siddharth Shrimali sent v2 of his patch to properly validate `git backfill` arguments, addressing Junio's feedback about error messaging and adding tests.

**Promisor file handling** -- Lorenzo Pegorari began a GSoC series to preserve .promisor file contents during repacks, starting with documentation and a helper function for consolidation.

**gitk maintenance** -- Johannes Sixt merged i18n improvements and color configuration updates for gitk, including standardized translation names and support for Git's color.decorate.* settings.

**git-gui updates** -- The git-gui maintainer merged UI tweaks (greyed commit comments, better diff alignment) and Meson build system improvements from multiple contributors.

## On the radar

**Geometric repacking edge case** -- Taylor Blau's series fixing MIDX bitmap generation with excluded packs needs to address a NULL pointer issue Jeff King identified in pack-objects handling.

**Test modernization** -- New contributor Aditya Indora is working on the standard test modernization microproject, converting `! test -f` checks as an onboarding task.