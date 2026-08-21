# This repo still lives in iCloud. Read before you move it.

**Status: NOT YET MOVED.** `~/Documents` has Desktop & Documents sync ON, so every repo
here grows iCloud conflict copies — files named `foo 2.py` beside `foo.py`, untracked and
not gitignored. They shadow real modules: a grep returns two answers and you can quote the
stale one without noticing. Anything that walks the working tree (an AST check, a linter, a
test that globs) sees them too.

Millages was moved out on **2026-08-21** to `~/src/millages`. It went cleanly. This note
records what that migration measured, so the next one does not rediscover it.

## Measured state of THIS repo, 2026-08-21

| | |
|---|---|
| path | `~/Documents/hass-auto-update` |
| size | 164K |
| conflict copies present | **0** |
| uncommitted (tracked) | 0 |
| untracked | 0 |
| own `.venv` | no |
| remote | present — history recoverable if the move goes wrong. |

It is an ordinary repo — history in `.git`, exactly one worktree — so moving it is `mv` plus path
fixes. Simpler than Millages, which was a worktree of a bare repo that already lived outside iCloud.

## Procedure that worked

1. **Commit or stash everything first.** Check `git status` AND `git log @{u}..HEAD` for unpushed
   commits. If there is no remote (see the table above), bundle to a non-iCloud path first.
2. **Move real untracked files out** — anything you want that git does not track.
3. **Quarantine conflict copies by MOVING them aside, not deleting.** Reversible until you have
   confirmed nothing real was in them.
4. `mv` the repo to its new home. Suggested `~/src/<lowercase-name>`, matching `~/src/millages`.
5. **Fix the paths the move breaks** — see below. This is the part that bites.
6. **Verify by USING it** — run the build/tests from the new path. Not by inspection.

## The gotchas, all measured during the Millages move on 2026-08-21

- **A Python `.venv` does NOT survive a move untouched.** Millages had **32 console scripts** in
  `.venv/bin/` carrying absolute shebangs, and **40 symlinks** from other checkouts pointing into
  that one venv. Every one needed rewriting or re-pointing. **A broken venv presents as a confusing
  interpreter error, not as "the repo moved"** — which is exactly why it costs hours. Often
  simplest: delete the venv and recreate it at the new path.
- **Absolute paths hide outside code too**: `.claude/settings*.json`, hooks, launchd/cron jobs, IDE
  configs, `.env` files, and any script with a hardcoded `~/Documents/...`.
- **The destination is not empty, and two names already COLLIDE.** `~/src` contains `rubarb`;
  `~/Developer` contains `homebrew-phil`. Both share a name with a repo still in `~/Documents`.
  **Check whether each is the same checkout or a second one before moving onto it.** A duplicate
  checkout outside iCloud recreates the exact "a grep returns two answers" problem the move exists
  to fix.
- **Move any disaster-recovery bundles OUT FIRST.** In the Millages move, both bundles were sitting
  inside the very tree being moved.
- **iCloud Drive is not one tree.** `~/Documents` and iCloud's `Documents` are the same content
  under two paths — but `~/Downloads` and iCloud's `Downloads` are **entirely different folders
  sharing a name** (2 entries vs 260, zero overlap), and the iCloud one is the real archive. So a
  sweep of `~/Documents ~/Downloads ~/Desktop` is **not** a sweep of iCloud Drive. A negative
  claim about what is "on disk" that skipped iCloud Drive has already been wrong once.

## Do not

- Do not move a repo while another session holds work in it.
- Do not delete conflict copies before confirming nothing real is in them — move them aside.
- Do not do all the remaining repos in one pass. Each carries its own path fallout.

---
*Written 2026-08-21 by the Millages reviewer session, after moving Millages out. Delete this file
once this repo has been moved.*
