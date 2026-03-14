Here's the daily digest for March 13, 2026:

**The day in brief.** A busy day with 121 emails across 40 threads, featuring significant progress on several fronts. Key highlights include final approvals for the `git replay --revert` feature and `git fast-import` signature handling, ongoing design discussions about `git checkout` autostash behavior, and multiple `the_repository` removal patches nearing completion. The day also saw Junio's "What's cooking" report and several new contributor submissions.

**Notable threads**

**`git replay --revert` reaches final form**  
Siddharth Asthana's series adding revert capability to `git replay` is now technically complete after Junio Hamano reviewed the final message formatting consolidation. The implementation builds on established patterns from `sequencer.c` while addressing all review feedback from Phillip Wood and Patrick Steinhardt. While higher-level interface questions (subcommands vs flags) remain open, the core functionality is ready for merging, particularly useful for GitLab's Gitaly service.

**Histogram diff bugfix approved**  
Yee Cheng Chin's fix for redundant output in the diff algorithm's compaction phase has been approved by both Junio Hamano and Phillip Wood after multiple review rounds. The patch addresses an edge case affecting <1% of diffs but meaningfully improves output quality when triggered. The solution uses `xdl_fall_back_diff()` to re-process shifted groups while preserving all diff flags.

**`pre-add` hook design reconsidered**  
Despite technical completion, Chandra Kethi-Reddy's `pre-add` hook series faces fundamental questions about whether its limited scope provides enough value. Phillip Wood raises concerns about dry-run behavior and submodule handling, while the maintainer remains skeptical about the concept's utility. The thread shows how even well-implemented features may stall if the core premise is questioned.

**Subtree recursion fix awaits review**  
Junio has pinged the list for reviewers on Colin Stagner's patch fixing `git subtree split` failures on Debian/Ubuntu systems with shell recursion limits. The changes appear sound but need validation from actual subtree users before merging. This highlights how niche features sometimes struggle to get attention even for clear fixes.

**Cover letter formatting finalized**  
Mirko Faina's cover letter formatting feature concludes with Junio accepting a compromise on format string prefixes - the "log:" prefix remains technically required but can be omitted when using %-placeholders, following `git log --pretty`'s DWIM behavior. The eight-iteration series demonstrates Git's careful interface design process.

**`git fast-import` signing improvements merged**  
Junio has merged Justin Tobler's series adding `sign-if-invalid` mode to `git fast-import` after addressing final review feedback about test branch references and warning messages. The changes consolidate signing infrastructure while maintaining compatibility with OpenPGP, X.509, and SSH signatures.

**In brief**  
**Upload-pack performance series** -- Patrick Steinhardt's 10-part series reducing write contention in `git-upload-pack` is approved, demonstrating 40% faster clones by batching writes.

**ODB counting abstraction** -- Patrick Steinhardt's object counting interface refactoring receives final approval from Toon Claes, completing another piece of the ODB abstraction effort.

**`git maintenance list` command** -- Rémy Léone's new subcommand for viewing registered maintenance repos gets code review feedback about config handling duplication.

**`git apply -p` validation** -- Post-merge discussion continues about Windows test portability in Mirko Faina's argument parsing fix, with consensus forming around inline test data.

**Partial clone test modernization** -- Siddharth Shrimali's hash-independent test helper improvements get final approval after addressing shell portability concerns.

**`the_repository` removal progress** -- Multiple patches advance the effort: Burak Kaan Karaçay's `run-command.c` changes are merged, while Phillip Wood targets worktree functions and Tian Yuchen addresses `mktree`.

**Reference transaction hooks extended** -- Eric Ju adds a "preparing" phase to coordinate distributed reference updates before locking, approved by Junio with minor i18n refinements pending.

**On the radar**  
**Rustification effort** -- Ezekiel Newren's work continues in the background, though platform support questions from Randall Becker remain unresolved.

**SHA-256 interoperability** -- brian m. carlson's work progresses per Junio's "What's cooking" report, now targeting Git 3.0 without being a blocker.

**`git checkout` autostash behavior** -- The design discussion continues about whether to unify with `-m` or keep as separate features, with submodule handling emerging as a key concern.