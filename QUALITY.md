# Quality

How each SACD project guards against bugs, regressions, memory leaks, supply-chain problems and
bad releases. Every item below links to the workflow, script, test or document that implements
it, so each claim can be checked. Practices that a person carries out (at release time, or when
measuring) are marked **manual** and link to where they are written down. Where something is
not covered yet, it is listed under [Known gaps](#known-gaps).

scarletbook-rs is a separate clean-room project. Its column only summarises what its public
repository says; nothing here is input to it.

**Legend:** ✓ links to the source; – means the project has no such check (often because it does
not apply, such as a sanitizer for a pure-Python project). The overview repository itself
(aclater/sacd) holds documents only and has no CI.

Short names for the repositories, used in the tables: **sacdfs**
([aclater/sacdfs](https://github.com/aclater/sacdfs)), **pysacd**
([aclater/pysacd](https://github.com/aclater/pysacd)), **beets**
([aclater/beets-pysacd](https://github.com/aclater/beets-pysacd)), **scarletbook**
([aclater/scarletbook-rs](https://github.com/aclater/scarletbook-rs)), **spec**
([aclater/sacd-spec](https://github.com/aclater/sacd-spec)).

## Contents

- [Per-PR CI: build, test, lint](#per-pr-ci-build-test-lint)
- [Correct output: golden and byte-identical tests](#correct-output-golden-and-byte-identical-tests)
- [Hostile and damaged images](#hostile-and-damaged-images)
- [Fuzzing](#fuzzing)
- [Sanitizers and memory checks](#sanitizers-and-memory-checks)
- [Supply chain and code scanning](#supply-chain-and-code-scanning)
- [Branch and tag rules](#branch-and-tag-rules)
- [Benchmarks](#benchmarks)
- [Documentation checks](#documentation-checks)
- [Issue triage](#issue-triage)
- [The clean-room separation](#the-clean-room-separation)
- [Release process](#release-process)
- [Known gaps](#known-gaps)

## Per-PR CI: build, test, lint

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| Test suite | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/ci.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) | – |
| Several CPU architectures | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/ci.yml) x86-64, arm64 | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/workflow.yml) wheels for x86-64 and arm64 | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/clib.yml) C library per target | – |
| Several operating systems | – (Linux only) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/portability.yml) macOS, Windows | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) Linux, macOS, Windows | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) Linux, macOS, Windows | – |
| Several language versions | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/msrv.yml) MSRV 1.85 | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) Python 3.11–3.14 | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) Python 3.11–3.14, oldest and latest beets | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) MSRV 1.85 | – |
| Format and lint, warnings as errors | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/ci.yml) rustfmt, clippy | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) ruff, mypy, C with `-Werror` | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) ruff, mypy | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) rustfmt, clippy | – |
| Coverage | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/coverage.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/coverage.yml) | – |
| Real mounts and shares | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/container.yml) container over SMB and NFS; [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/nfs.yml) NFS export | – | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) tags served by sacdfs `main` | – | – |
| Against the dependent project's latest code | – | – | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/ci.yml) pysacd release and `master` | – | – |
| C interface checks | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) CET shadow-stack mark kept | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) header and ABI baseline | – |
| AI review of each PR | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/pr-agent.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/pr-agent.yml) | – | – |

- **Tests** run on every pull request and every push to the default branch. sacdfs's FUSE mount
  tests are required in CI (`SACDFS_FUSE_TESTS=1`), so a missing FUSE fails the run instead of
  skipping the tests ([tests/mount.rs](https://github.com/aclater/sacdfs/blob/main/tests/mount.rs)).
- **Architectures and systems** catch code that only works where it was written: sacdfs runs
  its tests and its container share test on x86-64 and arm64 runners. pysacd tests macOS and
  Windows when the engine changes, and weekly
  ([portability.yml](https://github.com/aclater/pysacd/blob/master/.github/workflows/portability.yml)).
- **MSRV** (minimum supported Rust version) checks that the code still builds on the oldest
  compiler the project promises.
- **Lint as errors** stops warnings piling up. In sacdfs, `unwrap`, `expect` and `panic` are
  clippy warnings, so CI rejects them
  ([CONTRIBUTING.md](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#ground-rules)).
- **Real mounts and shares:** sacdfs builds its container, serves the test corpus and reads it
  back over SMB and NFS ([deploy/ci-share-test.sh](https://github.com/aclater/sacdfs/blob/main/deploy/ci-share-test.sh)),
  and checks that NFS file handles survive a restart
  ([tests/export.rs](https://github.com/aclater/sacdfs/blob/main/tests/export.rs),
  [tests/nfs-loopback.sh](https://github.com/aclater/sacdfs/blob/main/tests/nfs-loopback.sh)).
  Other sacdfs tests cover a source that stops answering
  ([tests/stall.rs](https://github.com/aclater/sacdfs/blob/main/tests/stall.rs)) and many
  readers of one image at once
  ([tests/concurrent.rs](https://github.com/aclater/sacdfs/blob/main/tests/concurrent.rs)).
- **Cross-project:** beets-pysacd tests against pysacd's development branch and against a build
  of sacdfs `main`, so a change in either shows up before it is released.
- **C interface:** scarletbook checks that the committed C header is what the generator
  produces and that the ABI matches its recorded baseline
  ([README, "ABI compatibility"](https://github.com/aclater/scarletbook-rs/blob/main/README.md#abi-compatibility)).
  pysacd checks that its engine keeps the CET shadow-stack hardening mark
  ([tools/check_cet.py](https://github.com/aclater/pysacd/blob/master/tools/check_cet.py)).
- **AI review:** a second model reviews each pull request, and every finding gets an answer
  (fixed, or why not) ([pysacd CONTRIBUTING.md](https://github.com/aclater/pysacd/blob/master/CONTRIBUTING.md),
  [beets-pysacd CONTRIBUTING.md](https://github.com/aclater/beets-pysacd/blob/main/CONTRIBUTING.md)).

## Correct output: golden and byte-identical tests

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| Conformance corpus with expected hashes | [✓](https://github.com/aclater/sacdfs/blob/main/tests/corpus.rs) | [✓](https://github.com/aclater/pysacd/blob/master/docs/development.md#the-toc-through-scarletbook) (manual, when the reader changes) | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/README.md#layout) | [✓](https://github.com/aclater/sacd-spec/blob/main/README.md#using-the-corpus) provides it |
| Pinned golden output | [✓](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#ground-rules) DSF and PCM | – | – | – | – |
| Byte-identical to a reference tool, in CI | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) `sacd_extract`, generated discs | – | – | – |
| Byte-identical on real discs | [✓](https://github.com/aclater/sacdfs/blob/main/docs/design.md) (manual) | [✓](https://github.com/aclater/pysacd/blob/master/docs/performance.md) (manual) | – | – | – |
| Correctness gate before a timing | [✓](https://github.com/aclater/sacdfs/blob/main/docs/performance.md#how-these-figures-were-taken) | [✓](https://github.com/aclater/pysacd/blob/master/docs/performance.md) | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/release.yml) | [✓](https://github.com/aclater/sacd-spec/blob/main/benchmarks/README.md) defines it |

- **The conformance corpus** of sacd-spec is a set of synthetic disc images (33 valid, 103
  malformed in the copy sacdfs carries) with the expected TOC, warnings and per-channel audio
  hashes of each ([corpus README](https://github.com/aclater/sacdfs/blob/main/tests/corpus/README.md)).
  sacdfs reads every case through its tree and readers, and through a real mount
  ([tests/corpus.rs](https://github.com/aclater/sacdfs/blob/main/tests/corpus.rs),
  [tests/mount.rs](https://github.com/aclater/sacdfs/blob/main/tests/mount.rs)).
  scarletbook runs every corpus case in `cargo test`.
- **Golden output:** sacdfs pins the hashes of the DSF files it serves (`golden_dsf_corpus`) and
  of its PCM output at 1, 2 and 6 channels and three gains. A change that moves those bytes must
  also bump the output version and be recorded in the changelog, so players and taggers see the
  change ([CONTRIBUTING.md](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#ground-rules)).
  The PCM converter is also measured for frequency response, level, distortion and determinism
  ([tests/pcm.rs](https://github.com/aclater/sacdfs/blob/main/tests/pcm.rs),
  [docs/pcm-quality.md](https://github.com/aclater/sacdfs/blob/main/docs/pcm-quality.md)).
- **Against `sacd_extract`:** pysacd's `pysacd-extract` must write the same files, names and
  bytes as `sacd_extract`. CI builds `sacd_extract` from a pinned upstream commit inside the
  runner and compares on generated discs only (job "compare with sacd_extract"): whole output
  trees in every compatible mode, names, ID3, DSDIFF, cue, XML, `-P` and the PS3 client. Real
  discs are compared by hand before each release. This comparison did not run in CI for releases
  0.3.0 to 0.4.3 ([pysacd#131](https://github.com/aclater/pysacd/issues/131)); before that it
  covered decoded audio only. Checked by hand afterwards: 0.4.3 and master on 5 real discs, 175
  runs, 0 differences between them and none from `sacd_extract` beyond documented exceptions,
  including `-b`, where `sacd_extract` repeats frames at some track boundaries; its code never enters the repository
  ([docs/development.md, "CI"](https://github.com/aclater/pysacd/blob/master/docs/development.md#ci)).
- **Real discs (manual):** a scarletbook update in sacdfs is gated like a release: every track
  of the real test discs must be bit-identical to the reference extractor
  ([docs/design.md](https://github.com/aclater/sacdfs/blob/main/docs/design.md)). pysacd records
  byte-identical output on its real test discs in every mode for each engine change
  ([docs/performance.md](https://github.com/aclater/pysacd/blob/master/docs/performance.md),
  [CHANGELOG.md](https://github.com/aclater/pysacd/blob/master/CHANGELOG.md)).
- **`sacdfs verify`** lets users check their own extracts against the mount before deleting
  them; its test feeds it rewritten, bit-flipped, truncated and recompressed files
  ([tests/verify.rs](https://github.com/aclater/sacdfs/blob/main/tests/verify.rs)).

## Hostile and damaged images

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| Malformed-image corpus in the tests | [✓](https://github.com/aclater/sacdfs/blob/main/tests/corpus/README.md) 103 cases | [✓](https://github.com/aclater/pysacd/blob/master/docs/security.md#hostile-images) about 95 images | [✓](https://github.com/aclater/beets-pysacd/blob/main/tests/test_discimage.py) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/SECURITY.md) | [✓](https://github.com/aclater/sacd-spec/blob/main/spec/07-validation-and-limits.md) limits and required behaviour |
| Hostile disc text and file names | [✓](https://github.com/aclater/sacdfs/blob/main/docs/design.md) untrusted paths | [✓](https://github.com/aclater/pysacd/blob/master/docs/security.md#hostile-text) | [✓](https://github.com/aclater/beets-pysacd/blob/main/CONTRIBUTING.md) | – | – |
| Written security policy | [✓](https://github.com/aclater/sacdfs/blob/main/SECURITY.md) | [✓](https://github.com/aclater/pysacd/blob/master/SECURITY.md) | [✓](https://github.com/aclater/beets-pysacd/blob/main/SECURITY.md) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/SECURITY.md) | [✓](https://github.com/aclater/sacd-spec/blob/main/SECURITY.md) |

Every project treats a disc image as untrusted input: nothing in it may crash, hang, read or
write out of bounds, or grow memory without limit. sacd-spec's specification 7 lists every limit
and the required behaviour for each, so all readers reject the same images the same way.

- **pysacd** runs each of its hostile images through every extraction mode and every output
  format. Each run must end in a clean error or sane output: no other exception, no hang,
  nothing outside the output directory, no partial file, and no file larger than the image could
  hold. CI runs this under ASan and UBSan
  ([docs/security.md](https://github.com/aclater/pysacd/blob/master/docs/security.md#hostile-images)).
  Disc text, MusicBrainz data and network input pass through one sanitiser, with its own tests
  ([tests/test_safetext.py](https://github.com/aclater/pysacd/blob/master/tests/test_safetext.py)).
- **sacdfs** reads all 103 malformed corpus cases through the mount. Sidecar files are opened
  one path component at a time from a descriptor of the source, so links cannot escape it
  ([docs/design.md](https://github.com/aclater/sacdfs/blob/main/docs/design.md)).
- **scarletbook** bounds every allocation, checks every position, returns typed errors and lets
  no panic cross the C ABI ([SECURITY.md](https://github.com/aclater/scarletbook-rs/blob/main/SECURITY.md)).

## Fuzzing

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| Coverage-guided fuzzing | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/fuzz.yml) libFuzzer | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/fuzz.yml) Atheris | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) cargo-fuzz | – |
| Long scheduled run | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/fuzz.yml) 30 min weekly | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/fuzz.yml) weekly | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/fuzz-long.yml) weekly | – |
| Crashing inputs kept | – | [✓](https://github.com/aclater/pysacd/blob/master/tests/test_fuzz_engine.py) replayed by the tests | [✓](https://github.com/aclater/beets-pysacd/blob/main/CLAUDE.md) in `fuzz/regressions/` | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/fuzz-long.yml) as run artifacts | – |
| Seeded mutation tests in the suite | – | [✓](https://github.com/aclater/pysacd/blob/master/tests/test_fuzz.py) | – | – | – |

- **pysacd:** the libFuzzer harness feeds disc images to the C engine, built with ASan and
  UBSan, for 2 minutes on pull requests that touch the engine, the fuzz code or the scarletbook
  pin, and 30 minutes weekly with a cached corpus. A crashing input is kept, minimised, in
  `fuzz/regressions/` with its fix, and the test suite replays it under the sanitizers
  ([docs/development.md, "Tests"](https://github.com/aclater/pysacd/blob/master/docs/development.md#tests)).
  `tests/test_fuzz.py` also runs seeded mutation fuzzing on the Python parsers.
- **beets-pysacd** fuzzes its disc-image identifier with Atheris on relevant changes and weekly
  ([fuzz/fuzz_discimage.py](https://github.com/aclater/beets-pysacd/blob/main/fuzz/fuzz_discimage.py)).
- **scarletbook** fuzzes four targets (TOC, demultiplexing, DST, seek index), seeded from the
  conformance corpus: a short run on every pull request that changes code, and a long run weekly.
- **sacdfs** has no fuzz target of its own; image parsing is scarletbook's, fuzzed there, and
  sacdfs runs the malformed corpus (see [Known gaps](#known-gaps)).

## Sanitizers and memory checks

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| AddressSanitizer + UBSan | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) | – | – | – |
| ThreadSanitizer | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/ci.yml) | – | – | – |
| Miri (undefined behaviour in Rust) | – | – | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/miri.yml) | – |
| LeakSanitizer | – | – | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) C interface | – |
| No-growth test in CI | [✓](https://github.com/aclater/sacdfs/blob/main/tests/no_growth.rs) | – | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/CONTRIBUTING.md) | – |
| Memory soak | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/leaks.yml) 20 min weekly; 2 h on real discs before releases | – | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/soak.yml) 1 h weekly | – |
| Valgrind memcheck | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/leaks.yml) test suite, weekly | – | – | – | – |

- **pysacd** runs its whole test suite, hostile images included, with the C engine built under
  ASan and UBSan, and again under TSan to catch data races in the multithreaded engine. Both are
  required checks. The ASan job doesn't detect leaks; a separate engine job runs LeakSanitizer and valgrind memcheck on every pull request ([pysacd#122](https://github.com/aclater/pysacd/issues/122)).
- **scarletbook** runs its tests under Miri (a slice on changes, in full nightly), the C
  interface tests and the C smoke test under LeakSanitizer on every code change, a test that
  memory does not grow over many open, seek, decode and close cycles, and a weekly one-hour
  soak that fails unless resident memory and the heap level off
  ([CONTRIBUTING.md](https://github.com/aclater/scarletbook-rs/blob/main/CONTRIBUTING.md)).
  `unsafe` is confined to the C interface crate and one run-time dispatch module per library
  crate, and a test checks that each unsafe call there has its own `SAFETY:` comment.
- **sacdfs** checks for leaks at three levels ([docs/development.md](https://github.com/aclater/sacdfs/blob/main/docs/development.md)):
  - **Every pull request:** a no-growth test ([tests/no_growth.rs](https://github.com/aclater/sacdfs/blob/main/tests/no_growth.rs)) fails if
    the heap grows over repeated open, read and close cycles, along with regression tests for each
    leak found so far: removed or renamed images and their seek-index entries are freed
    ([tests/removed_images.rs](https://github.com/aclater/sacdfs/blob/main/tests/removed_images.rs),
    [tests/removed_images_index.rs](https://github.com/aclater/sacdfs/blob/main/tests/removed_images_index.rs)), and a worker pool's spare
    threads end.
  - **Every week:** the test suite under valgrind memcheck and a 20-minute soak on the test
    corpus ([leaks.yml](https://github.com/aclater/sacdfs/blob/main/.github/workflows/leaks.yml)).
  - **Before a release:** a 2-hour soak on real disc images ([scripts/soak.py](https://github.com/aclater/sacdfs/blob/main/scripts/soak.py)).
    Albums are added and removed under new names while files are read, stalls are forced and the
    filesystem is remounted. The soak fails unless resident memory levels off. For 0.4.0 it found
    three leaks that the small test images could not show
    ([#153](https://github.com/aclater/sacdfs/issues/153),
    [#167](https://github.com/aclater/sacdfs/issues/167),
    [#169](https://github.com/aclater/sacdfs/pull/169)), and passed after their fixes.
  - **Published memory:** peak memory is published with each performance round
    ([docs/performance.md](https://github.com/aclater/sacdfs/blob/main/docs/performance.md)).

## Supply chain and code scanning

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| CodeQL | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/codeql.yml) Rust, Actions | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/codeql.yml) Python, C, Actions | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/codeql.yml) Python, Actions | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/codeql.yml) Rust, Actions | – |
| semgrep | – | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/semgrep.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/semgrep.yml) | – | – |
| zizmor (workflow security) | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/zizmor.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/zizmor.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/zizmor.yml) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/zizmor.yml) | – |
| cargo-deny (advisories, licences, sources) | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/deny.yml) | – | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) | – |
| Dependency review | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/dependency-review.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/dependency-review.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/dependency-review.yml) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/dependency-review.yml) | – |
| OpenSSF Scorecard | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/scorecard.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/scorecard.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/scorecard.yml) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/scorecard.yml) | [✓](https://github.com/aclater/sacd-spec/blob/main/.github/workflows/scorecard.yml) |
| Locked, hash-checked dependencies | [✓](https://github.com/aclater/sacdfs/blob/main/Cargo.lock) `--locked` | [✓](https://github.com/aclater/pysacd/blob/master/docs/development.md#ci) `--require-hashes` | [✓](https://github.com/aclater/beets-pysacd/tree/main/locks) `--require-hashes` | [✓](https://github.com/aclater/scarletbook-rs/blob/main/Cargo.lock) `--locked` | – |
| Provenance of the scarletbook build used | [✓](https://github.com/aclater/sacdfs/blob/main/docs/design.md) exact crates.io pin | [✓](https://github.com/aclater/pysacd/blob/master/docs/development.md#the-scarletbook-library) attestation checked | – | – | – |
| Dependabot | [✓](https://github.com/aclater/sacdfs/blob/main/.github/dependabot.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/dependabot.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/dependabot.yml) | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/dependabot.yml) | [✓](https://github.com/aclater/sacd-spec/blob/main/.github/dependabot.yml) |
| No private names in files or commits | [✓](https://github.com/aclater/sacdfs/blob/main/scripts/check-private-names.sh) | [✓](https://github.com/aclater/pysacd/blob/master/scripts/check-private-names.sh) | [✓](https://github.com/aclater/beets-pysacd/blob/main/scripts/check-private-names.sh) | – | – |

- **CodeQL** fails the run on findings, and runs weekly as well, to pick up new queries.
  **semgrep** adds a second static analyser for the Python and C code.
- **zizmor** audits the workflows themselves (credentials left on checkouts, untrusted text in
  scripts). Every action is pinned by full commit SHA
  ([sacdfs CONTRIBUTING.md](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#ground-rules)).
- **cargo-deny** checks Rust dependencies for known advisories, disallowed licences and
  unexpected sources, on every pull request and weekly, since advisories appear without a code
  change ([deny.toml](https://github.com/aclater/sacdfs/blob/main/deny.toml)).
  **Dependency review** fails a pull request that adds a dependency with a known high or
  critical vulnerability.
- **Hash-locked installs:** pysacd and beets-pysacd install every Python package in CI pinned
  and hash-checked; pysacd's lock tool refuses package releases younger than two weeks, and
  Dependabot bumps with a two-week cooldown
  ([docs/development.md, "CI"](https://github.com/aclater/pysacd/blob/master/docs/development.md#ci)).
- **scarletbook provenance:** pysacd links scarletbook's released C library. A required check
  verifies that every pinned archive matches its digest and carries a build provenance
  attestation from scarletbook-rs's release workflow, for the pinned tag and commit
  ([tools/scarletbook.py](https://github.com/aclater/pysacd/blob/master/tools/scarletbook.py)).
  sacdfs pins scarletbook exactly from crates.io, where cargo's checksum fixes the bytes.
- **Private names:** a CI check fails on private details (host names, private addresses) in
  tracked files and commit identities.

## Branch and tag rules

Each repository has two [rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets),
with no bypass actors:

| Rule | sacdfs | pysacd | beets | scarletbook | spec | sacd |
|---|---|---|---|---|---|---|
| Default branch: changes only by pull request, no force-push, no deletion | [✓](https://github.com/aclater/sacdfs/rules) | [✓](https://github.com/aclater/pysacd/rules) | [✓](https://github.com/aclater/beets-pysacd/rules) | [✓](https://github.com/aclater/scarletbook-rs/rules) | [✓](https://github.com/aclater/sacd-spec/rules) | [✓](https://github.com/aclater/sacd/rules) |
| Required status checks | [✓](https://github.com/aclater/sacdfs/rules) 11 | [✓](https://github.com/aclater/pysacd/rules) 15 | [✓](https://github.com/aclater/beets-pysacd/rules) 13 | [✓](https://github.com/aclater/scarletbook-rs/rules) 7 | – | – |
| Branch must be up to date with the default branch before merging | [✓](https://github.com/aclater/sacdfs/rules) | [✓](https://github.com/aclater/pysacd/rules) | [✓](https://github.com/aclater/beets-pysacd/rules) | [✓](https://github.com/aclater/scarletbook-rs/rules) | – | – |
| `v*` tags immutable (no update, deletion or force-push) | [✓](https://github.com/aclater/sacdfs/rules) | [✓](https://github.com/aclater/pysacd/rules) | [✓](https://github.com/aclater/beets-pysacd/rules) | [✓](https://github.com/aclater/scarletbook-rs/rules) | [✓](https://github.com/aclater/sacd-spec/rules) | [✓](https://github.com/aclater/sacd/rules) |

- **The required checks** are, in sacdfs: fmt + clippy, tests, MSRV, cargo-deny, dependency
  review, CodeQL (Rust, Actions), coverage, the container share test on x86-64 and arm64, and the
  NFS export test. In pysacd: lint and tests on Python 3.11–3.15, ASan + UBSan, TSan, the engine leak check, the
  `sacd_extract` comparison, the private-names check, CodeQL (Python, C, Actions), semgrep and
  scarletbook provenance. In beets-pysacd: lint, the private-details check, CodeQL, semgrep and
  eight test legs across systems, Python, beets and pysacd versions. In scarletbook: lint, tests,
  cargo-deny, MSRV, CodeQL and dependency review.
- **No required check is path-filtered** in sacdfs and pysacd, because a skipped required check
  blocks the merge; slow, rarely-failing jobs run on the default branch, tags or a schedule
  instead ([sacdfs CONTRIBUTING.md](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#ground-rules),
  [pysacd docs/development.md, "CI speed"](https://github.com/aclater/pysacd/blob/master/docs/development.md#ci-speed)).
- **Up-to-date branches** are required wherever status checks are, so a pull request's green checks
  always describe the code it merges into. A pull request that was green against an older default
  branch once broke sacdfs. sacd-spec and this repository have no check that runs on every pull
  request, so the rule has nothing to attach to there
  ([#37](https://github.com/aclater/sacd/issues/37)).
- **Immutable tags** mean a release that went wrong is followed by a new version, never moved
  ([sacdfs CONTRIBUTING.md, "Releasing"](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#releasing)).
- **sacd-spec** has no CI tests to require; its pull requests are merged by the owner after a
  pre-merge check, and every change to a specification records its sources in PROVENANCE.md
  ([CONTRIBUTING.md](https://github.com/aclater/sacd-spec/blob/main/CONTRIBUTING.md)).

## Benchmarks

| Practice | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| Timings under an exclusive benchmark lock (manual) | [✓](https://github.com/aclater/sacdfs/blob/main/docs/performance.md#how-these-figures-were-taken) | [✓](https://github.com/aclater/pysacd/blob/master/docs/performance.md) | – | – | – |
| Alternating runs, means of 3 or more | [✓](https://github.com/aclater/sacdfs/blob/main/docs/performance.md#how-these-figures-were-taken) | [✓](https://github.com/aclater/pysacd/blob/master/docs/performance.md) | – | – | [✓](https://github.com/aclater/sacd-spec/blob/main/benchmarks/README.md) method |
| Compared with earlier versions, regressions called out | [✓](https://github.com/aclater/sacdfs/blob/main/docs/performance.md) history | [✓](https://github.com/aclater/pysacd/blob/master/CHANGELOG.md) per release | – | – | – |
| Benchmark regression check in CI | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/ci.yml) smoke run only | – | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/bench-regress.yml) informational | – |

- **The method (manual):** timings are taken under a benchmark lock so that nothing else
  benchmarks at the same time, on a quiet machine (sacdfs waits for a low load average and
  watches for compilers and fuzzers), with old and new builds run alternately and the mean
  reported. A correctness gate comes first: the output must be bit-identical before a timing
  counts ([sacdfs docs/performance.md](https://github.com/aclater/sacdfs/blob/main/docs/performance.md#how-these-figures-were-taken),
  [sacd-spec benchmarks](https://github.com/aclater/sacd-spec/blob/main/benchmarks/README.md)).
- **Across versions:** sacdfs keeps a history table with each change's before and after
  figures, and the figures of earlier releases beside them. pysacd's changelog gives each
  release's timings next to the previous release's. Regressions are named, such as the plain-DSD
  WAV slowdown that sacdfs 0.2.0 shipped as a known issue and a later release removed
  ([docs/performance.md](https://github.com/aclater/sacdfs/blob/main/docs/performance.md),
  [RELEASES.md](RELEASES.md)).
- **In CI:** scarletbook builds the base and the head of a pull request and times them
  alternately on the same runner; nightly runs compare with the latest release, so small
  slowdowns that add up still show. It is informational, not a required check. sacdfs runs its
  converter benchmark once per case as a smoke test, so it can't rot.

## Documentation checks

| Check | sacdfs | pysacd | beets | scarletbook | spec |
|---|---|---|---|---|---|
| Link check | [✓](https://github.com/aclater/sacdfs/blob/main/.github/workflows/docs-links.yml) | [✓](https://github.com/aclater/pysacd/blob/master/.github/workflows/docs-links.yml) | [✓](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/docs-links.yml) | – | [✓](https://github.com/aclater/sacd-spec/blob/main/.github/workflows/docs-links.yml) |
| Docs match the program | [✓](https://github.com/aclater/sacdfs/blob/main/.github/scripts/check-readme-options.py) options table = `--help` | [✓](https://github.com/aclater/pysacd/blob/master/tests/test_docs_index.py) docs index | – | [✓](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/ci.yml) header, changelog | – |
| Docs say which release they were tested on (manual) | [✓](https://github.com/aclater/sacdfs/blob/main/docs/clients.md) | – | – | – | – |

- **Links** are checked on pull requests and weekly, since links rot without a code change.
- **Docs that match the code:** sacdfs's README options table must equal `sacdfs --help`.
  The contributing guides of sacdfs, pysacd and beets-pysacd ask for the docs and the changelog in the same pull request.
- **Tested, not assumed (manual):** sacdfs's client and install guides state the version each
  result was tested on ("Tested on 0.3.0-rc.1"), and mark the rest "untested" or "not
  validated" ([docs/clients.md](https://github.com/aclater/sacdfs/blob/main/docs/clients.md));
  the TrueNAS steps were tested with the published 0.1.0 image
  ([docs/deployment.md](https://github.com/aclater/sacdfs/blob/main/docs/deployment.md)).

## Issue triage

Every open issue in sacdfs, pysacd, beets-pysacd, scarletbook-rs and this repository has a
milestone (the release it is planned for, or `Later`), a priority (`P1` must be in its
milestone's release, `P2` can slip one release, `P3` nice to have), a status and a model label.
An issue fixed on the default branch stays open, marked `status: fixed on main`, until the
release ships and has been verified. A pull request says `Closes #N` only if it fully resolves
the issue ([CONTRIBUTING.md](CONTRIBUTING.md);
[sacdfs](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#issues-milestones-labels-and-claiming),
[pysacd](https://github.com/aclater/pysacd/blob/master/CONTRIBUTING.md#issues-milestones-labels-and-claiming),
[beets-pysacd](https://github.com/aclater/beets-pysacd/blob/main/CONTRIBUTING.md),
[scarletbook-rs](https://github.com/aclater/scarletbook-rs/blob/main/CONTRIBUTING.md),
[sacd-spec](https://github.com/aclater/sacd-spec/blob/main/CONTRIBUTING.md)). sacd-spec has no
releases, so its issues close when the answer merges.

## The clean-room separation

As the public repositories describe it ([README, "The clean-room wall"](README.md#the-clean-room-wall)):

- **sacd-spec** is an implementation-free specification with no source code; PROVENANCE.md
  records what was consulted for each section, and the specifications were cold-read by a
  reviewer with access to that repository only
  ([README](https://github.com/aclater/sacd-spec/blob/main/README.md#the-clean-room-process)).
- **scarletbook-rs** is written only from sacd-spec, and records every source consulted in
  [CLEANROOM.md](https://github.com/aclater/scarletbook-rs/blob/main/CLEANROOM.md); every pull
  request lists its sources ([CONTRIBUTING.md](https://github.com/aclater/scarletbook-rs/blob/main/CONTRIBUTING.md)).
- **pysacd and sacdfs** never send code to scarletbook-rs: their needs go to it as issues. They
  use scarletbook only through its public API and released builds
  ([pysacd docs/clean-room.md](https://github.com/aclater/pysacd/blob/master/docs/clean-room.md),
  [sacdfs CONTRIBUTING.md](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#ground-rules)).

## Release process

Common to the projects that publish packages; each step links to where it is implemented.

1. **The default branch is green** and the docs describe the release. A release pull request
   bumps the version and dates the changelog section, which becomes the release notes. It merges
   only when its checks pass ([sacdfs](https://github.com/aclater/sacdfs/blob/main/CONTRIBUTING.md#releasing),
   [pysacd](https://github.com/aclater/pysacd/blob/master/docs/development.md#packaging-and-releases),
   [scarletbook-rs](https://github.com/aclater/scarletbook-rs/blob/main/RELEASING.md)).
2. **A dry run** (sacdfs): the release workflow runs on `main`, and on pull requests that change
   release files, building and testing every asset and publishing nothing
   ([release.yml](https://github.com/aclater/sacdfs/blob/main/.github/workflows/release.yml)).
3. **Release candidates first** (sacdfs and pysacd): a pre-release version is tagged and tested
   before the final one. sacdfs candidates are tested on a NAS and with media servers (four for
   0.2.0, one for 0.3.0); pysacd candidates are installed on clean machines and
   containers (Fedora, UBI, Debian; for 0.4.3, from PyPI) and run their tests
   ([sacdfs CHANGELOG.md](https://github.com/aclater/sacdfs/blob/main/CHANGELOG.md),
   [pysacd CHANGELOG.md](https://github.com/aclater/pysacd/blob/master/CHANGELOG.md)). Both are
   manual steps recorded in the changelogs. sacdfs 0.1.0 was released without one; the reason is
   in [DECISIONS.md](DECISIONS.md).
4. **The tag must match** the version in the package and have a changelog section, or the
   workflow stops before building (sacdfs, scarletbook-rs).
5. **The assets are tested as packaged,** before anything is published:
   - sacdfs builds its binaries on UBI 8 (oldest supported glibc), runs the FUSE tests, then
     serves the corpus from the packaged binary; each container image passes the SMB and NFS
     share test, and that tested image is the one pushed
     ([release.yml](https://github.com/aclater/sacdfs/blob/main/.github/workflows/release.yml));
   - pysacd installs every wheel, and a build from the sdist, in a clean environment and
     converts a synthetic disc with it
     ([tests/wheel_smoke.py](https://github.com/aclater/pysacd/blob/master/tests/wheel_smoke.py));
   - beets-pysacd installs the wheel in a clean virtual environment and loads the plugin
     ([workflow.yml](https://github.com/aclater/beets-pysacd/blob/main/.github/workflows/workflow.yml));
   - scarletbook-rs runs the correctness gate (every corpus case, one and all threads) on the
     very builds it publishes, then smoke-tests the packaged binaries against the manifest
     ([release.yml](https://github.com/aclater/scarletbook-rs/blob/main/.github/workflows/release.yml)).
6. **Provenance and checksums:** every published file gets a build provenance attestation
   (SLSA), attached to the release as `<file>.intoto.jsonl`; sacdfs
   also attests its container image in the registry, and sacdfs and scarletbook-rs publish
   SHA-256 checksums. Users check a download with `gh attestation verify`
   ([sacdfs README](https://github.com/aclater/sacdfs/blob/main/README.md),
   [pysacd docs/development.md](https://github.com/aclater/pysacd/blob/master/docs/development.md#packaging-and-releases),
   [beets-pysacd README](https://github.com/aclater/beets-pysacd/blob/main/README.md),
   [scarletbook-rs README](https://github.com/aclater/scarletbook-rs/blob/main/README.md#verifying-a-download)).
7. **Publishing without stored tokens:** PyPI (pysacd, beets-pysacd) and crates.io
   (scarletbook-rs) use trusted publishing through GitHub OIDC. pysacd publishes to TestPyPI
   first and to PyPI only if that succeeded, and only a `v*` tag can reach the `pypi`
   environment.
8. **After the release:** the issues it fixed are verified and closed, and the release is
   summarised in [RELEASES.md](RELEASES.md).

## Known gaps

What isn't covered yet, from open issues and from the checks above:

- **sacdfs fuzzing:** sacdfs has no fuzz target of its own; it relies on scarletbook's fuzzing
  and on the 103 malformed corpus cases ([sacdfs#161](https://github.com/aclater/sacdfs/issues/161)).
- **Benchmark regressions** are caught in CI only as an informational check, in scarletbook, sacdfs and pysacd; timings are still compared by hand at release.
- **pysacd's PS3 client** is untested on real hardware
  ([pysacd#10](https://github.com/aclater/pysacd/issues/10)).
- **scarletbook-rs before 1.0** (its own public issues): interface stability for integration
  ([#39](https://github.com/aclater/scarletbook-rs/issues/39)) and a review of every
  unsafe-code decision after 1.0 ([#199](https://github.com/aclater/scarletbook-rs/issues/199)).
