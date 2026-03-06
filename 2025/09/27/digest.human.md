# Git Mailing List Digest - 2025/09/27

**The day in brief.** A moderately busy Saturday with 32 emails across 8 threads, featuring significant progress on the `git repo stats` series and thoughtful discussion about Git's patch workflow. The standout items are the v4 submission of repository statistics functionality and Taylor Blau's proposal to formalize review requirements before merging.

## Notable threads

### Repository statistics command reaches v4

Justin Tobler submitted the fourth iteration of the `git repo stats` series, introducing comprehensive repository health reporting functionality. The command now includes reference counting, object type statistics, machine-readable output formats (keyvalue and NUL-delimited), and progress reporting. Key improvements in this version address i18n concerns by using `utf8_strwidth()` for proper display width calculation in table formatting. Junio Hamano raised important design questions about whether the tool should focus more on actionable metrics (like delta chains) rather than historical statistics, though the technical implementation appears sound. The series has progressed through multiple review rounds and now awaits final documentation polish before potential merging.

### Rust config handling adopts FFI approach

A series implementing boolean configuration parsing in Rust's `libgit-rs` transitioned to using FFI-wrapped C functions (`git_configset_get_bool()`) as directed by maintainers. The v3 submission expanded scope to include `get_ulong()` and `get_pathname()` methods following the same pattern. Junio provided feedback on patch organization, suggesting the documentation fix (PO README formatting) should be split out and the boolean implementation presented as a single clean patch rather than recording review iterations. The technical approach now matches project requirements by leveraging Git's native config parsing through FFI rather than Rust reimplementation.

### Workflow optimization discussion continues

Taylor Blau engaged with Junio Hamano's RFC about Git's patch workflow, proposing concrete changes to Documentation/SubmittingPatches that would require at least one positive "ack" from another contributor before merging. The response thoughtfully analyzed two maintainer expansion models (quorum vs subsystem maintainers) while expressing cautious skepticism about shortening the 7-day 'next' branch waiting period. This discussion represents an important step toward formalizing review practices as the project scales.

## In brief

**Documentation formatting fixes** -- Jean-Noël Avila addressed an Asciidoctor rendering issue in config/extensions.adoc and pretty-formats.adoc, working around a bug that incorrectly renders "+" characters after nested lists.

**Worktree directory visibility** -- Junio Hamano explained that worktree directories appear as untracked by design, treating them like any nested repository. A follow-up discussion highlighted an inconsistency in how `.git` files are handled between current and sibling worktrees.

**Credential wildcard matching** -- A feature request proposed adding partial URL matching for Git credential configuration, suggesting wildcard syntax to distinguish between path prefixes under the same host.

**Clang-format adjustment** -- A previously merged change to `.clang-format` (preventing unwanted spaces after control macros) was recognized as belonging to a separate topic from the repo stats series it originally accompanied.