# Git Mailing List Digest — 2025/09/21

**The day in brief.** A moderately active Sunday with 28 emails across 6 threads, dominated by technical refinements to the xdiff refactoring series and conclusive discussions about Rust adoption constraints on NonStop platforms. Key developments include resolution of the NO/YES/MAYBE macro naming issue in xdiff and definitive confirmation that Rust cannot be made mandatory due to NonStop's technical limitations.

## Notable threads

### xdiff refactoring cleanups finalized

The preparatory xdiff refactoring series (patches 1-10) saw extensive post-merge review, with Ezekiel Newren and Phillip Wood working through the final loose ends. The most substantive resolution came on the NO/YES/MAYBE macro naming issue — the macros will move to xprepare.c and use `bool` types to avoid namespace pollution while maintaining Rust compatibility. Wood also prompted several implementation clarifications about struct access patterns and helper function rationales, all of which Newren addressed with detailed explanations about the refactoring's maintainability goals. With these questions resolved, the series appears ready for the Rust-specific changes in patches 11-17.

### NonStop Rust constraints confirmed definitive

Randall Becker provided conclusive technical details ending the possibility of Rust support on NonStop platforms. Cross-compilation attempts fail due to proprietary licensing requirements and security audit incompatibilities, while native compilation remains impossible without vendor-provided Rust tooling. Newren acknowledged these constraints while outlining his efforts to minimize Rust adoption barriers elsewhere (targeting Rust 1.63.0, avoiding Cargo, single-threaded translations). The thread effectively closes with both technical and policy implications clear: Git cannot mandate Rust without breaking NonStop compatibility, leaving the project to decide how to reconcile these competing priorities.

### curl type-safety series approved

Johannes Schindelin's three-patch series addressing curl type mismatches in ARM64 builds received maintainer approval. The changes introduce a centralized `cast_size_t_to_curl_off_t()` helper and apply it to HTTP push and IMAP operations, replacing unsafe casts with proper bounds checking. Junio Hamano acked all patches with minor documentation tweaks, noting the series follows Git's tradition of improving surrounding code when fixing issues. The changes will benefit all platforms by preventing potential truncation in large file transfers, though the immediate trigger was clang's stricter ARM64 type checking.

## In brief

**git-rebase-clean discussion deferred** — Antonio Mennillo acknowledged Elijah Newren's detailed technical critique of the semantic conflict resolution tool but deferred substantive response until Wednesday due to travel. The thread remains paused on fundamental questions about handling overlapping conflicts and Git workflow philosophy.

**Empty Vietnamese submission** — An apparent accidental email containing only an iPhone signature line ("Được gửi từ iPhone của tôi") reached the list with no content or patches. Likely a mistaken submission requiring no action.