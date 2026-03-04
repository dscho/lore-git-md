# Git Mailing List Digest — 2025/07/13

## The day in brief

A quiet Sunday with 7 emails across 4 threads, primarily focused on platform compatibility issues. The most notable developments include a resolution to the macOS PCRE2 build configuration debate (now in its third iteration) and successful workarounds for building Git on the unsupported RHEL 6 platform. The line ending behavior discussion reached a clear conclusion about Git's intentional preservation of original line endings during checkout.

## Notable threads

**PCRE2 build configuration refined for macOS** — Carlo Marcelo Arenas Belón's v3 patch (following Junio's terminology clarification request) finalizes the meson build system's handling of PCRE2 dependencies on macOS. The solution now defaults to auto-detection globally while specifically disabling PCRE2 on macOS unless explicitly enabled via a new `macos_workaround_system_pcre2` option. This narrowly targets the problematic case where macOS's system-provided PCRE2 appears available but lacks necessary headers, preserving functionality for properly configured installations. The change aligns meson's behavior with the Makefile's opt-in model while maintaining PCRE2 support for `git grep --perl-regexp` when correctly set up.

**RHEL 6 build workaround confirmed** — Following collaborative debugging efforts led by Ramsay Jones, a contributor confirmed successful compilation on RHEL 6 using two key fixes: adding `#include <ctype.h>` to `sane-ctype.h` and disabling the CSPRNG subsystem via `CSPRNG_METHOD=`. The thread documents clear warnings about this being outside Git's supported platform matrix (RHEL 8+ is the risks of mixing glibc headers, and the security implications of disabling modern random number generation features. The working solution provides a stopgap for users stuck on legacy systems while strongly recommending upgrades.

## In brief

**Line ending behavior clarification** — Lidong Yan and Jason Cho's exchange conclusively establishes that `core.eol=lf` only affects line ending conversion during staging (`git add`), not checkout, explaining why committed CRLF files maintain their endings. The thread documents Git's intentional design choice to preserve original line endings during checkout unless overridden by attributes.

**Build system configuration precedence** — Carlo Marcelo Arenas Belón clarifies that RHEL 6 workarounds need to be written to `config.make` (not `config.mak`) to properly override default settings, providing the correct sequence for applying legacy system modifications.