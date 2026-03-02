# Git Mailing List Digest - 2025/04/24

**The day in brief.** A moderately busy Thursday with 61 emails across 18 threads, dominated by ongoing discussions about `git send-email` OAuth2 support and Windows-specific issues. Key developments include consensus forming around credential helpers for OAuth2 tokens, new Windows lock file problems surfacing, and signature handling improvements hitting test failures.

## Notable threads

**Windows lock file mysteries deepen** -- Johannes Schindelin and Jörg Hohwiller continue investigating spurious lock file errors in Git for Windows 2.48.1+, now confirmed as distinct from the known ReFS regression. New symptoms include GUI client freezes and "failed to run reflog" errors during garbage collection. The thread has progressed from initial hypothesis testing to confirmation of a persistent issue, though root cause analysis remains blocked without additional system diagnostics. Workarounds remain manual and impractical for regular workflows.

**OAuth2 support converges on credential helpers** -- After five iterations of the `git send-email` OAuth2 series, the discussion has shifted away from a new `smtp-passeval` option toward using Git's existing credential helper system for dynamic token generation. Junio Hamano strongly endorsed this approach while highlighting documentation gaps that led to the original workaround proposal. The thread also saw progress on the Authen::SASL Perl module integration, with Erik Huelsmann identifying a critical bug in the OAuth2 implementation that was causing authentication failures.

**Signature handling improvements hit snags** -- Christian Couder's follow-up to the recently merged signed commit support in fast-import/export encountered test failures in t9350, prompting Junio to temporarily revert the changes from 'seen'. The discussion revealed deeper questions about how to properly represent the relationship between signature protocols (OpenPGP/X.509/SSH) and hash algorithms (SHA-1/SHA-256), with Brian m. carlson noting the current approach conflates these concepts. Junio reinforced that backward compatibility applies even to unreleased features in 'master', setting clear expectations for any resolution.

**Meson benchmark integration finalizes** -- The long-running discussion about integrating Git's performance benchmarks with Meson reached consensus on handling the `time` command dependency. Patrick Steinhardt's proposal for configurable behavior (`-Dbenchmarks=auto/enabled/disabled`) earned Junio's approval, balancing developer convenience with strict checking when explicitly requested. The solution defaults to silent fallback when `time` is missing but errors when benchmarks are explicitly enabled, addressing Christian Couder's concerns about dependency visibility.

**In brief**

**Windows HTTP/2 regression** -- Lars Eriksen reports Git for Windows 2.49.0 lost HTTP/2 support due to SSL backend changes, breaking corporate authentication. Johannes Schindelin identified the cause as Secure Channel vs OpenSSL backend selection and suggested `http.sslBackend=openssl` as a workaround.

**Shell path detection in Meson** -- Patrick Steinhardt proposed preferring `/bin/sh` over PATH lookups in Meson builds to solve cross-compilation issues. Junio suggested aligning with Makefile's `SHELL_PATH` variable for consistency during the build system migration.

**CI dependency handling** -- Two patches from Junio make external dependency downloads (JGit, Git-LFS, Perforce) optional in CI scripts, addressing an Eclipse repository outage. The changes add proper cleanup and warning messages when downloads fail.

**On the radar**

**Symlinks in .git internals** -- A new thread probes whether Git should officially support symlinking `.git` subdirectories, prompted by Git LFS compatibility issues with Android's `repo` tool. Junio clarified that while symlinking individual subdirectories (like `objects/`) is an existing practice, symlinking the entire `.git` directory isn't intended to be supported. The discussion may lead to clearer project policy or warnings.