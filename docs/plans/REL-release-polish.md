# Implementation Plan: [REL] Release polish for v0.1

**Backlog ID:** REL-1 … REL-8 (new `docs/GOMBIT_BUILD_PLAN.md` §4 section — see §2 below)
**Milestone:** `post-v0.1`
**Depends on:** M0–M5 + ADMIN-1..3 (all on `main`)
**Size:** L (8 PR slices) · Labels: `docs`, `ci`, `cli`

> Scope flag (AGENTS.md, "If something looks missing from the backlog, flag it"):
> **none of this work exists in build plan §4 today.** §4 stops at the M6/post-v0.1
> batteries and never covers packaging, README/onboarding, issue templates, or a
> release pipeline. This plan therefore adds a new §4 subsection before any issue
> is opened (REL-0, §2) rather than silently creating out-of-backlog issues.

---

## 1. Goal

Take the repo from "complete framework, internal-facing docs" to "a stranger can
find it, understand it, install it, build something with it, and consume a
tagged release."

**Acceptance criteria for the epic:**

- [ ] README leads with badges, a positioning paragraph, and a copy-pasteable
      quickstart — not a list of doc links
- [ ] A `docs/tutorial.md` walks the full v0.1 loop end to end and is backed by a
      committed, CI-built example app
- [ ] `docs/installation.md` covers `go install`, release binaries, and the
      Atlas/Node prerequisites
- [ ] `.github/ISSUE_TEMPLATE/` ships bug report, feature request, and question
      forms
- [ ] A `Release` workflow cross-compiles `gombit` and publishes checksummed
      archives on a `v*` tag
- [ ] `gombit version` reports the release version stamped by that workflow
- [ ] CONTRIBUTING covers the outside contributor path, not just the agent path

---

## 2. Current baseline (audited 2026-08-18)

| Area | State | Gap |
| --- | --- | --- |
| `README.md` | 68 lines; 3 short code blocks then ~25 lines of doc links | No badges, no positioning, no quickstart, no feature list; every command is spelled `go run ./cmd/gombit …` (contributor form, not user form) |
| `CONTRIBUTING.md` | 43 lines; local checks + working agreement | No fork/branch/PR flow, no commit convention, no test-matrix instructions, no code-of-conduct link, no review-skill mention |
| `.github/` | `pull_request_template.md`, `workflows/ci.yml` only | No issue templates, no `SECURITY.md`, no `CODE_OF_CONDUCT.md`, no release workflow, no Dependabot |
| CI | `ci.yml` — lint → build → test → 9 parallel matrix jobs (sqlite/pg/mysql × database/migrations/conformance) + contract-drift. Solid. | No release job |
| Versioning | **No git tags.** No `Version` var, no `version` command, no `--version` flag in `cli/root.go` | Release binaries would be unstamped |
| `docs/` | 18 topic docs, 4 ADRs, well-written but reference-style | No index, no installation guide, no tutorial, no getting-started narrative |
| `examples/` | 15 per-feature dirs (`auth`, `admin`, `contract`, …), each a focused `main.go` | No end-to-end app tying them together |
| `LICENSE` | **MIT**, © 2026 LAA Software Engineering | Badge must say MIT (ACP's says Apache — do not copy verbatim) |
| `go.mod` | `go 1.25.7`, module `github.com/LAA-Software-Engineering/gombit` | Badge should read Go 1.25+ |

### REL-0 — backlog entry (do first, no code)

Add to `docs/GOMBIT_BUILD_PLAN.md` §4 a `### REL — Release polish (post-v0.1)`
subsection with one row per REL-1..REL-8 below, then open one GitHub issue per
row titled `[REL-n] …`, milestone `post-v0.1`, labels from §6 (`docs`, `ci`,
`cli`). No new milestones or labels.

---

## 3. Decisions taken

| # | Decision | Rationale |
| --- | --- | --- |
| R1 | Release triggers on `push: tags: v*` **and** `workflow_dispatch` (patch/minor/major) — **not** auto-tag on every merge to main | Gombit is an importable Go module: every tag is a permanent public API version on pkg.go.dev. ACP is a CLI-only binary, so its auto-patch cadence is safe there and not here. |
| R2 | Tutorial builds a **tasks** CRUD app: `new` → model → `make resource` → migrate → OpenAPI + TS client → React page → cookie login → admin registration | Exactly the v0.1 "one CRUD loop" (working agreement §6). One resource keeps it correct under CI; it still touches every shipped subsystem. |
| R3 | License badge says **MIT**; Go badge says **1.25+** | Matches this repo's `LICENSE` and `go.mod`, not ACP's. |
| R4 | Tutorial app is committed under `examples/tutorial/` and built in CI | Working agreement §2: "stable features ship docs and appear in an example app." A tutorial that rots is worse than none. |
| R5 | README keeps a *short* doc index at the bottom; the 25-line link dump moves to `docs/README.md` | Keeps the front page about the product, not the filesystem. |

---

## 4. Workstreams

### REL-1 — README rewrite (badges, quickstart, positioning)

Target structure, modeled on `agentic-control-plane` but adapted to a framework:

```
# Gombit
<5 badges>
**One-line positioning.** Django-for-Go: <what it gives you>.
## Why Gombit
## What's in the box            (feature checklist, M0–M5 + ADMIN-1..3)
## Quick start                  (prereqs → install → new → dev → make resource)
## The CRUD loop in 60 seconds  (annotated command sequence + what each emits)
## Architecture                 (mermaid: Gin+Huma → OpenAPI → TS client → React; GORM → Atlas)
## Admin                        (screenshot placeholder + 6 lines + link)
## Compared with                (Gin/Echo/Fiber/Encore/Buffalo positioning table)
## Documentation                (short index → docs/README.md)
## Status and roadmap           (M0–M5 done, ADMIN-1..3 done, post-v0.1 batteries)
## Contributing / License
```

Badge block (verbatim, corrected for this repo):

```markdown
[![CI](https://github.com/LAA-Software-Engineering/gombit/actions/workflows/ci.yml/badge.svg)](https://github.com/LAA-Software-Engineering/gombit/actions/workflows/ci.yml)
[![Release](https://github.com/LAA-Software-Engineering/gombit/actions/workflows/release.yml/badge.svg)](https://github.com/LAA-Software-Engineering/gombit/actions/workflows/release.yml)
[![Go 1.25+](https://img.shields.io/badge/Go-1.25+-00ADD8?logo=go)](https://go.dev/dl/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Go Reference](https://pkg.go.dev/badge/github.com/LAA-Software-Engineering/gombit.svg)](https://pkg.go.dev/github.com/LAA-Software-Engineering/gombit)
```

Quickstart body (must be verified by actually running it, §6):

```bash
go install github.com/LAA-Software-Engineering/gombit/cmd/gombit@latest
gombit new demo --database sqlite --auth cookie --ui mui
cd demo && gombit db migrate && gombit createsuperuser && gombit dev
```

Notes:
- Every user-facing command becomes `gombit …`; `go run ./cmd/gombit …` survives
  only in CONTRIBUTING (contributor form).
- The Release badge is **red until REL-4 lands and a tag exists** — land REL-4
  before REL-1, or the front page ships broken on day one.

### REL-2 — CONTRIBUTING + community health files

- Rewrite `CONTRIBUTING.md`: fork/branch/PR flow, conventional commits, the
  `.github/pull_request_template.md` requirement, how to run the DB matrix
  locally (with the Docker one-liners for pg/mysql pulled from `ci.yml`), how to
  run `goldentest -update`, and a pointer to the `code-review` skill.
- Keep the existing "Working agreement" section verbatim — it is the definition
  of done and already correct.
- Add `CODE_OF_CONDUCT.md` (Contributor Covenant 2.1, contact
  `leonardo.aa88@gmail.com`).
- Add `SECURITY.md`: supported versions table (`0.1.x`), private reporting via
  GitHub Security Advisories, 90-day disclosure, explicit non-scope (generated
  app misconfiguration, `VITE_*` treated as public per working agreement §5).

### REL-3 — Issue templates

`.github/ISSUE_TEMPLATE/` as **YAML issue forms** (not markdown), because the
backlog has a strict `[ID]` title convention worth enforcing in the form:

| File | Title prefix | Key fields |
| --- | --- | --- |
| `bug_report.yml` | `[bug] ` | gombit version (`gombit version` output), Go version, OS, database (dropdown sqlite/postgres/mysql), auth mode, repro steps, expected vs actual, `gombit doctor` output |
| `feature_request.yml` | `[feature] ` | problem statement, proposed surface, milestone guess (dropdown incl. `post-v0.1`), Django equivalent (free text — the framework's north star), alternatives |
| `question.yml` | `[question] ` | what you're trying to do, what you've read (checkbox list of docs), version |
| `config.yml` | — | `blank_issues_enabled: false`; contact links → Discussions, `docs/`, and SECURITY.md for vulns |

Labels applied by the forms must already exist per build plan §6 — if `bug` /
`question` do not, flag it rather than creating labels (AGENTS.md).

### REL-4 — Release workflow + `gombit version`

Two parts, one PR.

**(a) Version wiring** — `cli/root.go` currently has no version at all:

- Add `cli/version.go` with `var Version = "dev"` (overridable via ldflags),
  plus `runtime/debug.ReadBuildInfo()` fallback so `go install @latest` builds
  self-report their module version instead of `dev`.
- Wire `rootCmd.Version` (gives `--version` free) **and** add a `gombit version`
  subcommand printing version, commit, build date, Go version, OS/arch.
- Test: `cli/version_test.go` asserts both `--version` and `version` render, and
  that the default is `dev` when neither ldflags nor build info are present.
- Document in `docs/cli.md` (add a row to the command table + a `## gombit
  version` section).

**(b) `.github/workflows/release.yml`** — ACP's structure, with R1 applied:

```yaml
on:
  push:
    tags: ['v*.*.*']
  workflow_dispatch:
    inputs:
      bump: {type: choice, options: [patch, minor, major], default: patch}
permissions:
  contents: write
concurrency:
  group: release-${{ github.repository }}
  cancel-in-progress: false
```

Job steps, in order:
1. `actions/checkout@v7` with `fetch-depth: 0` (matches this repo's CI, which is
   already on v7 — do not downgrade to ACP's v4).
2. **Resolve version** — on `workflow_dispatch`, reuse ACP's semver-bump shell
   verbatim (it is correct and handles the no-tag first-release case → `v0.1.0`)
   and push the computed tag; on `push: tags`, take `${GITHUB_REF_NAME}`.
3. `actions/setup-go@v7` with `go-version-file: go.mod`, `cache: true`
   (repo convention; ACP pins `1.25.x` inline — prefer the file).
4. `go mod download && go mod verify`.
5. `go test ./... -count=1 -timeout=15m` — gate the release on the same suite CI
   runs on `main`. (DB matrix jobs are integration-tagged and stay in `ci.yml`.)
6. **Build matrix** — `CGO_ENABLED=0`, `-trimpath`,
   `-ldflags "-s -w -X github.com/LAA-Software-Engineering/gombit/cli.Version=$VERSION"`
   for linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64 from
   `./cmd/gombit`.
   *Check during implementation:* the `database` package uses `glebarez/sqlite`
   (pure Go) or `mattn/go-sqlite3` (cgo). If the latter, `CGO_ENABLED=0` still
   builds the **CLI** as long as `cmd/gombit` doesn't link the driver — verify
   with a `CGO_ENABLED=0 go build ./cmd/gombit` smoke step before assuming.
7. `tar czf` / `zip` per platform → `gombit-${VERSION}-<os>-<arch>.{tar.gz,zip}`,
   then `sha256sum * | tee SHA256SUMS.txt`.
8. `softprops/action-gh-release@v2` with `generate_release_notes: true`.

Deliberately **not** included: GoReleaser (a 40-line workflow is enough and adds
no dependency), Homebrew tap, Docker image, signing/SLSA provenance. Note them
in the plan's follow-ups, not this PR.

### REL-5 — Installation guide (`docs/installation.md`)

- Prerequisites table: Go 1.25+, Node 22+ (frontend + admin UI), Atlas Community
  Edition (`curl -sSf https://atlasgo.sh | sh -s -- --community`, required by
  `gombit db makemigrations`/`migrate`), and per-driver notes.
- Three install paths: `go install …@latest`, release archive + checksum
  verification (`sha256sum -c SHA256SUMS.txt`), and from source.
- Per-OS notes incl. WSL2, `$GOBIN`/`PATH` troubleshooting.
- `gombit doctor` as the verification step — the repo already has it, so
  "verify your install" is one command.
- Uninstall.

### REL-6 — Comprehensive tutorial (`docs/tutorial.md`)

Per R2, build a tasks app. Chapters, each ending in a runnable checkpoint:

1. **Install and scaffold** — `gombit new tasks --database sqlite --auth cookie --ui mui`; tour the generated tree and explain the `internal/<feature>/` law (§3.3).
2. **Run it** — `gombit dev`; the service table, `/docs`, `/openapi.json`, the Vite proxy.
3. **Your first model** — `internal/task/model.go`, GORM tags, `gombit db makemigrations` → read the emitted SQL → `gombit db migrate` → `gombit db status`.
4. **The resource generator** — `gombit make resource Task`; what it writes, what it refuses to overwrite, `--dry-run`.
5. **The contract** — Huma DTOs, validation tags, the D10 envelope, `gombit openapi generate`, `gombit routes`.
6. **The typed client** — `gombit client check --write`, the generated TS surface, `gombit client check` as a drift gate in your own CI.
7. **The frontend** — wire the generated client into a React list/create page with React Hook Form.
8. **Auth** — cookie/session mode, CSRF double-submit, protecting a route, `gombit createsuperuser`.
9. **The admin** — `admin.Register` for `Task`, `GET /api/v1/admin/meta`, the SPA at `/admin/`, permissions/groups + the superuser bypass (ADMIN-3).
10. **Custom commands** — `gombit make command`.
11. **Ship it** — `gombit build --embed` → a single binary; config via env (`docs/config.md`).
12. **Where next** — links to the reference docs, ADRs, roadmap.

Rules for the prose: every command block copy-pasteable; every file shown in
full or with an explicit elision marker; each chapter states what "it worked"
looks like. Cross-link the reference doc for each subsystem rather than
duplicating it.

### REL-7 — End-to-end example app (`examples/tutorial/`)

- Commit the finished tutorial output (per R4) with its own README pointing back
  at `docs/tutorial.md`.
- Decide during implementation whether it lives as a full `gombit new` tree or as
  a trimmed app reusing the framework module — prefer the latter if the former
  fights the "generated apps are not committed in-tree" rule in AGENTS.md; if
  a full tree is needed, `.gitignore` its `node_modules` and vendor nothing.
- Add a CI step to `ci.yml`'s `build` job: `go build ./examples/...` so the
  tutorial cannot silently break.

### REL-8 — Docs index + release checklist

- `docs/README.md`: a table of the 18 topic docs grouped by
  Getting started / Runtime / Data / Contract / Frontend / Auth / Admin /
  Operations / Decisions, replacing the README link dump.
- `CHANGELOG.md`: Keep a Changelog format, `## [Unreleased]` + an `0.1.0` entry
  assembled from the M0–M5 + ADMIN-1..3 history.
- `docs/releasing.md`: the maintainer runbook — verify `main` green, update
  CHANGELOG, `git tag -a v0.1.0`, push, watch the workflow, verify pkg.go.dev
  and the archive checksums.

---

## 5. Order of work

```
REL-0 (backlog §4 + issues)
  └─ REL-4 (version + release workflow)   ← first, so the README badge is green
       ├─ REL-5 (installation)            ← needs the release artifact names
       ├─ REL-3 (issue templates)         ← independent, needs `gombit version` for the bug form
       └─ REL-6 (tutorial) ─ REL-7 (example app)
            └─ REL-1 (README) ─ REL-8 (docs index + CHANGELOG + releasing.md)
                 └─ REL-2 (CONTRIBUTING + CoC + SECURITY)
```

REL-1 lands late on purpose: it links to the tutorial, installation guide, and
release badge, so writing it first means writing it twice.

---

## 6. Verification

| Workstream | How it is verified |
| --- | --- |
| REL-1 | Run the quickstart verbatim in a clean dir on a clean `$GOPATH`; every badge URL returns 200; mermaid renders on github.com |
| REL-2 | Markdown link check; CoC/SECURITY appear in GitHub's community-standards checklist |
| REL-3 | Open a draft issue from each form in the UI; confirm labels and title prefixes apply |
| REL-4 | `cli/version_test.go`; dry-run the workflow on a throwaway `v0.0.1-rc1` tag in a fork; `sha256sum -c` each archive; run each binary's `version` on at least linux/amd64 + darwin/arm64 |
| REL-5 | Follow it on a clean WSL2 shell and a clean macOS shell; `gombit doctor` exits 0 |
| REL-6/7 | Execute every command in order into an empty dir; diff the result against `examples/tutorial/`; `go build ./examples/...` in CI |
| REL-8 | All `docs/*.md` appear exactly once in the index; no orphans |

Repo-wide gate before tagging v0.1.0: `go test ./...`, `golangci-lint run`,
`gombit client check` (no drift), and the full `ci.yml` matrix green on `main`.

---

## 7. Risks and open points

| Risk | Mitigation |
| --- | --- |
| `CGO_ENABLED=0` cross-compile fails if the sqlite driver is cgo-based | Smoke-build `./cmd/gombit` with cgo off in REL-4 step 1; if it fails, either the CLI drops the driver import or release only ships pure-Go targets and documents the cgo path |
| Release badge red until a tag exists | REL-4 ordered before REL-1; tag `v0.1.0` as part of REL-4's merge |
| Tutorial drifts as the framework changes | REL-7 commits the output and CI-builds it; consider a follow-up that extracts the tutorial's command sequence into a scripted smoke test |
| `go install …@latest` resolves nothing pre-tag | Same fix: REL-4 first. Until then the README must say "from source" |
| Doc-heavy PRs are hard to review | One workstream per PR, each linking its `[REL-n]` issue per the working agreement |
| Adding REL-* to §4 could be read as scope creep | REL-0 is explicitly a *packaging* section under `post-v0.1`, adds no runtime capability, and touches no M6 battery (working agreement §6) |

**Open point for the maintainer:** pkg.go.dev will index every tag permanently.
If `v0.1.0` is meant as a preview, tag `v0.1.0-rc.1` first — the release
workflow handles pre-release tags, but `generate_release_notes` will need
`prerelease: true` wired to a tag-pattern condition.

---

## 8. Definition of done

- [ ] §4 carries a REL subsection; one issue per REL-n, milestone `post-v0.1`
- [ ] Every PR follows `.github/pull_request_template.md` and links its issue
- [ ] `main` is green on the full matrix; `v0.1.0` is tagged and the release has
      5 archives + `SHA256SUMS.txt`
- [ ] `go install github.com/LAA-Software-Engineering/gombit/cmd/gombit@v0.1.0`
      works from a clean environment and `gombit version` prints `v0.1.0`
- [ ] A reader who has never seen the repo can go from the README to a running
      tasks app with an admin login using only committed docs
