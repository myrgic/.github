# Shared CI/CD workflow modules

Reusable GitHub Actions workflows for every `myrgic` repo. A caller references
one with `uses:` and gets a gate it does not have to re-author, re-verify, or
remember to keep current.

**Why centralize.** A gate that lives in one repo's `ci.yml` protects one repo,
and the next repo re-derives it — usually after the same incident. A module here
is written once, reviewed once, and inherited. When the rule changes, it changes
in one place.

## Available modules

| Module | Purpose | Key inputs |
|---|---|---|
| `go-ci.yml` | Go build / test / lint / optional e2e | `go-version`, `build-tags`, `run-lint`, `run-e2e`, `e2e-script` |
| `python-ci.yml` / `py-ci.yml` | Python lint / typecheck / test | `python-version`, `run-pyright` |
| `pr-checks.yml` | PR-shape checks | — |
| `fixture-hygiene.yml` | Blocks operator-identifying data in captured test fixtures | `sanitizer`, `fixture-glob`, `extra-patterns` |

## `fixture-hygiene.yml`

Use in any repo whose `testdata/` contains recordings from a **live machine**
(agent traffic, CLI stream captures, HTTP transcripts, session logs).

```yaml
jobs:
  fixture-hygiene:
    uses: myrgic/.github/.github/workflows/fixture-hygiene.yml@main
    with:
      sanitizer: internal/acp/testdata/sanitize_fixture.py
```

Two layers, and the difference between them matters:

1. **Repo sanitizer (`--verify`)** — preferred. The repo owns a script that
   understands its fixture schema and can **rebuild frames from an allowlist**.
   This is the only layer that can establish a fixture is clean.
2. **Built-in pattern scan** — schema-agnostic defense in depth: home paths,
   socket paths, plugin caches, MCP inventories, key material, credential
   tokens. It can prove a fixture is *dirty*. It **cannot** prove one is clean,
   and it says so in its own output rather than implying otherwise.

### The incident this encodes

`myrgic/cogos` PR #588 committed four golden stream-json fixtures captured from
a live workstation into a **public** repo. A redaction commit substituted the
username, and both its commit message and the fixture README stated the result
was sanitized. It was not: the fixtures still carried a private status board
embedded in a `SessionStart` hook payload (project state, ~148 task ids, two
real personal names), the full `system.init` MCP/plugin inventory naming which
third-party accounts were connected, local plugin-cache paths, and an IPC socket
path. It sat in public for three days.

The defect was **the method, not the diligence**:

> A denylist redaction cannot be verified, so it gets trusted instead of checked.
> An allowlist synthesis can be verified.

Two rules follow, and this module exists to enforce the second:

- **Rebuild, don't redact.** Construct the fixture from an allowlist of fields
  known safe. A new upstream field is then excluded *by default* — the failure
  mode becomes a missing field, never an unnoticed leak.
- **Assert the post-condition mechanically.** Verification that depends on
  someone remembering to look is not verification.

### Writing a repo sanitizer

Contract: expose `--verify` (exit non-zero on violation) and `--in-place`.
See `myrgic/cogos:internal/acp/testdata/sanitize_fixture.py` as the reference
implementation. Two properties worth copying:

- **Refuse to trade one defect for another.** It will not write if sanitizing
  would change frame `type`/`subtype`/order/count — privacy must not be bought
  by silently degrading the corpus the fixtures exist to provide.
- **Prove the detector fires before trusting it.** Run `--verify` against a
  known-dirty fixture and confirm a non-zero exit. A checker that has only ever
  said "clean" has not been tested.

### If a leak already shipped to a public repo

1. Determine whether the branch is merged. Unmerged → a branch history rewrite
   is safe and sufficient; `main` is untouched.
2. Rewrite **every** commit that touched the fixtures, not just the tip — each
   carries its own blob and each stays reachable by SHA.
3. Push with `--force-with-lease`, never `--force`.
4. Re-verify by **fetching from the remote**, not by inspecting your local copy.
5. Force-push orphans the blobs but **GitHub retains them until GC**. Old SHAs
   remain readable by anyone who recorded them. File a GitHub Support request to
   purge, and treat the data as disclosed until it completes.
