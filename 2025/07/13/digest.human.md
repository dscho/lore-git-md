# Git Mailing List Digest — 2025/07/13

## The day in brief

A quiet Sunday with 7 emails across 4 threads, primarily focused on platform compatibility issues. The most notable discussions clarified line ending behavior in Git (confirming `core.eol=lf` only affects staging, not checkout) and refined the meson build's PCRE2 handling for macOS. A RHEL 6 build issue reached resolution with a documented workaround.

## Notable threads

**Line ending behavior clarification** — Lidong Yan and Jason Cho concluded their discussion about `core.eol=lf` behavior, confirming this setting only normalizes line endings during staging (`git add`) while preserving original line endings during checkout. This explains why Jason's license header file maintained CRLF endings despite the configuration. The exchange documents a key limitation in Git's line ending handling architecture that tooling must account for, though no changes to Git itself were proposed.

**PCRE2 handling in meson build** — Carlo Marcelo Arenas Belón's v3 patch refined the meson build's PCRE2 configuration, changing the default to 'auto' globally while specifically disabling PCRE2 on macOS unless explicitly enabled via a new `macos_workaround_system_pcre2` option. This targets macOS's broken system PCRE2 headers while preserving functionality on other platforms. Junio requested minor terminology cleanup around "wrap" references in the option description, suggesting the technical approach is sound but needs clearer documentation.

## In brief

**RHEL 6 build workaround** — A user confirmed successful compilation on RHEL 6 using `CSPRNG_METHOD=` and a `ctype.h` include fix, though the thread reiterated this platform is unsupported. The solution documents a viable path for legacy systems despite Git's official RHEL 8+ support policy.

**Build configuration precedence** — Carlo Marcelo Arenas Belón clarified that RHEL 6 workarounds must use `config.make` (not `config.mak`) to properly override defaults, detailing the build system's configuration file hierarchy.