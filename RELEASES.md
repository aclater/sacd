# Releases by major version

A running summary of what each minor version line of each project brought, so progress can be
followed by major step and not only by point release. Below 1.0 the minor version is the major
step: it may change behaviour, and a patch release only adds or fixes.

This page summarises public release notes only. The sources are each project's `CHANGELOG.md`,
its GitHub releases and its milestones; for any detail, they win. Newest first. Last updated
2026-11-14.

| Project | Latest release | In progress |
|---|---|---|
| [sacdfs](#sacdfs) | 0.6.3 (2026-11-13) | nothing recorded |
| [pysacd](#pysacd) | 0.5.1 (2026-11-14) | nothing recorded |
| [beets-pysacd](#beets-pysacd) | 0.2.1 (2026-11-14) | nothing recorded |
| [scarletbook-rs](#scarletbook-rs) | 0.5.8 (2026-11-05) | 0.6.0, then 1.0.0 |

To keep this page current, each project's release checklist includes "update aclater/sacd
RELEASES.md".

## sacdfs

### 0.6 line

NAS deployment: start and stop at boot, one Compose file, plain start-up messages.

**0.6.3** (2026-11-13):

- A player beside four copies on a server too slow to decode them all at real time stalls far
  less than on 0.6.2 or 0.6.1: in the gate, worst read 3.4 s on 2 slow cores over SMB against
  12.3 s and 7.7 s ([#357](https://github.com/aclater/sacdfs/issues/357)). A second player starting
  beside copies is slower to start than on 0.6.2 ([#363](https://github.com/aclater/sacdfs/issues/363)).
- The installer fails loudly on a download cut short ([#352](https://github.com/aclater/sacdfs/issues/352)),
  switches QNAP's autorun back off on uninstall ([#353](https://github.com/aclater/sacdfs/issues/353)),
  and stops at once on a system other than Linux.
- The docs advise `rsize=262144` on Linux SMB mounts: a tag scan then costs about a seventh of the
  server CPU ([#266](https://github.com/aclater/sacdfs/issues/266)).

Tested as 0.6.3-rc.1 against 0.6.2: a 2-hour soak on two machines, stop checks, release artifacts,
the installer in containers and on Synology and QNAP in VMs, Tier 1 media servers, seven Linux client
distributions, a player beside copies over SMB, Windows 11 in a VM and macOS 27.2 in a VM on Apple
hardware (where a player beside four copies stalls on both releases, [#378](https://github.com/aclater/sacdfs/issues/378)).

**0.6.2** (2026-11-11):

- An installer: one `curl ... | sh` line finds out what it runs on (Synology, QNAP, ASUSTOR,
  TerraMaster, UGREEN, ReadyNAS, OpenMediaVault or plain Linux) and sets sacdfs up the way that
  system's guide does ([#340](https://github.com/aclater/sacdfs/issues/340)).
- WAV copies over SMB are paced so a player beside them keeps its reads short, and a single copy is
  faster again ([#271](https://github.com/aclater/sacdfs/issues/271)). On a server too slow to decode
  several copies at real time, a player beside four copies stalls longer than on 0.6.1
  ([#357](https://github.com/aclater/sacdfs/issues/357)).
- sacdfs exits after an outside unmount ([#322](https://github.com/aclater/sacdfs/issues/322)), and a
  stop signal during start-up is no longer lost ([#334](https://github.com/aclater/sacdfs/issues/334)).
- The core is split from the FUSE layer, a first step towards Windows support
  ([#22](https://github.com/aclater/sacdfs/issues/22)); no behaviour change.

Tested as 0.6.2-rc.1 against 0.6.1: a 2-hour soak on two machines, stop checks, release artifacts,
the installer on Linux and on Synology and QNAP in VMs, Tier 1 media servers, seven Linux client
distributions, Windows 11 in a VM, and a player beside copies over SMB and NFS. macOS was not run.

**0.6.1** (2026-11-11):

- On SIGTERM, sacdfs now exits after a grace period even while clients hold files open
  (`--stop-grace`, 10 s by default; [#276](https://github.com/aclater/sacdfs/issues/276)), so a
  service manager's stop no longer waits for ever.
- DST areas read faster on one thread: scarletbook is built in one codegen unit
  ([#270](https://github.com/aclater/sacdfs/issues/270)).
- Simpler install guides for players and NAS systems
  ([#310](https://github.com/aclater/sacdfs/issues/310)), and notes on SMB client settings
  ([#266](https://github.com/aclater/sacdfs/issues/266), [#325](https://github.com/aclater/sacdfs/issues/325)).
- A faster pace for WAV copies over SMB ([#271](https://github.com/aclater/sacdfs/issues/271)) was
  taken out before the release because it could stall a player beside several starting copies.

Tested on main against 0.6.0: a 2-hour soak on two machines, the stop check, and a player beside
whole-file copies over SMB.

**0.6.0** (2026-11-10):

- Safe start and stop for the release binary on a NAS: a systemd unit and a `sacdfs-ctl` script for
  systems without systemd, now shipped in the release archives
  ([#289](https://github.com/aclater/sacdfs/issues/289)). Start at boot was checked on QNAP, ASUSTOR,
  Synology, TerraMaster and ReadyNAS systems in VMs.
- One Compose file for every NAS container manager
  ([#283](https://github.com/aclater/sacdfs/issues/283)); Unraid and TrueNAS app-store entries
  prepared, not yet submitted ([#284](https://github.com/aclater/sacdfs/issues/284)).
- Plain start-up messages and checks for NAS users, and root's mount is readable by other users
  ([#282](https://github.com/aclater/sacdfs/issues/282)).
- Directory listings over SMB cost sacdfs about a third fewer requests
  ([#205](https://github.com/aclater/sacdfs/issues/205)).
- Strict real-disc tests that run in the release gate
  ([#220](https://github.com/aclater/sacdfs/issues/220)).

Tested as one release candidate: five music servers and NFS and SMB clients on seven Linux
distributions, Windows 11 and macOS in VMs, real discs, a whole-library comparison with 0.5.0 and a
2-hour soak on two machines, with no regression against 0.5.0.

### 0.5 line

**0.5.0** (2026-11-08): faster DST reads through the mount
([#210](https://github.com/aclater/sacdfs/issues/210)); players on the same host read the mount
directly ([#186](https://github.com/aclater/sacdfs/issues/186)); MusicBrainz tags on demand and an
optional background lookup ([#185](https://github.com/aclater/sacdfs/issues/185)); sacdfs no longer
gets killed listing under a memory limit ([#237](https://github.com/aclater/sacdfs/issues/237)); release
binaries for glibc 2.17 and aarch64 ([#244](https://github.com/aclater/sacdfs/issues/244)); a
read-only share with the right sizes ([#217](https://github.com/aclater/sacdfs/issues/217)).
Tested as one release candidate against 0.4.1.

### 0.4 line

Memory and leaks, and crates.io.

- Lower memory: the default frame cache is 32 MiB, and fixed overhead is cut
  ([#150](https://github.com/aclater/sacdfs/issues/150)). Peak memory of a full-library import
  at the default cache was 616 MiB on 0.1.0 and is 61 MiB on 0.4.0. An import-like scan fell from
  109 to 45 MiB against 0.3.0, and the first listing of a 200-image library is 55 % faster.
- Leak hunting ([#151](https://github.com/aclater/sacdfs/issues/151)): a soak harness for real
  discs, a no-growth test on every pull request, and a weekly memcheck and corpus soak. They found
  and fixed three leaks that the small test images could not show:
  - removed or renamed images stayed in the tree
    ([#153](https://github.com/aclater/sacdfs/issues/153));
  - the seek-index cache kept every disc ever opened
    ([#169](https://github.com/aclater/sacdfs/pull/169));
  - spare worker threads never ended ([#167](https://github.com/aclater/sacdfs/issues/167)).

  A library with albums added and removed under new names grew about 94 MiB/h and now levels off
  over 2 hours. An open file now keeps reading after its image is removed.
- `ls -l` and `getfattr -d` over the container's NFS export no longer print "Unknown error 524"
  ([#174](https://github.com/aclater/sacdfs/issues/174)). rc.4's first fix made Roon over SMB send
  21,767 extended-attribute requests per rescan; rc.5 keeps that at 0.
- scarletbook 0.5.7 from crates.io ([#157](https://github.com/aclater/sacdfs/pull/157)); sacdfs
  itself is on crates.io from 0.4.0 (`cargo install --locked sacdfs`,
  [#155](https://github.com/aclater/sacdfs/issues/155)).
- A shorter README, with details moved to docs/
  ([#159](https://github.com/aclater/sacdfs/issues/159)).

Tested as five release candidates on a TrueNAS SCALE NAS with Roon and Plex, a Fedora NFS client
with SELinux enforcing, five music servers and a 2-hour soak on real discs.

**0.4.1** (patch release; it adds and fixes, and a scanner sees one-time changes on upgrade):

- `sacdfs verify` no longer exits 0 with files unchecked after a worker panic, and a stopped run
  leaves no report that looks finished ([#196](https://github.com/aclater/sacdfs/issues/196));
  `sacdfs tags import` exits 2 when some extracts weren't imported
  ([#198](https://github.com/aclater/sacdfs/issues/198)).
- Folders with no SACD image are hidden, art included
  ([#175](https://github.com/aclater/sacdfs/issues/175)).
- Discs without text get names and tags with no network: `Track NN`, and album and artist from the
  image's path ([#185](https://github.com/aclater/sacdfs/issues/185), phase 1). Files of such
  discs change once.
- MPD finds folder art ([#148](https://github.com/aclater/sacdfs/issues/148)).
- Container: `SACDFS_MOUNT_TIMEOUT` (default 300 s) for large libraries on slow disks
  ([#232](https://github.com/aclater/sacdfs/issues/232)); a "Small NAS: sizing" guide.
- A hung image's WAV reads no longer take decoder threads from other images, so reads of healthy
  discs no longer stall and fail with `EIO` while one source hangs
  ([#238](https://github.com/aclater/sacdfs/issues/238); present in 0.4.0 too; audio bytes unchanged).
  Known limit: a long hang with clients that keep retrying can still use every read thread of the
  mount ([#241](https://github.com/aclater/sacdfs/issues/241), planned for 0.5.0).
- Memory after the start-up walk is back to 0.4.0's
  ([#224](https://github.com/aclater/sacdfs/pull/224)).
- scarletbook 0.5.8; fuzz targets, an informational benchmark job, a fairer soak thread check
  ([#161](https://github.com/aclater/sacdfs/issues/161),
  [#162](https://github.com/aclater/sacdfs/issues/162),
  [#179](https://github.com/aclater/sacdfs/issues/179)).

Tested as three release candidates, with a new [release test plan](https://github.com/aclater/sacdfs/blob/main/docs/test-plan.md):
Roon and Plex on a whole library, five scripted servers against 0.4.0, Windows 11, foobar2000,
JRiver Media Center, Linux NFS and SMB clients and the FFmpeg family of tools (rc.1); the rc.2 gate; rc.3 then added the #238 fix:

- Memory on a real library of 4,902 DSF files (32-core machine), listed / after a head-and-tail
  read of every file / 10 minutes idle: 0.4.0 29.0 / 79.5 / 35.2 MiB, rc.1 33.2 / 95.1 / 54.0,
  rc.2 27.2 / 72.7 / 33.0.
- Whole-library mount (warm NFS, fresh index cache, medians of 5 alternating rounds): 0.4.0 5.00 s,
  rc.1 0.76 s, rc.2 1.02 s. First listing 1.55 / 1.36 / 1.42 s. rc.2 is about 0.26 s slower than
  rc.1 (accepted; the arena limit is the suspected cause, not confirmed). Cold-cache mount not
  measured on rc.2.
- 60-minute soak passed: 224,491 reads, 686 GB, 0 unexpected errors, 19 stalls, 11 remounts, memory
  +1.4 MiB/h. The 2-hour soak on rc.2 failed on #238, which rc.3 fixes. On rc.3, every soak had 0 unexpected errors (rc.2 beside it: 0 to 14). Two 2-hour runs on one machine failed the memory trend check on single steps (+8.02 and +8.90 MiB/h, limit 4.4); 6- and 8-hour runs passed, flat at +0.08 and +1.14 MiB/h. The cause of the steps is unknown ([#247](https://github.com/aclater/sacdfs/issues/247)).
- SMB 3.1.1 and NFS 4.2 listings correct, 10 files byte-identical over both, writes refused, SMB
  reconnects after a restart. `SACDFS_MOUNT_TIMEOUT` and the health check pass on the rc.2 image
  under Docker and rootful Podman (`docker compose` not run).
- Upgrade rc.1 to rc.2 on the NAS apps: identical names and counts, no re-read seen for 12 minutes.
- Not verified: the arm64 image at run time, a cold-cache mount.

Known limit: an existing NFS mount gets "Stale file handle" after a container restart under rootful
Podman ([#218](https://github.com/aclater/sacdfs/issues/218)); the fix is planned for 0.5.0
([#221](https://github.com/aclater/sacdfs/pull/221)).

**Quality scorecard, 0.4.1** (first one; facts only, no score).

1. *Bugs that escaped 0.4.0, and the check that should have caught them.* The severity is this
   page's reading of each issue's text, not a label from the issues.

   | Issue | What | Severity | Fixed in 0.4.1 | Check that should have caught it |
   |---|---|---|---|---|
   | [#196](https://github.com/aclater/sacdfs/issues/196) | `verify` exited 0 after a worker panic, with that file missing from the report | High: false success, and users delete extracts on `verify`'s word | Yes | Fuzzing of the `verify` readers and an error-path test (found by the new fuzz targets) |
   | [#198](https://github.com/aclater/sacdfs/issues/198) | `tags import` exited 0 when files failed | Low: scripting only, nothing is deleted on its word | Yes | A test of each exit code |
   | [#202](https://github.com/aclater/sacdfs/issues/202) | `verify -o /dev/stdout` redirected to a file failed | Low. The issue says the failing form was introduced on main by the #196 fix, before the release; the changelog says 0.4.0 wrote the rows and then exited 1. So not an escape of the same kind | Yes | A test with stdout redirected to a regular file (the pipe case was tested) |
   | [#204](https://github.com/aclater/sacdfs/issues/204) | One SMB 3 client drops the mount after a sacdfs restart of about 8 s (2 of 2 tries) | Medium for that client; remounting works. Not classified further: the cause is unknown | No: open | The test plan's restart check on every client (the plan is new) |
   | [#217](https://github.com/aclater/sacdfs/issues/217) | The SMB share does not advertise read-only and looks 100 % full | Low: cosmetic, writes are refused cleanly. The issue says it is not a regression | No: open | A check of what file managers show |
   | [#218](https://github.com/aclater/sacdfs/issues/218) | NFSv4 clients get "Stale file handle" after a Podman restart (Docker keeps them) | Medium: the docs said otherwise | No: documented as a known limit, fix planned for 0.5.0 | A restart test against the container's own NFS server; CI used the kernel's server over NFSv3 |
   | [#232](https://github.com/aclater/sacdfs/issues/232) | The container gave up after 60 s mounting a large library on a slow disk | Medium: the container exits and does not start | Yes (`SACDFS_MOUNT_TIMEOUT`) | A large-library, slow-disk test (the 122,676-file case was an emulated disk) |

2. *Wrong data.* Checked: copies over SMB were byte-identical to the served files (186 of 186 audio
   files on Windows in rc.1); DSF and WAV files from the 136-image corpus and five bench discs hash the same as
   with 0.4.0 after the scarletbook 0.5.8 move; on rc.2, 10 files were byte-identical over SMB 3.1.1
   and NFS 4.2; a tree comparison of the whole library against 0.4.0 found the changes listed in the upgrade notes
   (names and tags of discs without text, added cover images, hidden folders with no audio).
   Result: no wrong audio or wrong data was found. Not covered: Japanese, Chinese and Korean disc
   text, and the other layouts listed in the test plan.

3. *Regressions against 0.4.0.* Memory after a full read is lower (72.7 against 79.5 MiB on the
   gate library) after rc.1's rise to 95.1 was fixed ([#224](https://github.com/aclater/sacdfs/pull/224)).
   The whole-library mount is faster (1.02 against 5.00 s warm), though 0.26 s slower than rc.1.
   Straight reads through the mount were unchanged within noise on the corpus (DST stereo DSF 8.75
   to 8.72 s; plain-DSD DSF 2.83 to 2.55 s; medians of 11). Jellyfin's loudness scan was 6 % slower on rc.1
   (508 s against 478 s, one run each, not repeated on rc.2). A scanner sees one-time changes on
   upgrade (listed in the upgrade notes). Cold-cache mount time was not measured on rc.2.

4. *Gaps.* No physical Windows client (Windows 11 ran in VMs; a Robocopy speed difference of 3.3x
   between two modes is unexplained, [#225](https://github.com/aclater/sacdfs/issues/225)); no
   real N100-class hardware (small-NAS sizing was measured on limited containers); cold-cache mount;
   the arm64 image at run time; the compose file (`docker compose` not run); what the rc.3 soak's memory steps are
   (#247); a long hang with retrying clients, which no soak covers (#241); Jellyfin, Windows and macOS on rc.3, carried over from earlier candidates; and the coverage gaps listed in the
   [test plan](https://github.com/aclater/sacdfs/blob/main/docs/test-plan.md) (Kodi, Gerbera, Strawberry, Sonos, phones and
   streamers, macOS Finder and players, discs with unusual layouts).

Releases: [0.4.1](https://github.com/aclater/sacdfs/releases/tag/v0.4.1) (2026-11-07),
[0.4.0](https://github.com/aclater/sacdfs/releases/tag/v0.4.0) (2026-11-03).

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

### 0.5 line

Trust in what shipped: the comparison with `sacd_extract` works in CI again.

- **0.5.1 is a security release** ([notes](https://github.com/aclater/pysacd/releases/tag/v0.5.1)).
  A private review of 0.5.0 found three problems a crafted disc image can trigger, each with a
  published advisory: output placed outside the output folder
  ([GHSA-gx45-h8q4-vwmc](https://github.com/aclater/pysacd/security/advisories/GHSA-gx45-h8q4-vwmc)),
  one disc replacing another disc's files without a warning
  ([GHSA-whv9-98xg-4xw9](https://github.com/aclater/pysacd/security/advisories/GHSA-whv9-98xg-4xw9)),
  and a small image producing very large output
  ([GHSA-q45f-7qmf-37x3](https://github.com/aclater/pysacd/security/advisories/GHSA-q45f-7qmf-37x3)).
  Smaller findings on network requests, credentials and output handling are fixed with them.
  Upgrade if you convert images you did not rip yourself.
  - Changed defaults: a run stops when it would write mostly silence or filler for audio the image
    lacks (more than 10 seconds beyond the audio found; `--allow-silence` gives the old behaviour
    for a run), and `pysacd-extract --templates` keeps files that exist (`--overwrite` replaces
    them). Engine ABI 7.
  - Output for ordinary discs is unchanged: 0.5.0 against 0.5.1 on three real discs in 35 modes
    (105 runs), no difference, and none from `sacd_extract` beyond documented exceptions.
  - Also in 0.5.1: Python 3.10 again and the engine builds with GCC 11
    ([#140](https://github.com/aclater/pysacd/issues/140)); macOS wheels for macOS 11 (arm64) and
    10.13 (x86-64) ([#142](https://github.com/aclater/pysacd/issues/142); not yet run on a macOS
    older than 15); scarletbook-rs 0.5.9 linked; one worker thread for plain DSD to DSF or DSDIFF
    ([#137](https://github.com/aclater/pysacd/issues/137)); CI builds and tests in Fedora and UBI
    containers ([#144](https://github.com/aclater/pysacd/issues/144)).
- CI did not compare pysacd with `sacd_extract` for releases 0.3.0 to 0.4.3: a check for the engine
  looked for a function it no longer exported, so 44 tests skipped in a job that stayed green
  ([#131](https://github.com/aclater/pysacd/issues/131)). Checked by hand on 5 real discs, 175 runs:
  no differences from 0.4.3 but the fixed `-P` last line, and none from `sacd_extract` beyond
  documented exceptions. CI now fails if tests start skipping unexpectedly.
- CI checks the C engine for leaks ([#122](https://github.com/aclater/pysacd/issues/122)), has an
  informational benchmark job ([#123](https://github.com/aclater/pysacd/issues/123)) and tests
  Python 3.15 ([#132](https://github.com/aclater/pysacd/issues/132)).
- Links scarletbook-rs 0.5.8: output and speed unchanged.

### 0.4 line

pysacd-extract on scarletbook's writers.

- DSF and DSDIFF files are laid out by scarletbook-rs 0.5.4's output API; pysacd's own writers
  are deleted. Output is byte-identical to 0.3.1 in every mode.
- 0.4.1: faster DST decoding via scarletbook 0.5.5; DST stereo to DSF 16 % faster on a 16-core
  machine, output byte-identical.
- 0.4.2: DSF audio laid out by scarletbook-rs 0.5.6's layout fill (pysacd's own block layout
  deleted); plain DSD to DSF 2.1 times faster on NVMe, DST about 1.5 % faster, output
  byte-identical ([#114](https://github.com/aclater/pysacd/issues/114)).
- 0.4.3: links scarletbook-rs 0.5.7, winning back 0.5.6's one-thread DST stereo regression
  (DST stereo to DSF on one thread 32.94 s against 33.50 in 0.4.2), output byte-identical
  ([#119](https://github.com/aclater/pysacd/pull/119)).
- Workaround for early Sony discs whose last area frame is missing: filled with silence, warns,
  exits 0 instead of 1.
- Behaviour change: a file scarletbook won't lay out from a hostile or damaged TOC is reported
  and not written, and the run exits 255 (0.3.1 wrote it from the TOC as it stood).

Releases: [0.4.0](https://github.com/aclater/pysacd/releases/tag/v0.4.0) (2026-11-01),
[0.4.1](https://github.com/aclater/pysacd/releases/tag/v0.4.1) (2026-11-02),
[0.4.2](https://github.com/aclater/pysacd/releases/tag/v0.4.2) (2026-11-02),
[0.4.3](https://github.com/aclater/pysacd/releases/tag/v0.4.3) (2026-11-02).

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

- **0.2.1 is a security release** ([notes](https://github.com/aclater/beets-pysacd/releases/tag/v0.2.1))
  for users of `remove_iso` (off by default): an image could be deleted without having been
  imported, because "already in the library" is decided by a fingerprint of the TOCs and the size,
  not the audio
  ([GHSA-mjx6-hpg7-mjqj](https://github.com/aclater/beets-pysacd/security/advisories/GHSA-mjx6-hpg7-mjqj)).
  Now an image is removed only when every selected area was imported from it in that run and its
  own album verified; anything else keeps the image and says why.
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

- 0.5.8: seek index loader refuses a form the library never writes; `bench realtime`; corrected benchmark runner; `scarletbook-ffi` on crates.io. Reports no speed change.

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
