# Security policy

This repository holds no code. A vulnerability belongs to the project it is in, and each
project has its own policy with its scope and what a report should contain. This page says
where to report, what all five projects promise, and how a vulnerability is handled.

## Where to report

Report privately, through GitHub's private vulnerability reporting for the project concerned.
Please don't open a public issue, pull request or discussion for a vulnerability.

| Project | Report privately | Policy |
|---|---|---|
| sacdfs | [new advisory](https://github.com/aclater/sacdfs/security/advisories/new) | [SECURITY.md](https://github.com/aclater/sacdfs/blob/main/SECURITY.md) |
| pysacd | [new advisory](https://github.com/aclater/pysacd/security/advisories/new) | [SECURITY.md](https://github.com/aclater/pysacd/blob/master/SECURITY.md), [threat model](https://github.com/aclater/pysacd/blob/master/docs/security.md) |
| beets-pysacd | [new advisory](https://github.com/aclater/beets-pysacd/security/advisories/new) | [SECURITY.md](https://github.com/aclater/beets-pysacd/blob/main/SECURITY.md) |
| scarletbook-rs | [new advisory](https://github.com/aclater/scarletbook-rs/security/advisories/new) | [SECURITY.md](https://github.com/aclater/scarletbook-rs/blob/main/SECURITY.md) |
| sacd-spec | [new advisory](https://github.com/aclater/sacd-spec/security/advisories/new) | [SECURITY.md](https://github.com/aclater/sacd-spec/blob/main/SECURITY.md) |

If you don't know which project it is in, report it to the tool you were running; it will be
moved if it belongs elsewhere. Private reporting is also on for
[this repository](https://github.com/aclater/sacd/security/advisories/new), for a problem that
spans projects.

If you aren't sure whether a problem is a security problem, report it privately anyway: a
private report can be made public later, and a public one can't be made private. If you can't
use GitHub's private reporting, open an issue that says only that you have a security report
and need a private way to send it, with no details.

Don't send copyrighted disc contents. A synthetic image, or the few sectors involved, is
enough.

## What to expect

The projects are maintained by one person, so these are targets, not guarantees:

- **acknowledgement** within 7 days;
- **assessment** (confirmed or not, and how severe) within 14 days;
- **a fix** for a confirmed issue within 90 days, usually much sooner, in a new release.

## Coordinated disclosure

Please keep the details private until a fixed release is out, or until 90 days after your
report, whichever comes first; if a fix needs longer, a date is agreed with you. If the problem
is already public or is being exploited, say so: the fix and the advisory then go out as soon
as they can.

Once it's fixed, the issue is published as a GitHub security advisory (with a CVE when it
warrants one), saying which versions are affected and which release fixes it, and crediting
you unless you'd rather not be named.

## How a vulnerability is handled

The same steps apply in every project, whether the vulnerability came from a report or was
found by a contributor, an agent or an internal review.

1. **Private from the first moment.** Whoever finds it opens a draft advisory in the
   project's repository. Nothing about it goes into an issue, a pull request, a discussion, a
   commit message, a branch name, the [project board](https://github.com/users/aclater/projects/1)
   or the log of a public CI run. The issue scheme in [CONTRIBUTING.md](CONTRIBUTING.md)
   (milestones, labels, claiming) doesn't apply; the draft advisory is the record.
2. **Assess.** Reproduce it; record the affected versions and the severity (the advisory's
   CVSS score); tell the reporter. Whether something is a vulnerability or ordinary hardening
   is the maintainer's call, not the finder's. Hardening with no security impact becomes a
   normal issue.
3. **Fix in private,** in the advisory's temporary private fork. GitHub Actions don't run
   there, so the project's checks are run by hand and the results noted in the advisory. The
   fix comes with a test that fails without it.
4. **Release,** with a "Security" entry in the changelog that names the advisory and says what
   to upgrade to, without a recipe for the attack.
5. **Publish the advisory** once the release is out, with a CVE if it warrants one and credit
   to the reporter. Follow-up hardening can then be discussed in public.

**Across the clean-room wall.** The rule in [README.md](README.md#the-clean-room-wall) holds
for vulnerabilities too. Someone who has read pysacd's or sacdfs's code and finds that the
cause is in scarletbook doesn't open anything against scarletbook-rs. They say so in the
advisory of the project they were working on, and the maintainer takes it to scarletbook-rs as
a private statement of the behaviour and the input, with no code. The dependent project moves
to scarletbook's fixed release once it is out. Outside reporters can report to scarletbook-rs
directly.
