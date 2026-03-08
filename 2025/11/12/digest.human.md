# Git Mailing List Digest - 2025/11/12

**The day in brief.** A busy Wednesday with 78 emails across 18 threads, featuring significant progress on multiple fronts. Key highlights include Junio Hamano's comprehensive whitespace handling series reaching its final form, resolution of the long-running Git data model documentation effort, and important security discussions around attribute macro expansion. The day also saw Git v2.52.0-rc2 released with 623 commits from 81 contributors.

## Notable threads

### Git Data Model Documentation Finalized

Julia Evans' `gitdatamodel.adoc` documentation patch series reached its seventh and final iteration after extensive review. The new man page provides a comprehensive explanation of Git's core concepts (objects, references, index, and reflogs) in accessible language while maintaining technical accuracy. The only remaining debate centered on how to define branches - whether as simple refs under `refs/heads/` or as higher-level workflow tools. Junio Hamano approved the file type presentation but suggested the branch definition might be too implementation-focused. This documentation fills a long-standing gap and represents a significant collaborative achievement.

### Whitespace Handling Series Complete

Junio Hamano's 12-patch series introducing comprehensive handling of incomplete-line whitespace errors (WS_INCOMPLETE_LINE) reached its fourth and final revision. The changes establish end-to-end support for missing terminating newlines through `core.whitespace` and `.gitattributes`, implemented across diff generation, patch application, and project-wide enforcement. The series maintains backward compatibility while adding strict but configurable newline policy controls, with Git's own codebase now enforcing these rules for C files, headers, shell scripts, and documentation. The v4 changes were minor refinements, leaving only one non-critical line counting semantic as a #leftoverbits item for potential future work.

### Security Hardening for Attribute Macros

An important security discussion unfolded around Jeff King's patch converting recursive attribute macro expansion to an iterative approach to prevent stack overflow attacks. While agreeing the technical solution worked, Patrick Steinhardt and Ben Knoble raised concerns that it merely shifted the failure mode from stack exhaustion to potential heap exhaustion. The thread revealed Git's security philosophy for resource exhaustion: let processes die when they exceed reasonable bounds, relying on OS-level protections. Consensus formed that the iterative approach was sufficient given existing protections (100MB attribute file limit) and the obscure nature of macro attacks.

### ASan Findings Addressed

Jeff King posted a 9-patch series addressing various issues found through AddressSanitizer analysis, including a memory safety fix for incremental bitmaps' name-hash lookups and safer string parsing in cache-tree and fsck. The series introduces NO_MMAP by default under ASan to improve detection reliability and enables strict string checking in tests. Review discussion surfaced architectural questions about whether to develop more generalized parsing utilities, with Junio Hamano acknowledging this as an interesting direction for future work.

### In Brief

**Submodule path encoding** -- Adrian Ratiu proposed a simpler case-folding collision detection method for submodule gitdir paths, replacing hybrid filesystem/text comparison with direct directory scanning.

**Fast-import signature stripping** -- Christian Couder clarified technical points about validation behavior in the proposed 'strip-if-invalid' mode, agreeing to restructure tests to avoid SHA-256/Rust dependencies.

**Rebase trailer support** -- Phillip Wood provided final review feedback on Li Chen's v6 series adding `--trailer` support to `git rebase`, indicating it's in good shape with only minor cleanup needed.

**Last-modified bugfix** -- Toon Claes advocated for including a `git last-modified` correctness fix in v2.52, showing it addresses inconsistent results compared to `git log`.

**Committer identity controls** -- ZheNing Hu continued advocating for a `--committer` flag in `git commit`, though maintainer skepticism about its necessity appears to be growing.

**HTTP auth fix** -- A minimal 2-line fix ensures authentication headers are included in remote-curl's probe_rpc() requests, addressing blob download failures in filtered clones.

### On the Radar

**Mixed hash repositories** -- brian m. carlson confirmed SHA1/SHA256 repository mixing is intentionally unsupported and proposed hardening `git add` to reject such cases explicitly.

**OSX keychain credential helper** -- Discussion continues about whether to revert a problematic optimization or implement a more comprehensive fix for credential storage issues.