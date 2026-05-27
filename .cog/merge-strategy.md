# Merge Strategy

How the Myrgic org merges pull requests. The default is **squash**; merge-commits are reserved for curated multi-commit series. Rebase-merge is not used.

Public `main` records one commit per landed change. The full pre-squash history survives in the feature branch and local checkouts, so squashing `main` loses nothing.

## Rules

1. **Squash by default** — `gh pr merge <n> --squash --delete-branch`. One PR is one commit on `main`. Bisect-friendly, trivially revertable, and the PR is the natural unit of change.

2. **One concern per squash.** Don't bundle orthogonal changes into a single squashed PR — it defeats per-concern bisect and revert. Split unrelated changes into separate PRs; co-squash only when the changes are genuinely co-dependent.

3. **Merge-commit for a curated multi-commit series** — `gh pr merge <n> --merge --delete-branch`. Use when the branch is two or more deliberately-ordered commits forming a named unit (a phased ADR/RFC series, a staged refactor with one commit per concern). The merge commit's subject names the cluster, and the individual steps stay bisectable on `main`. "Curated" means the PR description explains why the commits are separate; absent that justification, squash.

4. **Never squash identity-referenced commits.** Release/tag commits, or any commit whose SHA is referenced elsewhere (a cross-repo pin, a document citing the commit), keep their identity. Don't collapse them.

5. **PR title is permanent history.** `--squash` uses the PR title and body as the commit message, so the title must be a clean conventional-commit subject and the body should summarize the change — not concatenate "wip" / "address review" messages.

6. **Delete the local branch after merge.** `--delete-branch` only removes the *remote* branch; a squash produces a new commit SHA, so the local branch lingers as `ahead 1, behind N` against a deleted upstream ref and `git branch -d` refuses it. After merging:

   ```
   git checkout main && git pull --ff-only && git branch -D <merged-branch>
   ```

   Periodic sweep for branches stranded this way:

   ```
   git fetch -p && git branch -vv | grep ': gone]'
   ```

   then delete the listed branches.

Rebase-merge (`--rebase`) is available but is not the org default; prefer squash (single concern) or merge-commit (named series).
