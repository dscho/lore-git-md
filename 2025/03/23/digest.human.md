# Git Mailing List Digest — 2025/03/23

**The day in brief.** A moderately active Sunday with 24 emails across 15 threads, featuring security hardening for git-shell command overrides, final polish on several near-ready patches, and ongoing discussion about `git blame` porcelain output. The most notable developments include Junio Hamano's detailed review of the git-shell security model and a resolved discussion about SMTP error handling in `git-send-email`.

## Notable threads

### Security model finalized for git-shell command overrides

Ayman Bagabas's series allowing git-shell built-in commands to be overridden by scripts in git-shell-commands reached its third iteration with security-focused changes. The discussion, involving Chris Torek and others, centered on hardening permission checks — replacing `F_OK` (file existence) with `X_OK` (executable permission) in the `access()` call. While concerns about TOCTOU race conditions were noted, the consensus accepted this as consistent with git-shell's existing security model. The v3 patch appears ready for maintainer review after methodically addressing all security feedback.

### Junio weighs in on `git blame` porcelain format stability

Junio Hamano provided a detailed review of a proposed change to `git blame` porcelain output that would add markers for ignored/unblamable lines. While agreeing with the need to expose this information, he cautioned against modifying the commit hash line format (proposed as `?<hash>` or `*<hash>`) since many parsers rely on the strict 40-character SHA-1 format. The discussion now needs an alternative approach that maintains backward compatibility while still surfacing the new information, likely through additional metadata lines rather than modifying the fundamental record delimiter.

### SMTP error handling tests fail despite RFC compliance

Zheng Yuting's GSoC project to make `git-send-email` SMTP error handling RFC 5321-compliant hit a snag with 169/215 tests failing in `t9001-send-email.sh`. The technically sound implementation distinguishes temporary (4xx) from permanent (5xx) errors using regex-based status code parsing, but the widespread failures suggest either fundamental incompatibility with current test expectations or needed updates to test assumptions. The series remains blocked pending investigation into whether these represent necessary test updates for the new behavior or actual regressions.

## In brief

David Mandelberg submitted a final optimization for remote name completion with slashes, now with comprehensive test coverage in t9902-completion.sh, while checking if minor updates to queued patches are still acceptable. Junio Hamano noted a style nit (C++ comments) in the nearly complete MyFirstContribution tutorial modernization series. Justin Tobler's default branch advice suppression series received positive feedback from Junio for its flags-based approach and new test coverage. Eric Sunshine caught a typo in Josh Steadmon's Rust crate packaging series, which is otherwise complete. Lucas Seiki Oshiro will squash test commits in the gitconfig userdiff driver series per Patrick Steinhardt's request.

## On the radar

The bulk-checkin sign comparison discussion concluded with Junio clarifying the safety rationale for `ssize_t` to `size_t` casts. Jean-Noël Avila guided a new contributor on reviving the dormant Greek Pro Git translation. Brian M. Carlson requested reproduction details for a Perl merge conflict marker placement bug. A GSoC applicant proposed consolidating ref-related commands into `git-refs`, while another explored a dedicated repository info query command.