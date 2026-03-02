# Git Mailing List Digest — 2025/04/09

**The day in brief.** A moderately busy Wednesday with 34 emails across 12 threads, dominated by philosophical debates about Change-ID metadata and several test improvements nearing completion. The most consequential discussion centered on whether Git should record Change-IDs in commit headers, while multiple test fixes reached final implementation stages.

## Notable threads

**Change-ID standardization debate intensifies**  
The multi-day discussion about standardizing Change-IDs reached its philosophical peak today with exchanges between Nico Williams, Theodore Ts'o, and Junio Hamano. Junio invoked Git's historical design principles against recording metadata, drawing parallels to Linus Torvalds' early decisions about rename tracking. Nico countered that explicit metadata captures valuable user intent, though he retracted his earlier analogy to rename heuristics after Eric Sunshine provided historical evidence. The thread remains at an impasse between workflow utility and Git's content-addressable design philosophy, with Junio suggesting any solution must maintain semantic consistency across all Git projects rather than offering per-project configuration.

**Submodule test printf formatting finalized**  
After several rounds of review, Subhaditya Nath's fix for POSIX-compliant printf usage in t7422-submodule-output.sh reached consensus. The thread settled on using "%d" formatting for both submodule name and path components, requiring the loop counter to be passed twice. Junio Hamano and Eric Sunshine agreed this approach best maintained consistency with the test file's existing style, despite some redundancy. The change affects only test setup code while improving reliability for testing submodule status --recursive behavior.

**Rebase header propagation design debate**  
Phillip Wood's proposal to preserve commit headers during rebase operations faced pushback from Junio Hamano, who rejected the idea of configurable header propagation as undermining commit object integrity. Junio argued standardized headers should either always or never propagate, with no per-project customization. Phillip pointed to GitHub's successful 18-month use of similar functionality via git replay as real-world validation. The discussion has shifted from implementation details to fundamental questions about Git's metadata handling philosophy during history operations.

## In brief

The git-p4 encoding fix received real-world validation when Fahad Al-Rashed confirmed the test case passes in a production Python 3 environment, clearing the last technical hurdle for Nikolay Shustov's series.

Christian Fredrik Johnsen's typo fix in refs.c (correcting "checking checking" to "checking") completed review and will be queued, affecting only documentation.

Anthony Wang's Perforce test improvements are nearly ready, with final questions about removing legacy git tag commands now that verification uses git show-ref --verify.

Philippe Blain raised workflow concerns about rapid merges to 'next' in the perf-test-fixes series, prompting Junio to suggest better communication mechanisms for part-time contributors.

Elijah Newren provided post-merge feedback on Patrick Steinhardt's object-file refactoring, praising the architectural improvements while noting minor commit message nits.

## On the radar

The git-gui bug report about handling '#' characters in commit messages may relate to a recently fixed issue, as noted by Eric Sunshine. The original bugreport attachment will be needed to confirm whether this is a regression or new edge case.

A Windows-specific certificate verification issue surfaced in Brazilian Portuguese locales when running git update-git-for-windows, showing CRYPT_E_NO_REVOCATION_CHECK errors with corrupted Portuguese text in the message. The report was redirected to the Git for Windows issue tracker.