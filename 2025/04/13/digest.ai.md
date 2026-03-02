# Git Mailing List Digest — 2025/04/13

**The day in brief.** A quiet Sunday with 11 emails across 6 threads**,** featuring minor test suite fixes, a bug report about `git difftool` behavior, and an emerging discussion about GPG signing in the email workflow. The most notable development is the technical exploration of why commit signatures don't work with patch-based workflows and what alternatives might exist.

---

## Notable threads

### GPG signing limitations in patch email workflows

Klaus Frank's inquiry about signing patches via `git send-email` sparked a technical discussion about fundamental constraints in Git's email workflow. Matt Hunter explained that commit signatures become invalid when patches are applied via `git am` because the committer identity changes. Brian m. carlson expanded on this, noting that while PGP/MIME signing would technically work, Git lacks the infrastructure to verify such signatures. The thread reveals a long-standing gap between the desire for authenticated patches and Git's current capabilities, with potential solutions ranging from MIME support to alternative header-based approaches. This discussion may lead to future proposals, though historical community preferences have favored patch quality over authentication.

### `git difftool` loses meld file operations

A new report documents unexpected behavior when using `git difftool --tool=meld -d`, where files copied during the visual diff session disappear rather than persisting in the working directory. The response clarifies this is intentional - the `-d` flag operates on temporary directories that Git cleans up after the session. While this safety mechanism prevents accidental changes, it creates a confusing experience for users expecting GUI operations to affect their working copy. The thread suggests improving documentation to make this limitation clearer, rather than changing the underlying behavior which serves as a safety feature.

---

## In brief

Philippe Blain's performance test fixes gained a third commit correcting documentation in p7821-grep-perl.sh, completing this small series that's already queued in 'next'. The edge case behavior of empty prerequisites in grep performance tests was confirmed correct, with Patrick Steinhardt verifying the implementation handles all expected scenarios.

Koji Nakamaru provided a straightforward solution for pre-loading GitHub PATs using `git credential approve`, addressing Lukas Michael's credential management question on macOS. The multi-remote synchronization discussion saw D. Ben Knoble reiterating the local-repository-as-source approach, though it remains at odds with stateless CI requirements.