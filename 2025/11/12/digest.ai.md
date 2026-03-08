Here's the daily digest for November 12, 2025:

## The day in brief

A busy day with 78 emails across 18 threads, featuring significant progress on multiple fronts. Key highlights include Junio Hamano's comprehensive whitespace handling series reaching its final form, resolution of a security vulnerability in attribute macro expansion, and important discussions about Git's data model documentation. The day also saw the release of Git v2.52.0-rc2 and a "What's cooking" report tracking the project's progress.

## Notable threads

### Finalizing incomplete-line whitespace handling

Junio Hamano's 12-patch series implementing comprehensive handling of missing terminating newlines (WS_INCOMPLETE_LINE) reached its final form today. The series now provides end-to-end support through `core.whitespace` and `.gitattributes`, covering diff generation, patch application, and project-wide enforcement. The implementation includes colored no-newline markers in diffs, `--check` integration, and automatic correction capabilities. The final patch enables these checks for Git's own codebase, applying them to C files, headers, shell scripts while excluding documentation files.

### Security hardening for attribute macro expansion

A security discussion concluded with consensus on converting recursive attribute macro expansion to an iterative approach to prevent stack overflow attacks. While the solution trades stack usage for heap allocation, participants agreed this provides more robust protection given existing platform limits. The thread revealed Git's general security philosophy of relying on OS-level resource protections rather than application-level restrictions for such cases.

### Git data model documentation finalized

Julia Evans' `gitdatamodel.adoc` documentation patch series reached version 7 after extensive review, with only minor phrasing issues around branch definitions remaining. The document now comprehensively explains Git's core concepts (objects, references, index, and reflogs) without requiring plumbing command knowledge. Junio Hamano raised final questions about whether to define branches narrowly as refs or more conceptually as workflow tools, but the technical implementation appears ready for merge.

### Mixed hash algorithm validation

Following a bug report about submodule checkout failures in SHA256 repositories, the thread established that mixing SHA1 and SHA256 repositories is intentionally unsupported. brian m. carlson announced an upcoming patch to explicitly reject such mixed-hash scenarios at the `git add` boundary, preventing repository corruption. This hardening measure aligns with the project's long-standing position on hash algorithm interoperability constraints.

### Fast-import signature stripping debate

The discussion about adding 'strip-if-invalid' mode to `git fast-import`'s `--signed-commits` option progressed with Christian Couder addressing test failures and Junio Hamano clarifying concerns about cascading signature invalidation. While the maintainer acknowledged the feature's utility for selective signature preservation during history rewriting, open questions remain about handling non-linear history cases.

## In brief

**Submodule path encoding refinement** -- Adrian Ratiu proposed a simpler implementation for case-folding collision detection in submodule gitdir paths, replacing hybrid filesystem/text comparison with direct directory scanning.

**`git fetch` tag handling regression fix** -- Karthik Nayak clarified behavior of `prune_refs()` in the context of the tag-fetching fix, confirming no transaction is present during pruning operations.

**OSX keychain credential helper revert** -- Koji Nakamaru proposed reverting a problematic optimization that could prevent valid credentials from being stored when multiple helpers are involved.

**ASan findings series** -- Jeff King submitted a 9-patch series addressing memory safety issues found through AddressSanitizer analysis, including bitmap handling fixes and fsck improvements.

**`git last-modified` bugfix** -- Toon Claes clarified that a patch previously framed as an optimization actually fixes incorrect output when files are specified via grep versus direct path.

**Rebase trailer support** -- Phillip Wood reviewed Li Chen's v6 series adding `--trailer` support to `git rebase`, indicating it's in good shape with only cleanup needed.

## On the radar

**Committer identity controls** -- ZheNing Hu's proposal for `--committer` flag symmetry in `git commit` faces maintainer skepticism about its necessity, with Junio Hamano questioning whether it solves a real problem.

**HTTP authentication fix** -- A minimal patch addresses missing auth in `remote-curl`'s `probe_rpc()`, particularly impacting filtered clones, though token expiration cases remain for future work.