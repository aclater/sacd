# Releases by major version

A running summary of what each minor version line of each project brought, so progress can be
followed by major step and not only by point release. Below 1.0 the minor version is the major
step: it may change behaviour, and a patch release only adds or fixes.

This page summarises public release notes only. The sources are each project's `CHANGELOG.md`,
its GitHub releases and its milestones; for any detail, they win. Newest first. Last updated
2026-11-02.

| Project | Latest release | In progress |
|---|---|---|
| [sacdfs](#sacdfs) | 0.3.0 (2026-11-02) | 0.4.0 |
| [pysacd](#pysacd) | 0.4.2 (2026-11-02) | 0.5.0 |
| [beets-pysacd](#beets-pysacd) | 0.2.0 (2026-11-02) | nothing recorded |
| [scarletbook-rs](#scarletbook-rs) | 0.5.6 (2026-11-02) | 0.6.0, then 1.0.0 |

To keep this page current, each project's release checklist includes "update aclater/sacd
RELEASES.md".

## sacdfs

### In progress: 0.4.0

Milestone 0.4.0: cover.jpg for MPD (#148). No unreleased entries yet.

### 0.3 line

Milestone: robustness (stalls, lost mounts), the tag layer, remaining scarletbook adoption,
performance follow-ups. From `CHANGELOG.md` [Unreleased]:

- **Tag layer** (#81): `--tags DIR` serves user tags from one TOML file per disc, named by its
  TOC identity, so edits survive renames. Only the files a tag file changes get new bytes, mtime
  and generation. `sacdfs tags import` writes tag files from the tags of extracts that
  `sacdfs verify` matches, so those extracts can be deleted without losing the tags.
- **`--pauses kept`** (#112): serves every track with its pause, as DSF and WAV, so
  `sacdfs verify` reports IDENTICAL, not IDENTICAL-PAUSE, against pause-kept extracts. Off by
  default; default output is unchanged byte for byte.
- **scarletbook and scarletbook-pcm 0.5.3 to 0.5.6** (#128): output bytes unchanged. Plain-DSD
  stereo WAV reads 19 % faster and plain-DSD DSF 22 % faster than 0.2.0; this supersedes 0.2.0's
  known issue (plain-DSD WAV slower than 0.1.1, scarletbook-rs#216).
- **Memory slabs** (#73): the frame cache lives in memory mapped outside the heap, at most
  `--cache-mb`; a full-library import replay at the default cache peaks at 300 MiB (was 304).
- **Lost-mount fix** (#54): a sacdfs stopped after its mount was unmounted from outside could
  unmount the next sacdfs on the same path; it now checks its own FUSE connection.
- **Stall handling** is in the milestone's scope (stall #72 and stalled-share #63 are closed
  issues in it); it is not yet described in the changelog.
- Docs: how to make Plex show a multichannel copy as its own album (#138).

Releases: [0.3.0](https://github.com/aclater/sacdfs/releases/tag/v0.3.0) (2026-11-02), after
[0.3.0-rc.1](https://github.com/aclater/sacdfs/releases/tag/v0.3.0-rc.1).

### 0.2 line

Memory, scanner pickup and scarletbook adoption; the first line with output-version bumps.

- Memory during a full-library import halved (325 MB at the default cache, 60 MB at
  `--cache-mb 16`, against 631 and 92 MB in 0.1).
- New `sacdfs verify`: checks extracted DSF/DFF files against what sacdfs serves before they are
  deleted, lists tags a deletion would lose, reports reclaimable space; never deletes anything.
- Scanners that skip unchanged folders (Roon) find new files, changed tags and new output
  versions without a remount; folder art shows in each area folder.
- Multichannel files get their own album name, ending in ` (Multichannel)`
  (`--no-multichannel-album-suffix` turns it off).
- WAV conversion and seeking are now scarletbook-rs's (`scarletbook-pcm`, scarletbook 0.5.3);
  DSF files are written by scarletbook.
- **On upgrade:** output versions moved (DSF to 3, WAV to 4), so file mtimes and NFS generations
  change and copies and scanners pick up the new bytes; audio is unchanged. The persistent seek
  index of 0.1 is ignored and rewritten.
- Known issue at release (plain-DSD WAV reads slower than 0.1.1) was fixed in the 0.3 line's
  scarletbook 0.5.6, above.

Releases: [0.2.0](https://github.com/aclater/sacdfs/releases/tag/v0.2.0) (2026-11-02; after
four release candidates).

### 0.1 line

The first release: DSF and WAV on demand from SACD images.

- Read-only FUSE filesystem showing images as DSF files with ID3v2.4 tags, plus 24-bit PCM WAV
  files, decoded on demand by scarletbook.
- Persistent seek index, bounded caches, concurrent reads of one image, `--flatten`.
- Stable inode numbers and NFS export; a container image with Samba and NFS-Ganesha for a NAS.
- 0.1.1: sidecar files passed through (folder art, `Artwork/`, `Scans/`, playlists), cheaper
  probes, correct `df` size; audio bytes identical to 0.1.0.

Releases: [0.1.0](https://github.com/aclater/sacdfs/releases/tag/v0.1.0) (2026-11-01),
[0.1.1](https://github.com/aclater/sacdfs/releases/tag/v0.1.1) (2026-11-01).

## pysacd

### In progress: 0.5.0

Milestone 0.5.0: performance, pysacd's own pipeline around scarletbook. No unreleased entries yet.

### 0.4 line

pysacd-extract on scarletbook's writers.

- DSF and DSDIFF files are laid out by scarletbook-rs 0.5.4's output API; pysacd's own writers
  are deleted. Output is byte-identical to 0.3.1 in every mode.
- 0.4.1: faster DST decoding via scarletbook 0.5.5; DST stereo to DSF 16 % faster on a 16-core
  machine, output byte-identical.
- 0.4.2: DSF audio laid out by scarletbook-rs 0.5.6's layout fill (pysacd's own block layout
  deleted); plain DSD to DSF 2.1 times faster on NVMe, DST about 1.5 % faster, output
  byte-identical ([#114](https://github.com/aclater/pysacd/issues/114)).
- Workaround for early Sony discs whose last area frame is missing: filled with silence, warns,
  exits 0 instead of 1.
- Behaviour change: a file scarletbook won't lay out from a hostile or damaged TOC is reported
  and not written, and the run exits 255 (0.3.1 wrote it from the TOC as it stood).

Releases: [0.4.0](https://github.com/aclater/pysacd/releases/tag/v0.4.0) (2026-11-01),
[0.4.1](https://github.com/aclater/pysacd/releases/tag/v0.4.1) (2026-11-02),
[0.4.2](https://github.com/aclater/pysacd/releases/tag/v0.4.2) (2026-11-02).

### 0.3 line

pysacd moves onto scarletbook-rs; its own engine is removed.

- Its own TOC parser, demultiplexer and DST decoder are gone; scarletbook-rs, linked statically
  into the engine from a checksum-verified release, reads and decodes everything. All of pysacd
  is now GPL-2.0-or-later.
- Output is byte-identical to 0.2.2 on the corpus and real test discs, except four hostile
  corpus images (truncated audio gives silence; invalid DST frames are filled with 0x55).
- DSF extraction stays 2.5x (DST) to 3.6 to 4.0x (plain DSD) faster than `sacd_extract`.
- 0.3.1: reading a TOC is 4.9 times faster than in 0.3.0; DST extraction to DSF a little faster;
  the engine is again marked for CET shadow stacks on x86-64 Linux.
- New `--exact` option reads the area up to a far-in track instead of starting provisionally.

Releases: [0.3.0](https://github.com/aclater/pysacd/releases/tag/v0.3.0) (2026-11-01),
[0.3.1](https://github.com/aclater/pysacd/releases/tag/v0.3.1) (2026-11-01).

### 0.2 line

First releases on PyPI.

- Pure-Python TOC reader; multithreaded C engine writing DSF, DSDIFF, WAV and FLAC in one pass.
- `pysacd-extract`, a drop-in replacement for `sacd_extract` with byte-for-byte the same files
  apart from documented exceptions; `pysacd convert` with MusicBrainz tags and cover art.
- 0.2.1: `pysacd convert --pauses`. 0.2.2: DST decoding about 1.8x faster per frame; DST discs
  extract 2.4 to 2.6x faster than `sacd_extract` on all threads, 3.0x on one core; signed build
  provenance on every release file.

Releases: [0.2.0](https://github.com/aclater/pysacd/releases/tag/v0.2.0) (2026-10-29),
[0.2.1](https://github.com/aclater/pysacd/releases/tag/v0.2.1) (2026-10-29),
[0.2.2](https://github.com/aclater/pysacd/releases/tag/v0.2.2) (2026-10-30).

## beets-pysacd

### In progress

No unreleased entries in `CHANGELOG.md` and no open milestone at the time of writing.

### 0.2 line

Catalogue SACD images in place of deleted extracts, for sacdfs storage reclamation.

- New `beet sacd-migrate`: finds library albums that are extracts of an ISO and records the
  ISO, track and sacdfs path of each; report-only unless `--apply`, never deletes files.
  `--verify REPORT` accepts only albums whose extracts `sacdfs verify` found IDENTICAL or
  IDENTICAL-PAUSE; `--catalogue --mount DIR` adds an album for an ISO area with no extracts.
- New `beet sacd-tags-export --tags DIR`: writes sacdfs tag-layer files from matched albums.
- Disc fingerprints unchanged on pysacd 0.3.0, so no upgrade action; releases now carry offline
  verifiable build provenance (`.intoto.jsonl`).

Releases: [0.2.0](https://github.com/aclater/beets-pysacd/releases/tag/v0.2.0) (2026-11-02).

### 0.1 line

First releases.

- `beet sacd-import` converts each SACD area with pysacd to DSF or FLAC and imports it as an
  album; hybrid SACDs import each area on its own medium; `beet sacd-info` prints a TOC.
- DVD-Audio, DVD-Video and Blu-ray images are identified and catalogued, never converted.
- 0.1.1: `pauses: yes` option (needs pysacd 0.2.1). 0.1.2: the plugin's own imports prefer an
  SACD edition over the same album's CD edition.

Releases: [0.1.0](https://github.com/aclater/beets-pysacd/releases/tag/v0.1.0) (2026-10-29),
[0.1.1](https://github.com/aclater/beets-pysacd/releases/tag/v0.1.1) (2026-10-29),
[0.1.2](https://github.com/aclater/beets-pysacd/releases/tag/v0.1.2) (2026-10-30).

## scarletbook-rs

Summarised from its public GitHub release notes and milestones only.

### In progress: 0.6.0, then 1.0.0

Milestone 0.6.0 (open) lists performance work: plain-DSD area to file (#202), DST decode against
a reference implementation (#203), a deep performance cycle (#224), a CI benchmark subset
(#226), and integration follow-ups (#216, #228). Milestone 1.0.0 depends on the interface
freeze (#39, via 0.6.0 and one consumer cycle) and the patent-hygiene review (#40).

### 0.5 line

Output API, cancellation, seek hints and faster decoding.

- 0.5.0 to 0.5.1: remaining TOC fields through the C interface, cancellable and observable
  long reads, cheaper provisional (cold) seeks.
- 0.5.2: seek hints and saved confirmations, so a remount can seek with about one read.
- 0.5.3: `scarletbook-pcm` PCM output changes (new stopband edges, exact DSD silence).
  **On upgrade:** PCM converted with 0.5.2 or earlier and cached should be converted again.
- 0.5.4: DSF and DSDIFF writer choices for byte-compatibility with another extractor's files;
  the departures that do not conform to DSDIFF are made only when named.
- 0.5.5: DST decoding faster on generic x86-64 builds (a 50-minute stereo disc, all threads,
  6.21 s against 7.45 s) and new x86-64-v3 archives (**match release asset names exactly**).
- 0.5.6: plain-DSD output up to 3.7x faster and DSD-to-PCM about 20 % less CPU; stereo DST on
  one thread 2.9 % slower on the Zen 2 machine measured.

Releases: [0.5.0](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.0) (2026-10-31),
[0.5.1](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.1) (2026-10-31),
[0.5.2](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.2) (2026-11-01),
[0.5.3](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.3) (2026-11-01),
[0.5.4](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.4) (2026-11-01),
[0.5.5](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.5) (2026-11-01),
[0.5.6](https://github.com/aclater/scarletbook-rs/releases/tag/v0.5.6) (2026-11-02).

### 0.4 line

Speaker layouts, parallel decoding in C and provisional reading.

- Loudspeaker layouts and per-channel speaker order.
- Parallel decoding through the C interface (caller-owned thread pools).
- 0.4.1: provisional reading in the C interface and cold-seek measurements.

Releases: [0.4.0](https://github.com/aclater/scarletbook-rs/releases/tag/v0.4.0) (2026-10-31),
[0.4.1](https://github.com/aclater/scarletbook-rs/releases/tag/v0.4.1) (2026-10-31).

### 0.3 line

Frame ranges, C library archives and faster DST.

- Decoded audio of any frame range, including a track with its pause.
- C library release assets for Linux, macOS and Windows with checksums and build provenance,
  and a documented ABI compatibility policy.
- DST decoding of several frames in lockstep on one thread; an optional per-disc diagnostic log.

Releases: [0.3.0](https://github.com/aclater/scarletbook-rs/releases/tag/v0.3.0) (2026-10-31).

### 0.1 and 0.2 lines

Benchmark program `bench` for the sacd-spec benchmarks, with checksums; 0.2.0 adds signed build
provenance.

Releases: [0.1.0](https://github.com/aclater/scarletbook-rs/releases/tag/v0.1.0) (2026-10-30),
[0.2.0](https://github.com/aclater/scarletbook-rs/releases/tag/v0.2.0) (2026-10-30).
