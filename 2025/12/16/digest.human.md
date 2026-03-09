Here's the daily digest for December 16, 2025:

---

### The day in brief
December 16 saw steady activity with 93 emails across 21 threads. The day was dominated by ongoing technical discussions around several key areas: Windows symlink support preparations, submodule path encoding, hook API modernization, and object size reporting. Notable progress was made on multiple fronts, with several patch series reaching final refinement stages.

---

### Notable threads

**Submodule gitdir path encoding reaches final polish**  
Adrian Ratiu's series to improve submodule gitdir path handling through the new `extensions.submodulePathConfig` extension is nearing completion. Today's discussion focused on final refinements to the migration command's atomicity (Patrick Steinhardt suggested reversing the config change order for better failure recovery) and documentation improvements (Junio Hamano requested more concrete examples of when users would need this feature). Josh Steadmon surfaced two implementation issues: unexpected behavior with pre-existing encoded paths and a bug where recursive clones weren't properly creating required gitdir config entries when the extension is enabled. The series appears technically sound but these edge cases will need addressing before final integration.

**Hook API modernization awaits final review**  
Adrian Ratiu's 11-part series to modernize Git's hook API saw positive review from Patrick Steinhardt ("mostly good") with only minor questions remaining about the pre-push hook's status filtering behavior. Junio Hamano maintained cautious optimism but emphasized wanting to personally review the foundational API changes before merging to 'next'. The thread highlighted Git's careful approach to core infrastructure changes, where passing tests alone isn't sufficient grounds for integration without thorough architectural review.

**Windows symlink support preparations complete**  
Johannes Schindelin's 10-part series to prepare Git's test suite for Windows symlink support was accepted by Junio Hamano after three iterations. The changes fix long-standing test issues that would surface when MSYS2 enables symlink support by default, touching areas from apply.c's executable bit handling to mingw.c's O_CREAT|O_EXCL behavior. A separate 5-part follow-up series from Schindelin and Karsten Blees addressed additional Windows symlink prerequisites, including getcwd() resolution and strbuf_readlink() improvements. These changes clear critical blockers for proper Windows symlink support.

**Object size reporting series finalized**  
Justin Tobler's series adding object size reporting to `git repo structure` reached v4 with all i18n concerns resolved. The thread saw extensive discussion about translation marking consistency for "byte"/"bytes" strings, ultimately settling on removing a conflicting `N_()` wrapper in test code. The implementation now provides comprehensive size metrics (both inflated and on-disk) across all output formats with proper human-readable formatting and translation support. Junio approved the final version pending a minor documentation tweak to translator comments.

---

### In brief

**HTTP authentication fix for filtered clones** -- Lucas De Marchi confirmed a fix for probe_rpc() authentication in remote-curl, addressing 401 errors during filtered clone operations.

**`git replay` empty commit handling** -- Phillip Wood's series to make `git replay` drop redundant commits reached v3 with punctuation improvements from Junio's feedback and is ready for merging.

**MacOS build system refactoring** -- René Scharfe proposed v3 changes to improve Homebrew integration, introducing explicit USE_HOMEBREW_* flags while questioning if they add unnecessary complexity.

**Sign-off policy documentation finalized** -- The thread concluded with consensus documentation explaining Git's stance against automatic sign-off options (except format.signoff) to preserve DCO intentionality.

**Rev-list filter behavior documented** -- Justin Tobler's patch documenting `--filter-provided-objects` behavior was queued, though reviewers agreed the current design is suboptimal and may warrant future reconsideration.

**Outreachy internship begins** -- Bello Olamide started work on reducing Git's global state, beginning with the `git_attributes_file` variable and studying prior feedback about accessing attributes through `struct index_state`.

---

### On the radar

**ODB abstraction refactoring** -- Patrick Steinhardt's packfile storage changes continue evolving, with Justin Tobler noting some transitional awkwardness in alternate ODB handling that will be resolved later in the series.

**MacOS iconv workarounds** -- Torsten Bögershausen's patches to address macOS 15's broken ISO-2022-JP conversion await feedback on the proposed three-pronged workaround strategy.

**Build system configuration** -- The discussion around Homebrew vs MacPorts handling in the macOS build system remains open, particularly regarding the fine-grained USE_HOMEBREW_* flags.