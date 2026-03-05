Here's the Git mailing list digest for September 16, 2025:

## The day in brief

A busy day with 112 emails across 30 threads, featuring significant discussions around stash behavior improvements, Rust infrastructure design, and color handling refactoring. Key highlights include the approval of configurable index restoration for `git stash`, ongoing debates about Rust build system architecture, and a comprehensive color handling cleanup series.

## Notable threads

**Stash gets configurable index restoration**  
D. Ben Knoble's series adding a `stash.index` config option was approved after multiple iterations. The change makes `stash apply/pop` restore the index by default when configured, addressing a long-standing inconsistency where `stash push` saves staged changes but `apply/pop` doesn't restore them. The final version uses runtime configuration rather than compile-time breaking changes, with thorough test coverage and positive reviews from Phillip Wood and Junio Hamano. This resolves a common user pain point while maintaining backward compatibility.

**Rust infrastructure design debate continues**  
Ezekiel Newren raised significant concerns about Patrick Steinhardt's Rust implementation approach, particularly around build system integration and Windows support. The discussion revealed tensions between an MVP-focused strategy and Newren's vision for a more modular, Cargo-first architecture. Ramsay Jones added practical concerns about toolchain conflicts on Linux Mint. While the core infrastructure is approved, substantial redesign work remains to address platform support and build system integration.

**Color handling refactoring**  
Jeff King submitted a 13-part series standardizing Git's color handling code, replacing numeric constants with proper enums and fixing subtle bugs in grep and diff colorization. The changes improve type safety and incidentally fixed performance issues in non-coloring cases. Patch 13 explores stronger type safety through struct wrappers as a proof-of-concept, though the approach's syntactic overhead may limit its practicality. The series represents a systematic cleanup of color-related code across 32 files.

**Shallow clone limitations in CI workflows**  
Philipp Hahn documented real-world pain points with `--shallow-exclude` in GitLab CI pipelines, where temporary merge commits break branch-based exclusion. The thread explored protocol limitations and converged on blobless clones as a pragmatic workaround. D. Ben Knoble and others provided technical analysis of the edge cases, highlighting how Git's shallow clone semantics interact with CI pipeline topologies.

**ODB transaction API finalized**  
Justin Tobler's series refactoring object database transactions reached completion, establishing clean interfaces in odb.[ch] while relocating implementation to object-file.[ch]. The changes simplify transaction nesting behavior and improve API clarity, with particular attention to `update-index --verbose` edge cases. The work enables future pluggable ODB backends while maintaining files-backend compatibility.

## In brief

**Tig 2.6.0 released** -- Thomas Koutcher announced a significant update to the Git text-mode interface with improved blame handling, diff display, and Unicode support.

**`git push --mirror` documentation** -- Gustavo Velasco-Hernández clarified the behavior in the man page, enumerating all three operations performed during mirror pushes.

**`git add -p` hunk splitting** -- Phillip Wood and Junio debated behavior changes, with Junio rejecting version gating in favor of either unconditional fixes or runtime configuration.

**`git send-email` Reply-To fix** -- NeilBrown addressed a duplicate header issue when using both `--compose` and `--reply-to` options.

**HP-UX pthread detection** -- A bugfix corrects threading support detection on HP-UX systems where gettext's transitive dependencies masked pthread linkage requirements.

**SOCKS proxy test failure** -- Brian Carlson reported a test failure in t5564 where libcurl 8.16.0 appears to misinterpret socks4:// URLs as HTTP proxies.

## On the radar

**Command deprecation framework** -- Kristoffer Haugsbakk's series improving messaging for deprecated commands like `whatchanged` needs minor adjustments before final approval.

**`git-repo-info` interface** -- Debate continues about whether the experimental command should output all fields by default or require an explicit `--all` flag.

**Worktree safety checks** -- Junio maintains that detached HEAD operations are safer than branch checkouts for temporary work, despite Gabriel Scherer's arguments about recoverability.