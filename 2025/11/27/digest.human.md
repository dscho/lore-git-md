# Git Mailing List Digest - 2025/11/27

**The day in brief.** A moderately busy Thursday with 41 emails across 13 threads, featuring significant progress on several fronts. Key developments include resolution of the Lisp userdiff driver discussion, architectural alignment on reference backend URIs, and the introduction of a major Rust infrastructure RFC. The `git replay` command saw multiple refinements, while translation and documentation improvements continued across several subsystems.

## Notable threads

### Rust infrastructure refactoring RFC

Ezekiel Newren proposed a significant restructuring of Git's Rust infrastructure into a Cargo workspace architecture (13-patch series). The RFC aims to improve modularity and compilation efficiency by splitting the Rust code into multiple crates (gitcore, link-with-c, generate-headers) rather than maintaining a single monolithic crate. The series undoes some of Patrick Steinhardt's prior Rust work and introduces new build system integration, though it remains incomplete with known issues around Windows builds and CI workflows. The proposal represents a major architectural shift that would affect how Rust integrates with Git's build system, with the RFC status indicating the author is seeking feedback before finalizing the approach.

### Reference backend URI implementation

The discussion around reference storage backend selection via URI syntax reached consensus on using environment variables (`GIT_REF_URI`) for temporary reference store overrides. Junio Hamano initially proposed explicit API calls but ultimately accepted the environment variable approach as fitting Git's existing patterns for temporary configuration overrides. The thread focuses on resolving remaining questions about URI format syntax (`://` vs `:`) and filesystem path handling. Karthik Nayak's implementation appears to be nearing finalization with this architectural question settled.

### Unified Lisp dialect support in userdiff

Scott Burson's v2 series introducing a unified "lisp" userdiff driver achieved technical resolution after addressing feedback about whitespace handling and backward compatibility. The implementation combines Scheme and Common Lisp/Emacs Lisp patterns through disjunctive regexps that handle case sensitivity, constrained whitespace, and vertical bar syntax. Johannes Sixt provided final review suggesting minor presentation improvements but confirmed the technical soundness of the approach. The patch series awaits only procedural adjustments to its submission format before integration.

### `git replay` behavior refinements

Multiple discussions refined the experimental `git replay` command's behavior. Siddharth Asthana confirmed the implementation already supports reverting non-contiguous commits through standard revision range syntax, addressing Junio Hamano's workflow requirements. Another patch modified `git replay` to drop commits that become empty during replay (matching `git rebase`'s behavior), with Phillip Wood suggesting an option to retain empty commits if needed. The command appears to be maturing rapidly with thorough attention to edge cases and documentation.

### Hook subsystem refactoring progress

Adrian Ratiu's hook modernization series saw review feedback addressed on the parallel process infrastructure changes. The discussion focused on final polish items for the pre-push hook conversion, particularly around pipe closure timing and stdin feeding mechanics. Junio Hamano's review indicated satisfaction with the design separation between generic and client-specific code, suggesting the foundational changes are stabilizing well.

## In brief

**Documentation warning for `--committer-date-is-author-date`** -- Kristoffer Haugsbakk's patch documenting the risks of this option received final approvals, concluding a thread that began with questions about the option's validity. The documentation now clearly warns about violating Git's timestamp ordering assumptions.

**Submodule checkout behavior inquiry** -- Alon Bar-Lev sought guidance on implementing support for non-branch references in submodules, specifically asking whether patches supporting Gerrit's review refs (refs/changes/) would be accepted.

**Bash completion for Git short options** -- The proposed feature faced fundamental objections from SZEDER Gábor and Junio Hamano, who questioned its value without explanatory context during tab completion.

**Performance optimization for patience diff** -- Yee Cheng Chin's v2 patch optimizing LCS calculation in the patience diff algorithm showed consistent 8-14% speedups across multiple codebases when using `git log --patience`.

**Scalar configuration cleanup** -- Derrick Stolee's series removing stale config values and adding comprehensive documentation received positive review from Junio Hamano, with only minor editorial fixes remaining.

**Localization inconsistencies** -- brian m. carlson highlighted an English-only "Everything up-to-date" message in `git push` that contrasts with localized equivalents in `git pull`, sparking discussion about plumbing stability versus user-facing i18n needs.

## On the radar

**Rust infrastructure debate** -- Ezekiel Newren's RFC series will likely generate significant discussion about Git's Rust integration strategy, particularly around platform support and build system implications.

**Reference backend migration** -- With the environment variable approach now accepted, attention turns to finalizing the URI format syntax as Karthik Nayak's series nears completion.

**Submodule refspec flexibility** -- Alon Bar-Lev's inquiry about non-branch references in submodules may lead to patches if maintainers indicate willingness to accept such changes.