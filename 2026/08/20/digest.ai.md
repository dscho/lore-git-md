# Git mailing list daily digest for 2026/08/20

## The day in brief
The Git mailing list saw active development across several fronts today. A long-awaited bugfix for Git's trailer parsing graduated to final integration, while the `git history squash` feature series addressed its last technical blocker. Documentation improvements and performance optimizations also made progress, with several series reaching completion or nearing readiness for merging.

## Notable threads

### Trailer parsing bugfix ready for final integration
[2026/06/04/21-27-51]

Kristoffer Haugsbakk posted v2 of the trailer-parsing patch that prevents Git from incorrectly interpreting URLs in commit messages as trailers. The patch, which addresses three concrete bug scenarios affecting both `--only-trailers` and `%(trailers:only)` formatting, is now ready for final integration.

The solution modifies `find_separator()` in `trailer.c` to exclude lines containing `://` from trailer parsing, using `starts_with(c, "://")` for improved readability. This implementation handles all URL schemes uniformly while preserving legitimate trailers. The patch includes comprehensive test coverage and documentation updates, and explicitly acknowledges the theoretical possibility of false positives (e.g., `1://` or `-://`).

Junio C Hamano queried the status of outstanding issues, and Kristoffer confirmed the patch is ready for integration after incorporating Jeff King's readability improvement.

### `git history squash` addresses final technical blocker
[2026/06/14/19-25-39]

Harald Nordgren posted v14 of the `git history squash` series, which is now functionally complete except for one critical autosquash marker resolution bug. The series introduces a new subcommand that folds a range of commits into their oldest ancestor in a single operation, avoiding the repeated conflict stops of a rebase-based approach.

Junio C Hamano identified a security and correctness concern in the autosquash marker resolution logic: the current implementation uses `istarts_with()` for case-insensitive prefix matching of commit OIDs, which would incorrectly match `"fixup! ABCdef"` to a commit whose OID starts with `"abcdef"`. This creates potential for unintended squashes or security issues if an attacker can craft a commit message that collides with a target OID when case is ignored.

The fix will require replacing `istarts_with()` with `starts_with()` to enforce case-sensitive matching, consistent with Git's standard OID handling. This is the final remaining technical hurdle before the series can be integrated.

### ODB transaction series completes final polish
[2026/08/06/21-38-53]

Justin Tobler posted v5 of the nine-patch series extending the ODB transaction system to make `git receive-pack` fully backend-agnostic. The series replaces hardcoded use of `git index-pack` and `git unpack-objects` with a new generic transaction interface, `odb_transaction_write_pack()`, enabling alternative ODB backends to handle incoming packfiles.

The v5 iteration addresses the last remaining feedback from v4, including Patrick Steinhardt's request for additional test coverage to verify the ".keep" file's migration during the reference update phase. The series now includes a new test step that uses a `reference-transaction` hook to check for the ".keep" file's presence in the main ODB during reference updates and its removal afterward.

The series also incorporates Patrick's suggested commit message rewording and resolves the `pack_fd` file descriptor lifecycle question raised by Junio C Hamano. The implementation is now technically complete and ready for merging.

### Documentation improvements for `gitdatamodel(7)`
[2026/08/20/18-55-20]

Kristoffer Haugsbakk posted a four-patch documentation series that improves the discoverability of the `gitdatamodel(7)` man page. The series adds cross-references from key documentation files (`git(1)`, `gitglossary(7)`, and related glossary content) to the data model tutorial, which was previously unlinked from any other documentation.

The changes are purely additive and touch only documentation files:
- Patch 1 adds `gitdatamodel(7)` to `command-list.txt` to ensure it appears in the "Guides" section
- Patch 2 adds cross-references from `git(1)` under "Description", "Terminology", and "See Also"
- Patch 3 adds links from four glossary terms (blob, commit, object, tree) to the data model page
- Patch 4 adds reciprocal links from `gitdatamodel(7)` back to `gitglossary(7)`

The author tested the changes by merging with the current `seen` branch and running the full test suite, confirming no regressions.

### HTTP authentication header preservation
[2026/08/20/03-21-04]

Aaron Plattner posted v2 of a bugfix that preserves `WWW-Authenticate` headers during HTTP redirects. The patch addresses a regression where Git's HTTP layer loses these headers when updating the credential struct for the new URL, breaking authentication when the redirect target requires a credential helper.

The fix introduces `credential_update_url()`, a new helper in `credential.c` that wraps `credential_from_url()` but preserves the `wwwauth_headers` field across URL updates using `SWAP()` for efficiency. The implementation is minimal and focused, with a new test in `t/t5563-simple-http-auth.sh` verifying the preservation behavior.

## In brief

- **[zsh completion]** D. Ben Knoble confirmed the zsh completion patch works as intended, aligning with the bash implementation for global Git options like `-C`, `--git-dir`, and `-p`.
- **[worktree repair]** Yoichi NAKAYAMA posted v2 of a bugfix for worktree repair functionality, introducing `read_gitfile_raw()` to preserve the original path style in `.git` files.
- **[geometric repacking]** Patrick Steinhardt praised the MIDX recovery logic in Elijah Newren's geometric repacking fix but raised a question about whether the fix is strictly necessary for callers that don't use `OBJECT_INFO_QUICK`.
- **[reftable optimization]** Jeff King provided benchmark results showing a 25% speedup for Karthik Nayak's reftable stack-reload optimization, confirming the effectiveness of the syscall reduction.
- **[ODB error handling]** Karthik Nayak and Patrick Steinhardt discussed code hygiene in the ODB error-handling series, with Karthik suggesting explicit enum usage for clarity and Patrick preferring the established return convention.
- **[git pull --hard]** Artur Bieniek endorsed Phillip Wood's `--reset` redesign for `git pull --hard`, arguing it better aligns with Git's safety principles and addresses a specific workflow gap for force-pushed or rebased upstream branches.
- **[git request-pull]** Gahtan nahdi reported a false warning in `git request-pull` when the `<ref>` argument is a commit hash instead of a ref name.
- **[symbolic-ref tests]** Nikolaus Schuetz clarified the motivation for `git symbolic-ref` exit code tests and explained the technical rationale for the differing exit codes (128 vs. 1 with `--quiet`).