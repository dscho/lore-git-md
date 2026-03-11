Here's the Git mailing list digest for March 9, 2026:

## The day in brief

A busy Monday with 115 emails across 37 threads saw several long-running efforts reach completion while new proposals emerged. Key developments include final approvals for HTTP 429 rate limiting, submodule remote handling, and `the_repository` removal in `wt-status.c`. The parallel hooks series resurfaced with a rebased version, while discussions around `git replay` subcommands and partial clone disk management gained traction.

## Notable threads

**HTTP 429 rate limiting support finalized**  
Vaidas Pilkauskas's series adding HTTP 429 (rate limiting) support to Git's HTTP client received Junio Hamano's approval for the strbuf fixes portion, with only the HTTP transport changes awaiting final review from Jeff King or Taylor Blau. The implementation includes comprehensive handling of Retry-After headers, configurable retry behavior, and Trace2 integration while maintaining Git's conservative fail-fast defaults. With extensive test coverage already in place, this five-iteration series appears ready for integration pending that final review.

**Submodule remote handling bugfix complete**  
After six iterations addressing technical implementation, test portability, and output ordering issues, Nasser Grainawi's patch fixing submodule remote handling has been approved for integration. The change removes the hardcoded assumption that submodule remotes are always named "origin", implementing a new `get-default-remote` helper command. Junio Hamano confirmed the series is ready for 'next' after the final test robustness improvements, resolving a 7-year-old NEEDSWORK comment in the process.

**Global state removal from wt-status.c**  
Shreyansh Paliwal's three-part series eliminating `the_repository` and `the_hash_algo` globals from the status subsystem received final approvals from both Karthik Nayak and Phillip Wood. The changes convert status-related functions to use explicit repository parameters throughout `wt-status.c`, following Git's established pattern for global state removal. With all technical concerns addressed across multiple iterations, Junio has queued the topic for integration.

**Parallel hooks series rebased**  
Adrian Ratiu resubmitted his parallel hook execution series as a rebase-only update (v3) from the version currently in 'next'. The 9-patch implementation provides multiple opt-in control points for parallelization through config settings, command-line options, and per-hook markers. While functionally unchanged from previous iterations, the rebase keeps the topic active as it awaits final review before potential merging.

**Editor configuration refactoring debate**  
The discussion around scoping editor configuration continued, with Karthik Nayak advocating for an incremental approach to moving `editor_program` from global to repository-specific storage. While acknowledging Burak Kaan Karaçay's arguments for full repository isolation, Karthik cautioned against prematurely bloating `struct repository` without clear need, suggesting first localizing the global variable within editor.c before deciding on final placement.

## In brief

**Reftable documentation update** -- brian m. carlson confirmed that reftable is stable despite being labeled "experimental" in docs, noting it will become the default in Git 3.0.

**Const-correctness in patch-id** -- Tian Yuchen's patch documenting the intentional const-cast in `patch_id_neq()` was approved with minor wording tweaks to clarify it preserves a rebase optimization from 2016.

**Git quiltimport edge cases** -- Sasha Levin and Junio Hamano discussed whitespace handling implications of replacing `echo` with `printf` in patch subject processing, revealing subtle behavioral differences in backslash handling between shells.

**Test modernization** -- Pablo Sabater's GSoC contribution updating t9200 to use test helpers was approved after addressing commit message feedback, while another test patch was flagged for incorrectly applying assertions to prereq checks.

**Subcommand autocorrection** -- Jiamu Sun's series received style feedback on variable declarations and control flow in the autocorrect implementation, with the author agreeing to restructure patch 2/7 as requested.

**Documentation synopsis conversion** -- Jean-Noël Avila's ongoing effort converted several more man pages to the consistent synopsis style, with Kristoffer Haugsbakk contributing minor wording improvements.

## On the radar

**Partial clone disk management** -- Yuvraj Singh Chauhan's GSoC exploration of where to implement disk space recovery functionality (gc/maintenance/backfill) remains in early discussion stages with no consensus yet on the right approach.

**git replay subcommands** -- Toon Claes's RFC proposes converting the experimental command's --onto/--advance/--revert options to subcommands, following Git's conventional CLI patterns while maintaining backward compatibility.

**Remote group push** -- Usman Akinyemi's RFC for push-to-group functionality received key design guidance from Junio Hamano establishing that group pushes should behave identically to sequential individual pushes for both simple push.default and force-with-lease cases.