# Git Mailing List Digest - 2025/08/27

**The day in brief.** A busy Wednesday with 74 emails across 20 threads, dominated by major discussions around Rust adoption policies, the long-planned default branch name change to "main", and improvements to deprecation messaging. Key developments include Junio Hamano weighing in on platform support concerns for Rust, the finalization of the `core.commentChar=auto` removal, and productive refinements to user guidance for deprecated commands.

## Notable threads

### Rust adoption and platform support debate intensifies

The ongoing discussion about introducing Rust into Git's core saw significant policy-level exchanges today. Taylor Blau and Junio Hamano debated the long-term maintenance implications of Rust adoption, particularly around security backporting and platform support. Randall Becker from HPE NonStop provided crucial context about their volunteer-based maintenance model, revealing that some "commercial" platforms rely on individual contributors rather than vendor teams. Junio expressed skepticism about maintaining dual C/Rust implementations indefinitely, while Blau argued Rust's security benefits may reduce backport needs. This fundamental debate about tradeoffs between security and portability remains unresolved but has reached clearer articulation of positions.

### Default branch name change reaches final" stage

Phillip Wood's 6-part series to change Git's default branch name from "master" to "main" (when built with WITH_BREAKING_CHANGES) saw its final implementation patch posted today. The changes update refs.c and documentation while maintaining backward compatibility. Junio approved the core implementation but questioned removing GIT_TEST_DEFAULT_INITIAL_BRANCH_NAME support, suggesting the test infrastructure should remain for potential future naming changes. The series represents the culmination of planning since 2020's deprecation warning (commit 675704c74dd) and aligns with major Git forges' defaults.

### `core.commentChar=auto` removal finalized

After months of discussion, Phillip Wood's series removing the problematic `core.commentChar=auto` configuration was merged today. The final implementation includes sophisticated deprecation warnings that detect config locations and provide tailored unset commands or custom choice options. Oswald Buddenhagen's concerns about over-engineering the solution were addressed through implementation compromises rather than wholesale rejection. The new deprecation framework may serve as a model for future configuration changes, balancing verbose user assistance against implementation simplicity.

### Improved guidance for deprecated commands

Kristoffer Haugsbakk's series to improve user experience around deprecated commands like `git whatchanged` saw extensive discussion today about error message wording. The thread refined how to guide users to alternatives while minimizing unnecessary maintainer emails. Junio ultimately endorsed Eric Sunshine's phrasing that explicitly ties contact requests to unsuccessful replacement attempts. The series combines URL generation, documentation updates, and clear `git log` equivalents to provide more actionable guidance than the previous passive "email us" approach.

### SMTP autoconfiguration security concerns addressed

Aditya Garg responded to Brian Carlson's security and sustainability concerns about the proposed SMTP autoconfiguration feature that queries Mozilla's ISPDB service. Garg is seeking explicit permission from Mozilla for long-term integration and has agreed to implement proper SSL verification in HTTP::Tiny for older Perl versions. These were the final blockers for a feature that otherwise appears technically ready, with the thread now awaiting Mozilla's response.

## In brief

**Sparse-checkout refactoring** -- Ayush Chandekar reports progress on moving sparse-checkout configuration variables from globals to `struct repository`, following Derrick Stolee's phased approach recommendation.

**SQLite proposal for SHA-1/SHA-256 mapping** -- Eric Wong suggests SQLite as an alternative to existing proposals for storing bidirectional hash mappings, citing its mature transaction support and mmap capabilities.

**Sparse index optimization relevance confirmed** -- Derrick Stolee notes the `git ls-files` sparse index optimization has proven useful for `git mergetool` performance, showing broader applicability than initially expected.

**Slab allocator API refactoring** -- Junio Hamano and ノウラ | Flare propose renaming and restructuring Git's slab allocator interface to prevent NULL pointer dereferences by combining clearing and freeing operations.

**Hostname-based config includes** -- Ayush Sharma's proposal for hostname-based conditional config includes receives initial review feedback emphasizing the need for documentation and tests.

**`git-push` documentation improvements** -- Julia Evans continues refining her `git-push` man page updates, with Junio Hamano approving her restructuring of the "where to push" section while discussion continues on refspec explanations.

## On the radar

**Rust compatibility debate** -- Brian Carlson's response in the Rust thread argues strongly against making Rust optional, citing security benefits and project direction, setting up a fundamental policy debate.

**Commit comments proposal** -- Rune Philosof's suggestion for commit comment functionality is being steered toward existing `git-notes` capabilities by Junio and others, though the original proposer hasn't yet responded.