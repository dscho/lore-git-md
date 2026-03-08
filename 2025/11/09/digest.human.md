# Git Mailing List Digest - 2025/11/09

**The day in brief.** A quiet Sunday with 12 emails across 6 threads, mostly wrapping up ongoing discussions. The most notable developments include a resolution to the long-running Git data model discussion, a bugfix for `git diff` exit codes, and a new `--committer` option proposal for `git commit`. Documentation and technical refinements dominated the day's traffic.

## Notable threads

### Git data model documentation finalized

The months-long discussion around Julia Evans' Git data model documentation (`gitdatamodel.adoc`) reached its conclusion today. Ben Knoble and Junio Hamano exchanged final thoughts on whether branch references "record" or "reference" commit IDs - a seemingly minor semantic distinction that Junio argued carries important pedagogical weight for understanding Git's design philosophy. While acknowledging the practical impact may be small, Junio made a compelling case for precise terminology that distinguishes implementation details from conceptual models. With all substantive issues resolved across six iterations, this thread appears ready for merging, marking a significant milestone in improving Git's educational documentation.

### `git diff` exit code bug fixed

A subtle but important bug in `git diff`'s exit code behavior was identified and fixed through collaborative discussion. The original issue involved `--find-copies-harder` incorrectly reporting no changes (exit code 0) when combined with `--quiet` and `--no-ext-diff`. René Scharfe's initial fix addressed the immediate symptom, but Phillip Wood raised a deeper architectural question: why perform expensive rename detection at all in quiet mode? This led to a more elegant solution that disables all rename detection when `--quiet` is specified, yielding both correct behavior and a 3.6x performance improvement in affected cases. The thread demonstrates Git's careful approach to bug resolution - addressing not just the immediate issue but examining underlying design assumptions.

### `git commit --committer` proposed

A new feature proposal adds `--committer` option to `git commit`, mirroring the existing `--author` functionality. The patch provides a symmetrical way to override committer identity without environment variables, addressing a long-standing interface inconsistency. The change includes comprehensive tests covering basic functionality, error cases, and interactions with existing features like `--amend`. While still in proposal stage, the patch follows established patterns and appears well-positioned for inclusion, offering clearer scripting capabilities for multi-identity workflows.

## In brief

**xdiff FFI documentation** -- Phillip Wood provided feedback on Ezekiel Newren's technical documentation about C/Rust FFI type safety, suggesting expansions to `char` type semantics and build system integration.

**Windows credential helper build** -- Thomas Uhle acknowledged Junio's decision to defer their Makefile standardization patch until after the 2.52.0 release, respecting Git's policy against non-urgent late-cycle changes.

**Three-way merge behavior** -- Bhavik Bavishi confirmed understanding of Chris Torek's explanation about `git cherry-pick` versus direct patch application behavior, closing that discussion thread.