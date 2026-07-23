---
name: pr
description: Closes the current feature — format auto-fix, PR toward the index branch, merge with a merge commit and closing references, branch deletion. No per-feature server CI (it runs only on the release PR); the fast-gate is the per-feature gate.
---

# /pr — closing a feature

Closes the feature on the current branch by opening its PR toward the index branch and merging it. **There is no per-feature server CI to wait for**: the full CI (build + test + lint) runs only on the release PR `index/**`→`main` (see `workflow.md`, "Gate CI"). The per-feature gate is the local **fast-gate** hook, which already ran build + unit tests on push. Read `docs/agents/workflow.md` first if you haven't already done so in this session.

> **Structured epics only.** This command merges a **feature branch into the index**. In a *simple epic* (no feature has tickets — see `workflow.md`, "Epica semplice") there are no per-feature PRs: features land as commits on the single `epic/**` branch and the only PR is `epic/**`→`main`, opened as the **Release** step. Don't run `/pr` there.

## Preconditions

- The current branch is `feature/{n}-{slug}` (never `main`, never `index/**`).
- Every ticket of the feature has its commit with `Closes #{ticket}` in the message. Verify with `git log origin/index/{epic-n}-{epic-slug}..HEAD --oneline`: if a ticket is missing, stop and complete it first.

## Steps

1. **Format (auto-fix)** — apply the formatters and fix lint locally, don't just verify:

   | Command | Purpose |
   |---|---|
   | `dotnet format App.sln` | apply backend formatting (writes) |
   | `npm run format` (in `App/frontend`) | apply Prettier (`prettier --write`) |
   | `npx ng lint` (in `App/frontend`) | fix the findings |

   If the auto-fix changes files, commit them on the feature branch (a fix commit, without `Closes`).

2. **Push and PR toward the index** — the push triggers the local `fast-gate` (build + unit tests on the touched side); it must pass to push. The PR toward `index/**` does **not** start the server CI (only the release PR to `main` does):

   ```bash
   git push -u origin feature/{n}-{slug}
   gh pr create --base index/{epic-n}-{epic-slug} \
     --title "{feature title}" \
     --body "..."   # what the feature delivers, in Creator-facing language
   ```

   **No `Closes #{feature}` in the body**: that reference goes in the merge commit (step 4), where it creates the issue↔commit link and acts as a safety net on `main`; the actual closing happens at step 5.

3. **No per-feature CI to wait for.** The fast-gate already verified the touched side locally on push, and the server CI does not run on `index/**` PRs. Proceed straight to the merge. (Only if the push was rejected by the fast-gate do you fix, commit and push again — see "Parking" for a non-removable external blocker.)

4. **Merge** — always a merge commit, **never squash or rebase** (they rewrite the commits and lose the tickets' `Closes` references):

   ```bash
   gh pr merge --merge --delete-branch \
     --subject "Merge feature #{n}: {title}" \
     --body "Closes #{n}"
   ```

   Then switch back to the index and align it: `git switch index/{epic-n}-{epic-slug} && git pull`.

5. **Close the issues** — merged into the index = closed (this is what unblocks the frontier's `blocked_by`, see `workflow.md`):

   ```bash
   MERGE_SHA=$(git rev-parse HEAD)
   gh api --paginate repos/{owner}/{repo}/issues/{n}/sub_issues \
     --jq '.[] | select(.state == "open") | .number' \
     | xargs -I % gh issue close % --comment "Contenuta nel merge della feature #{n} nella index ($MERGE_SHA)."
   gh issue close {n} --comment "Mergiata nella index ($MERGE_SHA)."
   ```

6. **Summarize**: feature merged, issues closed, next frontier feature (or hand-off to acceptance testing if it was the last one).

## Parking — when the fast-gate stays red on a non-removable external blocker

There is no per-feature server-CI counter to trip (see `workflow.md`, "Politica di fallimento"). The per-feature gate is the local fast-gate: if it keeps failing on something that is **not a defect of the code** and that you cannot remove (a shared/dirty environment, a missing external service), apply the run's general rule — document and stop on this feature — and park it:

1. `parked` label on the feature (idempotent creation, see `issue-tracker.md`);
2. comment on the feature with the diagnosis: what's failing, what you tried, remaining hypotheses;
3. leave the feature branch as-is (unmerged); if a PR was opened, leave it **in draft** (`gh pr ready --undo`);
4. switch back to the index (`git switch index/…`) and continue with the next **unblocked** feature. Features that depend on the parked one stay blocked by construction.

The full server CI reappears only on the release PR `index/**`→`main`; a red release PR is fixed on the `index/**` branch and re-pushed.
