Here's the daily digest for September 8, 2025:

## The day in brief

A busy day with 143 emails across 29 threads, dominated by ongoing discussions about Rust infrastructure, submodule path encoding, and command deprecation frameworks. Key developments include Patrick Steinhardt's v3 Rust RFC series, resolution of the promisor-remote capability extension, and continued debate about enterprise impact of the Rust transition.

## Notable threads

**Rust infrastructure RFC reaches v3** -- Patrick Steinhardt submitted the third iteration of his Rust infrastructure series (8 patches), now with improved build system integration, varint subsystem conversion, and CI testing. The series proposes a phased transition timeline culminating in mandatory Rust for Git 3.0. Discussion continues around type safety (Ezekiel Newren), build system design (brian m. carlson), and LTS support policies. Junio Hamano raised concerns about calling this a "test balloon" given known platform incompatibilities.

**Promisor-remote capability extension finalized** -- Christian Couder's 8-part series extending the promisor-remote protocol with configurable field validation received maintainer approval after addressing all technical feedback. The implementation adds server-advertised fields and client validation while modernizing string handling. Only a minor terminology debate ("field" vs "configuration variable") remains unresolved before merging.

**Enterprise impact of Rust transition** -- Randall Becker detailed strict compliance requirements (PCI, HIPAA) that demand official security backports for NonStop platforms lacking Rust support. Elijah Newren proposed designating LTS maintainers, while Becker emphasized auditors require fixes from official sources. The discussion revealed fundamental tension between Git's evolution and enterprise certification needs, with no clear resolution yet.

**Submodule path encoding series refined** -- Adrian Ratiu's v2 series implementing URL-style encoding for submodule gitdir paths addressed review feedback, adding platform-specific fixes and comprehensive test coverage. Phillip Wood and Jeff King provided detailed reviews on path length validation and config override mechanisms. The series awaits resolution on whether to add an `extensions.submoduleEncoding` version marker.

**Command deprecation framework with alias support** -- Jeff King's v3 series allowing aliases to shadow deprecated commands (like `whatchanged`) incorporated all technical feedback. Junio Hamano suggested using the new `DEPRECATED` flag rather than hardcoded command lists, which King implemented in the final revision. The series improves user messaging and provides programmatic deprecation detection via `--list-cmds=deprecated`.

## In brief

**HTTP clone issue resolved** -- A thread about HTTP/2 clone failures was closed as resolved server-side by GitHub, though discussion highlighted the value of resumable transfers for unreliable networks.

**git-new-workdir removal impacts** -- Users reported workflow disruption from the script's removal, as `git worktree` doesn't fully replace its ability to check out the same branch multiple times. Junio Hamano suggested detached HEAD as an alternative for inspection workflows.

**xdiff performance optimizations** -- A 4-part series from Alexander Monakov optimized xdiff's hash calculation, showing 8% speedups in `git log --oneline --shortstat` through instruction-level parallelism and branch prediction hints.

**Color handling fixes** -- Jeff King's v2 series fixed color regression in interactive patch modes (`add -p`, `stash -p`), properly separating prompt and diff colors while adding comprehensive test coverage.

**Fast-import docs fixed** -- Kristoffer Haugsbakk corrected asciidoc formatting in the `git-fast-import` manpage, removing unintended literal block formatting from a security warning paragraph.

## On the radar

**SHA-256 interoperability** -- brian m. carlson's status update revealed substantial but incomplete progress on SHA-1/SHA-256 interoperability, which may impact Git 3.0 timelines. Key challenges remain in shallow operations and submodule handling.

**Ref optimization architecture** -- Junio Hamano challenged the design of a proposed `git refs optimize` command, suggesting instead extending `git pack-refs` with backend-specific optimization hooks. The thread awaits the original contributor's response.

**WebDAV protocol proposal** -- Milan Hauth's suggestion to support WebDAV PROPFIND in Git's HTTP transport met resistance from Junio Hamano, who views it as contrary to dumb HTTP's intentionally minimal design.