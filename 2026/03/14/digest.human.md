Here's the daily digest for March 14, 2026:

## The day in brief
March 14 saw moderate volume with 62 emails across 28 threads, dominated by several significant patch series and GSoC proposal discussions. Key developments include Jiang Xin's v3 AI-assisted l10n workflow patches, Harald Nordgren's autostash-based branch switching improvements, and multiple GSoC proposals nearing submission deadlines. The day also featured ongoing discussions about kernel.org responsive design and routine test modernizations.

## Notable threads

### AI-assisted translation workflows reach v3
Jiang Xin's five-part series introducing AI-assisted workflows for Git's localization process has progressed to v3 with comprehensive documentation in `po/AGENTS.md`. The patches standardize PO file handling with `.gitattributes` filters, separate AI-specific instructions from human workflows, and demonstrate significant efficiency gains (82% step reduction, 76% time savings). While technically complete, philosophical questions about AI-generated content raised earlier by brian m. carlson remain unresolved. The maintainer appears satisfied with the technical implementation, suggesting this will likely progress despite outstanding concerns.

### Branch switching conflict resolution refined
Harald Nordgren's v4 patch series modifies `git checkout -m` to use autostash behavior for conflict resolution during branch switching, implementing Junio Hamano's preferred architectural approach over adding a separate autostash option. The changes improve conflict marker labeling and make stash creation silent during checkout operations. The submodule handling issue raised by Phillip Wood remains the primary technical blocker, with Junio suggesting submodules could handle their own stashing independently rather than expanding `git stash`'s responsibilities.

### Zombie process cleanup follow-up
Jeff King identified a memory leak in `transport_color_config()` that surfaced after Andrew Au's zombie process cleanup patch was queued. The 4-line fix switches to `repo_config_get_string_tmp()` to avoid allocations, addressing a leak that had existed since 2018 but only became detectable under specific test conditions. Junio confirmed he will queue this straightforward fix while noting a potential larger cleanup opportunity around color configuration helpers.

### `git subtree` recursion fix proposed
Colin Stagner proposed a v2 patch series addressing the long-standing recursion depth limitation in `git subtree` when processing large histories like PostgreSQL's (52k commits). The fix targets the mutual recursion between `process_split_commit()` and `check_parents()` in the shell script implementation. This represents progress on an issue first reported in 2021, with the patch now awaiting testing against the original reproduction case.

## In brief

**GSoC proposal for promisor remote priority** -- Lorenzo Pegorari's proposal to implement configurable fetch ordering for promisor remotes in partial clones received constructive feedback from Christian Couder, who suggested simplifying the priority configuration name and using a fixed range (1-100) rather than `UCHAR_MAX`.

**GSoC proposal for remote-object-info** -- Pablo Sabater submitted a revised proposal to complete and extend `git cat-file --remote-object-info`, addressing feedback to improve organization and clarity while demonstrating strong familiarity with Eric Ju's stalled v11 series.

**GSoC proposal for partial clone "un-fetch"** -- Siddharth Shrimali proposed allowing Git to "un-fetch" blobs in partial clones to reclaim disk space, showing promising technical understanding through preparatory study of the relevant code and prior test infrastructure contributions.

**Test modernization corrections** -- Junio C Hamano provided guidance on proper patch submission workflow after a test modernization patch attempted to correct an intermediate state that wasn't in the main repository, emphasizing the importance of presenting clean logical progressions.

**Coccinelle rule for strbuf usage** -- Deveshi Dwivedi proposed a Coccinelle rule to detect when `struct strbuf` is passed by value rather than pointer, identifying two existing instances in the codebase following Jeff King's suggestion during an earlier review.

**Format-patch commit list improvements** -- Mirko Faina's 7-patch series enhances `git format-patch --commit-list-format` with better error messages, refactored internals, a new "modern" preset format, and removal of the "log:" prefix requirement for custom formats.

## On the radar

**`the_repository` removal effort** -- Tian Yuchen's GSoC proposal about reducing Git's global state is nearing submission deadline, having incorporated feedback on configuration parsing distinctions while awaiting final confirmation from René Scharfe for full alignment with the broader effort.

**Kernel.org responsive design** -- Rito Rhymes' 6-patch series implementing mobile-friendly improvements for kernel.org has moved to public discussion, with Junio noting the Git list may not be the ideal venue despite the connection through mailing list archives.