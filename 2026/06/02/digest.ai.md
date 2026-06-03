# Git Mailing List Digest - 2026/06/02 (Tuesday)

**The day in brief.** A busy day with mostly routine activity - documentation updates, test infrastructure improvements, and ongoing series iterations dominated the traffic. Notable threads include the completion of the `the_repository` removal series and Junio's "What's cooking" report. The most significant technical discussion centered around HTTP authentication fixes and worktree metadata tracking.

## Notable threads

### `the_repository` removal series merges

Bello Olamide's 8-patch series completing the migration of configuration globals into `struct repo_config_values` reached its final form today with v5. The series, which has been through multiple review iterations, now removes the last remaining `the_repository` dependencies for configuration settings like `core.warnAmbiguousRefs` and `core.precomposeunicode`. 

Junio Hamano provided final review feedback about subtle architectural considerations in repository configuration access patterns, noting that the series remains merge-ready despite some remaining `the_repository` usage in object-file.c that can be addressed later. Patrick Steinhardt and Christian Couder agreed on the phased approach, with the maintainer signaling readiness to merge pending minor documentation fixes. The series represents a major milestone in the multi-year effort to eliminate global state from Git's codebase.

### Worktree metadata tracking proposal

A new feature proposal adds creation timestamps and user-provided descriptions to Git worktrees. The patch introduces:

- Automatic recording of creation timestamps in `.git/worktrees/<id>/created`
- A `--note` option for `git worktree add` to set descriptions
- New `git worktree annotate` subcommand
- Enhanced listing options (`--show-created`, `--show-note`, `--sort`)

Junio Hamano suggested in review that the existing `description` file mechanism might be more appropriate than introducing new `note` files, though he noted this wasn't a blocking concern. The proposal includes comprehensive test coverage and maintains backward compatibility.

### HTTP authentication bugfix

A bugfix addresses loss of WWW-Authenticate headers during HTTP redirects, which particularly impacts OAuth authentication flows. The 14-line change in http.c` preserves these headers when credentials are cleared during redirect processing, ensuring credential helpers can still see supported authentication methods. The patch includes new test cases verifying the fix works with bearer token authentication after redirects.

## In brief

**`git branch --prune-merged` review** -- Phillip Wood acknowledged receipt of Harald Nordgren's v11 series and plans to review it later this week. The series adds safe automated branch pruning with push-tracking safety checks and dry-run support.

**Test infrastructure improvements** -- Patrick Steinhardt's series fixing TAP output validation issues was reviewed, addressing problems in fsmonitor tests, Windows EBUSY handling, and git-p4 process cleanup. Junio noted a variable naming inconsistency that was quickly corrected.

**Documentation updates** -- Junio Hamano merged his own series updating SubmittingPatches with better typo-fix guidance and cover letter advice. The changes formalize existing mailing list practices into official contributor documentation.

**Timestamp prefix documentation** -- Luna Schwalbe's patch documenting the `@` prefix requirement for raw Unix timestamps was approved by Junio. The behavior has existed since 2012 but was never formally documented until now.

**Git Rev News** -- Christian Couder announced the 135th edition of the project newsletter, covering May 2026 developments and opening issue #846 for community contributions to the next edition.

## On the radar

**`b4` tool standardization** -- Patrick Steinhardt's series to promote `b4` for mailing list workflows sparked discussion about configuration approaches and documentation sequencing. The thread remains active with questions about threading models and new contributor onboarding.

**Priority queue optimizations** -- Kristofer Karlsson and René Scharfe continued discussing alternative approaches to optimizing `prio_queue_get()`, with benchmark results showing their methods perform comparably in most cases.