# Decisions

One line per major decision, with why and a link to where it is recorded in full. Newest
first.

| Date | Decision | Why | Record |
|---|---|---|---|
| 2026-10-31 | One overview repository and a cross-repository board; repositories stay where they are | One front door for the projects. Moving them into an organisation would break the clean environment's access tokens, PyPI trusted publishing and the CI services. | this repository |
| 2026-10-31 | sacdfs: no DoP; FLAC deferred; DSF and WAV side by side; 176.4 kHz / 24-bit default; `--pcm-gain` offered (0 dB default) | DoP files play as noise on equipment that doesn't support it, and players already make DoP from DSF. Compressed FLAC's size is unknown until encoded, which breaks a fast filesystem. 176.4 kHz is DSD64 divided by exactly 16. The 0 dB default can't clip. | [sacdfs#11](https://github.com/aclater/sacdfs/issues/11) |
| 2026-10-31 | sacdfs PCM: band-limit to 30–40 kHz (default 40) following Daniel Weiss; ~105–110 dB stopband accepted | No published dB standard exists. Weiss (Saracon) is the most authoritative voice: a proper decimation filter keeps DSD's noise out. Independent measurements found converters of this class transparent. A deeper stopband costs speed for nothing shown to be audible. | [sacdfs#11](https://github.com/aclater/sacdfs/issues/11) |
| 2026-10-31 | sacdfs relicensed to GPL-3.0-or-later | Lets it draw on GPL and LGPL code, such as pysacd's DSD-to-PCM converter. GPL-3.0 also accepts Apache-2.0 and GPL-3.0 code, which GPL-2.0 does not. SACD parsing and DST still come only from scarletbook. | [sacdfs#12](https://github.com/aclater/sacdfs/pull/12) |
| 2026-10-31 | sacdfs started early, before scarletbook 1.0, as a new public project | Owner's call. The performance of scarletbook makes an on-demand filesystem practical. | [sacdfs design](https://github.com/aclater/sacdfs/blob/main/docs/design.md) |
| 2026-10-31 | sacdfs keeps its own seek code instead of scarletbook's range API | The range API rescans from the start of the area (up to 132 ms per seek, against under 2 ms). A position-independent seek was requested upstream. | [scarletbook-rs#68](https://github.com/aclater/scarletbook-rs/issues/68) |
| 2026-10-30 | scarletbook-rs: no patents read, pre-2006 sources preferred, lawyer review before 1.0 | Avoids wilful-infringement exposure and keeps techniques to expired or public-domain prior art. | [scarletbook-rs#40](https://github.com/aclater/scarletbook-rs/issues/40) |
| 2026-10-30 | scarletbook-rs licensed MIT OR Apache-2.0 | The Rust ecosystem's norm. Apache-2.0 adds a patent grant, and MIT keeps GPL-2.0 consumers possible. | [scarletbook-rs README](https://github.com/aclater/scarletbook-rs) |
| 2026-10-30 | Performance goal: significantly faster than `sacd_extract` | It is the standard tool; the targets are in the benchmark document. Met: 2.8x to 4.0x. | [benchmarks](https://github.com/aclater/sacd-spec/blob/main/benchmarks/README.md) |
| 2026-10-30 | Upstream contributions as findings, not patches | The owner is product owner, not a developer who can defend a patch in review. Measured findings serve maintainers better. | — |
| 2026-10-30 | Clean-room split: sacd-spec (spec) and scarletbook-rs (clean implementation) | pysacd is GPL. A permissively licensed library usable by anyone needs an independent derivation, written only from a specification. | [sacd-spec README](https://github.com/aclater/sacd-spec) |
