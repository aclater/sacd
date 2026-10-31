# SACD projects

Open tools for reading Super Audio CD disc images: a specification written from scratch, a
clean-room Rust library built only from it, and the tools that use it. This repository is the
map: what each project is for, how they fit together, where things stand and why the big
decisions were made. It holds no code.

**Board:** [SACD projects](https://github.com/users/aclater/projects/1) (open issues and PRs of all
five repositories in one view).

## The projects

| Project | What it is | Licence | Side of the wall |
|---|---|---|---|
| [sacd-spec](https://github.com/aclater/sacd-spec) | Implementation-free specification of SACD disc images, plus a synthetic conformance corpus and the benchmark method and results | CC BY 4.0 (corpus CC0) | the wall itself |
| [scarletbook-rs](https://github.com/aclater/scarletbook-rs) | Clean-room Rust library: TOC, audio, DST decoding, C ABI; written only from sacd-spec | MIT OR Apache-2.0 | clean |
| [pysacd](https://github.com/aclater/pysacd) | Python toolkit and `sacd_extract`-compatible extractor (DSF, DSDIFF, FLAC, WAV), C engine; on [PyPI](https://pypi.org/project/pysacd/) | GPL-2.0-or-later | not clean |
| [beets-pysacd](https://github.com/aclater/beets-pysacd) | beets plugin: imports SACD images into a music library, MusicBrainz matching, DVD/Blu-ray audio cataloguing; on [PyPI](https://pypi.org/project/beets-pysacd/) | GPL-2.0-or-later | not clean |
| [sacdfs](https://github.com/aclater/sacdfs) | Read-only FUSE filesystem: mount a folder of images and browse every track as DSF (WAV in progress), decoded on demand by scarletbook | GPL-3.0-or-later | not clean |

```
                    sacd-spec  (specification + corpus)
                        │  read only by the clean side
                        ▼
                  scarletbook-rs  (clean Rust library)
                   │            │
         Rust API  │            │  C ABI (scarletbook-ffi)
                   ▼            ▼
                sacdfs        pysacd  (will switch its engine to scarletbook)
                                │
                                ▼
                          beets-pysacd
```

## The clean-room wall

scarletbook-rs is permissively licensed, so it must not be derived from GPL code. It is
written in an isolated environment by agents that have read **only** sacd-spec, and the
specification side is written without looking at implementations' code. Every spec PR records
its sources in [PROVENANCE.md](https://github.com/aclater/sacd-spec/blob/main/PROVENANCE.md), and
scarletbook-rs records its research in
[CLEANROOM.md](https://github.com/aclater/scarletbook-rs/blob/main/CLEANROOM.md).

Rules that keep it intact:

- Information may flow from the clean side to everyone else (its code is MIT/Apache).
- Nothing flows the other way. Requirements from pysacd or sacdfs reach scarletbook-rs as
  issues stating needs and measurements, never code, internals or links to non-clean code.
- Benchmarks of scarletbook are black-box: its released binaries are measured; numbers only are
  published in sacd-spec.
- This repository is a description, not an input to the clean side.

## Where things stand (2026-10-31)

- **scarletbook 0.3.0:** passes the whole corpus and real-disc hash checks. DST decoding 0.171 ms
  per frame on one core (4.0x `sacd_extract`); 2.8x to 3.5x `sacd_extract` with all cores.
  Working towards 1.0: interface stability, seeking, concurrent reads, channel assignment,
  patent hygiene review.
- **pysacd 0.2.2** and **beets-pysacd 0.1.2:** released; pysacd's own DST decoder is 2.4x to 3.0x
  `sacd_extract`.
- **sacdfs:** reads are bit-identical to extraction; reading a whole disc through the mount is
  faster than `sacd_extract` extracting it. WAV output under way; FLAC deferred.

Full figures: [sacd-spec benchmarks](https://github.com/aclater/sacd-spec/blob/main/benchmarks/README.md).

## Roadmap

1. **sacdfs WAV** (in progress): converter, then files, then a real-world pass
   ([sacdfs#11](https://github.com/aclater/sacdfs/issues/11)).
2. **Final scarletbook test**, once every scarletbook-rs issue and PR is resolved: correctness on
   all builds, an idle-machine round, a head-to-head of every tool, a whole-library survey.
3. **scarletbook 1.0** and publication on crates.io.
4. **pysacd on scarletbook:** pysacd switches its engine to scarletbook through the C ABI.
5. **Releases** of sacdfs, and of pysacd on the new engine.

## Decisions

The major decisions and why they were made: [DECISIONS.md](DECISIONS.md).
