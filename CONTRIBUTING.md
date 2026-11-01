# Contributing


## Issues: milestones, labels and claiming

Every open issue says which release it is for, how important it is, where it stands, and
what kind of agent should work on it. This repository holds the overview; the same scheme applies to every SACD project repository. The issue list then shows what's up without opening
each issue.

- **Milestone:** the release it is planned for (`0.2.0`, `0.3.0`, ...) or `Later`. Umbrella
  issues have no milestone.
- **Priority:**
  - `P1`: must be in its milestone's release;
  - `P2`: planned for it, can slip one release;
  - `P3`: nice to have.
- **Status:**
  - `status: in progress`: claimed and being worked on;
  - `status: in review`: a PR is open;
  - `status: fixed on main`: merged. The issue stays open until the release ships and has
    been verified, then it is closed.
  - `status: needs decision`: waiting on the owner.
- **Type:** `bug`, `enhancement`, `performance`, `documentation`, `plan`, or `tracking`
  (umbrella and design issues that track other work).
- **Model:**
  - `model: opus` for hard work: concurrency, performance, design, unexplained bugs;
  - `model: sonnet` for routine work: well-specified features, tests, docs, CI.

**Claiming.** Before starting on an issue, check that nobody has claimed it. Then:
1. Comment `Claimed by <agent> (<model>)`, and add `status: in progress`.
2. If you stop without a PR, comment that you're releasing it, and remove the label.
3. When you open a PR, switch the label to `status: in review`. The PR says
   `Closes #N` only if it fully resolves the issue, else `Part of #N`.
4. Don't start on an issue that someone else has claimed. Ask on the issue instead.

