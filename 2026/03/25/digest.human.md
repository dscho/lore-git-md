Here's the Git mailing list digest for March 25, 2026:

## The day in brief

March 25 saw significant activity across multiple fronts in Git development, with 139 emails across 25 active threads. The day was dominated by several major patch series reaching maturity, including the long-awaited `git replay --revert` feature, parallel hook execution, and remote group support for `git push`. Junio Hamano's "What's cooking" report provided a comprehensive snapshot of the project's current state, while test infrastructure improvements and ongoing `the_repository` removal work continued steadily.

## Notable threads

**`git replay --revert` reaches v6**  
Siddharth Asthana's series adding revert capability to `git replay` for server-side use cases like GitLab's Gitaly service is now ready for merging after six iterations. The final v6 addresses minor compiler warnings and documentation nits while maintaining the core functionality that treats reverts as merges with swapped arguments. The implementation follows established patterns from `sequencer.c` and includes comprehensive test coverage. Reviewers Phillip Wood, Patrick Steinhardt, and Junio Hamano have all signed off on the technical approach, with the series now awaiting final integration.

**Parallel hook execution nears completion**  
Adrian Ratiu's series enabling parallel hook execution has been merged to 'next' with post-merge refinements in progress. The latest discussion focuses on memory management design and API organization, with Patrick Steinhardt approving Adrian's preferred string-list-based cleanup approach. Junio noted a minor naming quirk between `struct hook` and `struct repository` fields that could be addressed in future cleanup work. This represents the final step in Adrian's multi-series effort to modernize Git's hook configuration system.

**Remote group support for `git push`**  
Usman Akinyemi's feature allowing `git push` to target remote groups (mirroring existing `git fetch` functionality) is now functionally complete after addressing all maintainer feedback. The implementation handles each remote in the group sequentially while respecting individual configurations, with failure handling that continues pushing to remaining remotes after any failure. Junio has approved the technical approach and the series appears ready for merging, though parallel push support remains a potential future enhancement.

**Test infrastructure hardening**  
Junio Hamano led a major effort to make Git's test suite more robust by enabling "set -e" (exit immediately on error) early in test execution. The 11-patch series fixes various test scripts to work with stricter error checking, catching mistakes like misspelled test directives that currently fail silently. Patrick Steinhardt provided constructive feedback suggesting more idiomatic shell constructs in several cases. The changes are purely mechanical but significantly improve the test framework's error detection capabilities.

**`the_repository` removal continues**  
Shreyansh Paliwal sent a 5-patch series eliminating global state usage in the refs subsystem as part of the ongoing `the_repository` removal effort. The changes make various helper functions explicitly take a `struct repository` parameter, systematically tackling different aspects of the refs implementation. While these are mechanical conversions rather than functional changes, they represent important progress in making Git's codebase more explicit about repository context.

## In brief

**Linux fsmonitor status check** -- Junio followed up on Patrick Steinhardt's process concerns about the Linux fsmonitor implementation series, noting the thread has gone quiet since questions were raised about submission cadence and AI involvement.

**xdiff refactoring v2** -- Ezekiel Newren sent a significantly narrowed refactoring series for xdiff's record cleanup logic, focusing on readability improvements in `xdl_cleanup_records()` after dropping more controversial structural changes from v1.

**GSoC proposal: "un-fetch" for partial clones** -- Siddharth Shrimali proposed adding functionality to move blobs back to promised-but-not-present state in partial clones, with careful attention to safety mechanisms and integration as a git maintenance task.

**Graph lane limiting v5** -- Pablo Sabater's GSoC project to add `--graph-lane-limit` to `git log --graph` progressed to v5 with improved documentation and test coverage, though Junio identified some implementation details needing final polish.

**`git replay` root commit documentation** -- Christian Couder and Ben Knoble discussed improving documentation about commit range syntax in `git replay` to prevent accidental root commit replay, with Junio suggesting more nuanced explanation of valid use cases.

**Fast-import signature handling** -- Justin Tobler responded to Junio's feedback about making invalid signature mode checking consistent between `fast-import` and `fast-export`, planning to move validation earlier in `fast-export`'s option parsing.

**Promisor file handling in repacks** -- Lorenzo Pegorari addressed Junio's documentation feedback in his GSoC series about promisor file handling during repacks, agreeing to reference commit 5374a290 and fix comment formatting.

## On the radar

**`git add -p` discard hunk proposal** -- Luiz Campos' RFC for adding a discard hunk action to `git add -p` sparked debate about command boundaries, with Junio suggesting a more comprehensive interactive command might better serve the workflow need.

**Test modernization microproject** -- Kaartic Sivaraam confirmed adding the pluralization fix project to Git's official microprojects list for programs like GSoC, building on Phillip Wood and Junio's identification of i18n issues in message handling.

**`git repo` help text improvements** -- Mahi Kassa's series to make subcommand help text more specific is now merge-ready after addressing all review feedback, with Junio acking the final version.