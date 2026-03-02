# Git Mailing List Digest - 2025/04/24

**The day in brief.** A moderately busy Thursday with 61 emails across 18 threads, dominated by continued refinement of the `git send-email` OAuth2 support series and several Windows-specific bug investigations. Notable developments include consensus forming around credential helpers for OAuth2 tokens, a regression in Git for Windows' HTTP/2 support, and test failures prompting temporary reversion of signature handling changes in fast-import/export.

## Notable threads

**OAuth2 support for send-email reaches consensus**  
The fifth iteration of Aditya Garg and Julian Swagemakers' series to add OAuth2 authentication to `git send-email` saw significant progress today. After extensive discussion about dynamic token generation, the authors agreed to drop the proposed `smtp-passeval` option in favor of using Git's existing credential helper system. Junio Hamano emphasized the need for better documentation to guide users toward this solution, while Greg Kroah-Hartman raised important considerations about Outlook detection in enterprise environments. Meanwhile, technical hurdles remain in the Perl Authen::SASL module implementation, with Erik Huelsmann identifying a critical bug in the OAuth2 authentication flow that was causing failures despite correct token generation.

**Windows lock file mystery deepens**  
Johannes Schindelin and Jörg Hohwiller continued investigating spurious lock file errors in Git for Windows, now confirmed as distinct from the known ReFS drive regression. New symptoms emerged today including "fatal: failed to run reflog" errors during garbage collection, though pulls still complete successfully. The thread has progressed from initial hypothesis testing to confirmation of a persistent issue affecting both command-line and GUI clients (IntelliJ, git-fork), though root cause analysis remains blocked without additional system diagnostics. The only known workaround - manually deleting non-existent `.lock` files - remains impractical for regular workflows.

**Signature handling reverted due to test failures**  
Junio Hamano temporarily reverted recent signature algorithm improvements for `fast-import`/`fast-export` after discovering test failures in t9350-fast-export.sh. The changes, which aimed to properly identify signature types (OpenPGP/X.509/SSH) rather than defaulting to "sha1", had prompted earlier discussion about backward compatibility and algorithm representation. Brian m. carlson highlighted an unresolved conceptual issue around conflating signature protocols with hash algorithms. The revert follows Git's standard practice of maintaining test stability while investigations continue, though the discussion revealed differing views on compatibility requirements for unreleased features.

**HTTP/2 support regression in Git for Windows**  
Lars Eriksen reported that Git for Windows 2.49.0 unexpectedly dropped HTTP/2 support, breaking authentication against corporate backends requiring the protocol. Johannes Schindelin identified the root cause as a backend-specific issue - the Windows build provides both OpenSSL-backed cURL (with HTTP/2) and Secure Channel-backed cURL without it), with the observed behavior depending on which backend is configured. The thread produced a concrete workaround (`http.sslBackend = openssl`) while highlighting an opportunity to improve bug report templates by including SSL backend configuration details.

**Meson build refinements**  
Patrick Steinhardt's series to improve shell path detection in Meson builds saw productive discussion about cross-platform compatibility. The patches add reporting of detected runtime paths and modify shell detection to prefer POSIX-specified `/bin/sh` over PATH lookups, solving real-world issues at GitLab and Debian where build/target host differences caused failures. Junio Hamano suggested aligning with the Makefile build's `SHELL_PATH` environment variable approach for consistency during the build system transition, while Justin Tobler validated the technical approach while recommending clarifying comments about search order.

## In brief

**Windows certificate verification workaround** -- Johannes Schindelin provided a targeted fix for CRYPT_E_NO_REVOCATION_CHECK errors in Brazilian Portuguese locales by suggesting curl's `--ssl-revoke-best-effort` flag for Git for Windows' update script.

**Azure DevOps SSL regression** -- Schindelin advised trying `http.sslbackend=schannel` as a workaround for Git for Windows 2.44.0+ cloning failures with Azure DevOps, moving the SSL-related investigation forward.

**String-list test modernization** -- Shejialuo's series converting string-list tests from shell to C unit tests progressed with refinements to test helper interfaces based on feedback from Patrick Steinhardt and Junio Hamano.

**index-pack delta chain fix** -- Derrick Stolee and Junio Hamano discussed safety implications of relaxing cycle detection in REF_DELTA chain handling, with Hamano emphasizing the need to maintain security against malicious packfiles.

**CI dependency handling** -- Two patches made external dependency downloads (JGit, Git-LFS, Perforce) optional in CI scripts to handle temporary outages gracefully, following the pattern already established in test skipping.

## On the radar

**Symlink support policy** -- An emerging discussion about whether to officially support symlinks in `.git` internals, prompted by Git LFS compatibility issues with Android's `repo` tool, may lead to clearer project guidelines about this usage pattern.