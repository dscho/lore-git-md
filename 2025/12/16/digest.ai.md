Here's the Git mailing list digest for December 16, 2025:

## The day in brief

A busy Tuesday with 93 emails across 21 threads, dominated by ongoing technical discussions around submodule path encoding, Windows symlink support, and object size reporting. Notable developments include finalization of the sign-off policy documentation and progress on multiple large refactoring efforts. The submodule path encoding series hit some implementation snags while the Windows symlink preparation work nears completion.

## Notable threads

### Submodule path encoding implementation challenges

The submodule gitdir path encoding series (started by Adrian Ratiu) encountered unexpected behavior in its collision detection logic. Josh Steadmon reported that the path encoding system fails to properly handle cases where a submodule path would conflict with an existing encoded path, instead showing an error about the existing gitdir. Additionally, the automatic configuration of `submodule.<name>.gitdir` paths during cloning isn't working as documented when `extensions.submodulePathConfig` is enabled globally.

Patrick Steinhardt provided extensive review feedback throughout the series, particularly around:
- Improving migration atomicity by reordering configuration changes
- Restricting automatic extension enablement to new repositories only
- Documentation clarity for the new configuration options

The thread shows the series moving from architectural consensus to implementation polish, with these newly discovered edge cases requiring attention before finalization.

### Windows symlink support preparation finalized

Johannes Schindelin's 10-part series preparing Git's test suite for Windows symlink support was accepted into the integration branches after three iterations of refinement. The changes address various platform-specific behaviors in the test suite that would fail when MSYS2 enables symlink support by default. Key fixes include:

- Proper handling of Windows path normalization in test comparisons
- Correcting `open()` behavior with `O_CREAT|O_EXCL` flags to match POSIX
- Platform-specific adjustments for credential cache and difftool tests

Junio Hamano approved the series after verifying the POSIX-standard justification for the mingw.c changes and confirming all test accommodations were properly scoped.

### Object size reporting reaches final form

Justin Tobler's series adding object size reporting to `git repo structure` reached its fourth iteration, now with comprehensive i18n support and refined test coverage. The implementation provides both inflated and on-disk size metrics for repository objects (commits, trees, blobs, tags) across all output formats (table, keyvalue, nul). 

Key technical elements:
- New `humanise_bytes()` and `humanise_count()` functions in strbuf.c
- Proper plural-form handling for translations
- Special compact "B" unit formatting for tag sizes
- Dynamic column width adjustment in table output

The series went through multiple rounds of review addressing i18n consistency, test script style, and documentation clarity before achieving merge readiness.

## In brief

**Sign-off policy documentation finalized** -- The project codified its stance against automatic sign-off configurations (beyond the historical `format.signoff` exception) with new documentation in both the FAQ and signoff-option.adoc, emphasizing the legal importance of explicit `--signoff` flags.

**HTTP authentication fix for filtered clones** -- Lucas De Marchi confirmed a fix for probe_rpc() requests failing during filtered clones when using git-credential-msal, by adding proper authentication headers to these requests.

**`git replay` empty commit handling** -- Phillip Wood's patch to make `git replay` drop commits that become empty during replay (matching rebase/cherry-pick behavior) received final punctuation polish from Junio Hamano and appears ready for merging.

**MacOS build system refactoring** -- René Scharfe proposed v3 changes to improve Homebrew integration in the build system, introducing explicit flags (USE_HOMEBREW_GETTEXT, USE_HOMEBREW_LIBICONV) while maintaining backward compatibility.

**Outreachy internship begins** -- Bello Olamide started work on reducing Git's global state, beginning with the `git_attributes_file` variable and studying prior feedback about accessing attributes through `struct index_state`.

## On the radar

**Hook API modernization** -- Adrian Ratiu's series to replace ad-hoc `run_command()` hook calls with a structured API awaits Junio Hamano's final review before potential merging to 'next', having addressed all technical feedback from other reviewers.

**ODB abstraction effort** -- Patrick Steinhardt's packfile storage refactoring continues with discussions about transitional states in the codebase as it moves toward tracking packs per source rather than per ODB.

**`--filter-provided-objects` behavior** -- Documentation now accurately reflects the surprising default where explicitly provided objects bypass filters, but discussion continues about whether this behavior should be changed in future.