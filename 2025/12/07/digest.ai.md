# Git Mailing List Digest - 2025/12/07

**The day in brief.** A moderately active Sunday with 23 emails across 14 threads saw several notable developments: Junio Hamano made key rulings on pending patches (accepting a Windows pthread emulation fix while rejecting short-option completion), the `git replay --revert` series advanced with refinements, and a serious `git pull --rebase` data loss bug was reported. The day also featured a "What's cooking" update and continued discussion about structured data versioning approaches.

## Notable threads

**Windows pthread emulation fix accepted**  
Junio Hamano gave final approval to Greg Funni's patch fixing POSIX compliance in Git's Windows pthread emulation layer, specifically addressing `pthread_cond_init`'s return value. While calling the comma-operator workaround "tricky and yucky," Junio acknowledged its correctness and plans to queue it for 'next' despite lacking Windows-specific review. This concludes a three-iteration series that maintains consistency with existing patterns in the Windows compatibility layer.

**Short-option completion rejected**  
In a decisive ruling, Junio closed discussion on adding bash completion for Git's short options (-C, -P, etc.), siding with SZEDER Gábor's critique that the feature would have limited value without explanatory help text during tab completion. The decision references Git's longstanding philosophy favoring discoverability through documentation over mere completion capability, echoing a 2007 discussion. Wiktor Mis's contribution was acknowledged but the rejection appears final unless someone can propose a way to incorporate help text display.

**git replay --revert refinements**  
Siddharth Asthana's `git replay --revert` series saw substantive review responses addressing Patrick Steinhardt's feedback. The author explained the architectural rationale for keeping revert message formatting separate between interactive (sequencer) and non-interactive (replay) contexts, while agreeing to several implementation improvements: adding defensive `BUG()` checks, safer memory management with `FREE_AND_NULL()`, and consolidated option validation. These changes will appear in a forthcoming v3, moving the series closer to readiness while maintaining its focus on consistent revert behavior across Git commands.

**Structured data versioning debate expands**  
The ongoing discussion about versioning structured data took an interesting turn with Simon Richter proposing database-native approaches using temporal columns, contrasting with previous focus on filesystem-based solutions. Using KiCad's CAD data challenges as a case study, Richter highlighted limitations of line-oriented diffs for multi-dimensional data. Cedric Sodhi countered that proper filesystem mapping strategies could leverage Git's existing capabilities with custom tooling. The thread continues to explore fundamental architectural tradeoffs without clear resolution.

**Serious pull --rebase bug reported**  
A concerning bug report demonstrated data loss in `git pull --rebase` when working with multiple push URLs. The issue manifests when pushes partially fail across configured destinations, causing the rebase operation to incorrectly handle local commits during subsequent synchronization. With clear steps to reproduce across Git versions, this appears to be a potentially serious issue warranting prompt investigation from rebase/push experts.

## In brief

**Multiple push URL memory leak** -- A focused fix from Junio addresses a memory leak in server capability handling when pushing to multiple remotes, accompanied by a new test case.

**git repo info --keys flag** -- A new flag to list available repository information keys saw initial implementation, with Junio suggesting refinements for option combination handling.

**git replay documentation** -- Kristoffer Haugsbakk's 3-part series clarifying `git replay`'s silent conflict behavior and undocumented `--contained` option received maintainer approval.

**Fetch tag backfill fix** -- Karthik Nayak's series fixing tag fetching during batched reference updates is queued for 'next' after a minor typo correction.

**whatchanged deprecation feedback** -- Users provided practical feedback about retaining `git whatchanged` functionality through aliases, highlighting its mnemonic value over equivalent `git log` invocations.

## On the radar

**ODB abstraction effort** -- Patrick Steinhardt's ongoing object database refactoring was noted in Junio's "What's cooking" as making progress, with several related topics now in 'next'.

**the_repository removal** -- René Scharfe's long-running effort to eliminate the global variable saw continued incremental progress mentioned in the status update.