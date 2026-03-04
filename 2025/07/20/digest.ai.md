# Git Mailing List Digest - 2025/07/20

**The day in brief.** A moderately active Sunday with 16 emails across 10 threads, featuring several technical refinements nearing completion. Key developments include final polish on the `pull.autoStash` config option, a new IMAP archiving feature for `git send-email`, and ongoing discussions about Rust integration licensing and Windows compatibility. The day also saw productive new contributor engagement and a thought-provoking proposal about Git's future in AI-scale development.

## Notable threads

**`pull.autoStash` configuration finalized**  
The second iteration of the `pull.autoStash` patch series addresses all remaining review feedback from Junio Hamano, establishing clear precedence rules for autostash behavior in `git pull`. The implementation now properly handles three configuration levels: command-line flags take highest priority, followed by the new `pull.autoStash` config, falling back to operation-specific settings (`rebase.autoStash` or `merge.autoStash`). With comprehensive test coverage (60+ new test lines) and improved documentation explaining the rebase/merge differences, this appears ready for final integration.

**IMAP folder support for sent emails**  
Aditya Garg introduces a new feature allowing `git send-email` to archive sent messages in IMAP folders, addressing limitations of providers like iCloud Mail that don't save SMTP-sent messages. The opt-in implementation pipes messages to `git imap-send` after successful SMTP delivery, preserving all headers (including Bcc) and batching them for efficient storage. The well-documented patch adds both config (`sendemail.imapfolder`) and command-line (`--imap-folder`) options while maintaining backward compatibility. This solves a real workflow pain point for users of certain email providers.

**Rust xdiff integration faces licensing questions**  
The technical discussion around Rust xdiff optimization takes a policy turn as Phillip Wood raises concerns about license compatibility between Git's GPL/LGPL code and Gitoxide's MIT/Apache dual-license. Wood argues that allowing Rust code to be used under different terms could effectively relicense Git without proper process, particularly problematic for LGPL components like xdiff. Meanwhile, Johannes Schindelin and Matthias Aßhauer continue addressing Windows build issues, with Aßhauer providing detailed guidance on target platforms and version compatibility. The thread now balances technical progress against important licensing considerations.

**AI-scale version control proposal**  
Skybuck Flying's speculative email sparks discussion about Git's suitability for massive parallel AI development, identifying challenges around concurrent agents, large binary assets, and semantic versioning. The proposal suggests both short-term integrations with MLOps tools (DVC, MLflow) and long-term architectural changes for AI-native version control. While lacking concrete patches, the email frames an important future-looking conversation about version control requirements in an increasingly automated development landscape.

## In brief

**Submodule configuration safety** -- K Jayatheerth proposes modifying the `--force` behavior in submodule path reuse to completely replace existing submodules rather than auto-incrementing names, addressing D. Ben Knoble's UX concerns.

**`git repo` command cleanup** -- The series simplifies subcommand parsing by removing redundant validation now handled by parse-options infrastructure, maintaining the same user-facing behavior with cleaner code.

**New contributor progress** -- Eric Frederickson transitions from initial inquiries to active microproject work, identifying a specific `git stash apply` issue affecting multiple test cases as his first contribution target.

**Bisect output clarification** -- Christian Couder authoritatively explains `git bisect skip` behavior, noting the reporter likely misused the command and that Git properly excludes known-good commits from "could be bad" output.

## On the radar

**Windows Rust compatibility** -- Matthias Aßhauer's detailed analysis of Windows build requirements for Rust xdiff integration suggests this may need careful version targeting, particularly for older Windows versions and non-x86 architectures.