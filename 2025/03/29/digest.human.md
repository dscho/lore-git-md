# Git Mailing List Digest — 2025/03/29

**The day in brief.** A moderately active Saturday with 33 emails across 20 threads, featuring final polish on several long-running series. Key developments include resolution of the reftable decoupling effort, final approval for the comma operator removal series, and continued investigation of Azure DevOps push hangs. Documentation refinements and GSoC proposal discussions rounded out the day.

## Notable threads

**Reftable decoupling ready for merging**  
Patrick Steinhardt confirmed the reftable library decoupling series (ps/reftable-sans-compat-util) is ready for merging after resolving Windows-specific build issues. The series systematically removes Git-specific dependencies from reftable code to improve portability, a key step in the ref backend abstraction work. Johannes Schindelin verified the MinGW-specific issues were resolved by moving mimalloc override declarations to `compat/posix.h`. Junio Hamano indicated he will now mark the topic for promotion to 'next', concluding several weeks of discussion and testing.

**Azure DevOps push hangs investigation deepens**  
New packet-level evidence emerged in the ongoing investigation of `git push --mirror` hangs with Azure DevOps. Akash S provided traces showing the server intermittently fails to send the required "git flush message 0000" packet during unsuccessful pushes. Brian M. Carlson confirmed this appears to be the root cause, noting the protocol requires this final flush packet. The discussion shifted from timeout configurations to protocol-level behavior, suggesting this is a server-side compliance issue rather than a client-side timeout handling problem. Microsoft support had previously attributed the issue to rate limiting, though standard timeout configurations failed to make Git abort when hung.

**Comma operator removal series approved**  
Junio Hamano gave final approval to the refactoring series eliminating comma operators from Git's codebase after Phillip Wood's positive review. The v3 changes addressed all technical concerns across multiple subsystems including networking, porcelain commands, diff algorithms, and test infrastructure. Notable refinements included portable sed syntax for compiler detection, style adjustments in diff-delta.c and wildmatch.c, and meson build integration for -Wcomma warnings. The series now covers all platforms including a fix for Clang/CUDA edge cases, with explicit exceptions for imported regex code.

**GSoC proposal refinements**  
Two GSoC proposal threads saw significant refinement today. Ayush Chandekar adjusted their `the_repository` removal proposal based on Shejialuo's feedback, moving from broad structural changes to a more incremental variable-by-variable strategy. Meanwhile, Zheng Yuting shared an updated draft for consolidating ref-related functionality into `git-refs`, now incorporating prior feedback about implementation approach and test strategy. The proposal outlines eight new subcommands with detailed timelines and maintains backward compatibility through preserved test suites.

## In brief

Documentation formatting for `git-update-ref` was finalized, with Junio Hamano implementing Jean-Noël Avila's suggested manpage corrections to properly show alternative command invocations. The `git blame` porcelain output fix received final polish with function renaming and output optimization before merging. A vimdiff mergetool patch adding REMOTE target support gained maintainer approval after D. Ben Knoble's positive review. The MyFirstContribution tutorial update series saw a polite follow-up from Jayatheerth K about its status in the maintainer's queue. A pre-commit hook documentation fix for merge-file examples was queued after confirming it matches existing patterns in other documentation files.

## On the radar

The SHA implementation reporting series faces new questions about naming conventions and code structure from Junio Hamano, who suggested using raw symbol names rather than human-readable labels in `git version --build-options` output. The `ib/diff-S-G-with-longhand` series adding descriptive alternatives to `-G`/`-S` options remains in limbo as D. Ben Knoble seeks guidance on how to properly shepherd the patches forward given sign-off questions.