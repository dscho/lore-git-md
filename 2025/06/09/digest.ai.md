# Git Mailing List Digest - 2025/06/09

**The day in brief.** A busy Monday with 76 emails across 17 threads, dominated by the finalization of a major `imap-send` overhaul and discussions around tag handling improvements. Key developments include the v18 submission of the comprehensive `imap-send` feature series, a debate about duplicate tag behavior, and the release of Git v2.50.0-rc2. Junio's "What's cooking" report provides a snapshot of the current development landscape.

## Notable threads

### `imap-send` resurrection complete

Aditya Garg's long-running `imap-send` overhaul reaches its final form in v18, addressing all review feedback. The series fixes a critical configuration parsing regression (broken since Git 2.46.0) and adds modern OAuth2.0 authentication support via both RFC-standard OAUTHBEARER and Google-proprietary XOAUTH2 protocols. New features include comprehensive management of IMAP folders via `--list` and `--folder` options, with RFC6154-compliant handling of folder enumeration. The implementation provides proper OpenSSL availability checking and refactored authentication logic, with user experience improvements like showing destination folders during send operations. After 18 iterations and extensive review, this substantial modernization of Git's email submission pathway appears ready for merging.

### Tag behavior debate unfolds

A discussion about improving tag conflict handling produced three distinct proposals: showing more detailed error messages when tags exist (original suggestion), keeping the current simple error (Junio's preference), or making tag creation idempotent for matching commits (Hilco Wijbenga's alternative). The thread evolved to consider technical nuances around annotated versus lightweight tags, with consensus that any idempotency solution would need to handle tag metadata differences. While no resolution emerged today, the exchange surfaces important considerations about Git's safety-vs-convenience tradeoffs and the principle of least surprise for destructive operations.

### Memory leak fixes finalized

Lidong Yan's memory leak fix in revision walking code (`prepare_show_merge()`) gained comprehensive test coverage in v2, creating a realistic merge conflict scenario to verify the fix. The test uses a three-branch setup (base, hello, goodbye) with conflicting changes to exercise `git show --merge` behavior during merge states. Junio provided final style nits on the test script, ensuring it meets Git's formatting conventions. This patch exemplifies how test design often evolves from minimal verification to more comprehensive scenarios, particularly for merge-related functionality.

## In brief

**Git v2.50.0-rc2 released** -- Junio announces the second release candidate with 604 non-merge commits from 85 contributors, featuring new `--combine-cruft-below-size` for repack`, TCP keepalive for HTTP, and machine-parsable `rev-list` output.

**macOS test prerequisites fixed** -- Ramsay Jones corrects missing test setup in `t/test-lib.sh` for Darwin systems, restoring POSIXPERM, BSLASHPSPEC and EXECKEEPSPID flags accidentally omitted in commit d3d8c601fd.

**Interactive rebase state cleanup** -- Øystein Walle fixes an edge case where invalid `rebase.instructionFormat` could leave repositories in half-initialized states, reordering operations to validate formats before state setup.

**Force-push confirmation proposal** -- Aditya Garg suggests interactive confirmation for `git push -f`, but Junio pushes back, arguing safety mechanisms should be shell-level rather than command-specific.

**Documentation link fix** -- Kristoffer Haugsbakk corrects a malformed `linkit:` reference to proper `linkgit:` syntax in maintenance documentation, verified via grep search.

## On the radar

**Reftable test migration** -- Patrick Steinhardt's ongoing effort to migrate reftable tests to the `clar` framework appears in Junio's "What's cooking" as making steady progress.

**Object store cleanup** -- The long-running effort to remove `the_repository` global continues advancing, with multiple patches in the pipeline according to today's status report.