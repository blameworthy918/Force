# security-controls — Design

**Status:** draft v2 · 2026-07-08 (revised after fresh-eyes security review)
**Owners:** maintainer team of 2–3 (blameworthy lead); solo until teammates onboard

A machine- and human-consumable library of security standards, requirements,
and controls — the central authoritative source that people and AI-assisted
development tools reference when deciding what to secure and how. Hosted on
company GitHub Enterprise. The repo is the foundation; consumption tools
come after it is complete and reviewed. Designed to grow: new frameworks,
new mapping shapes, and new maintainers must always arrive additively,
never as a redesign.

## 1. Goals

- Central authoritative source for security requirements and controls at a
  Fortune 100 healthcare employer; production use is the target, not a demo.
  Consumers are both humans (engineers, GRC, auditors) and machines
  (AI-assisted development tooling).
- Content: NIST CSF 2.0, NIST SP 800-53r5, HIPAA Security Rule (via
  SP 800-66r2 mapping), Microsoft Cloud Security Benchmark v2; CIS Controls
  8.1 (company-licensed) and company standards from RSA Archer as later
  phases.
- Small-file, context-efficient structure: an AI agent loads only the
  controls it needs, never a whole framework.
- Shared maintenance: agents do discrete tasks (discover, create, update,
  review); human review only where a change can alter meaning. **Merge
  decisions are made by deterministic code, never by an LLM** (§5).
- Resilient to change: frameworks, mappings, maintainers, and consumption
  patterns not known today must onboard via the existing recipes (§4)
  without restructuring.

### What v1 is (and is not)

v1 delivers the machine-consumable substrate: faithful canonical catalogs,
typed cross-framework mappings, and the unattended maintenance loop. The
"what should *we* secure" answer — org applicability, parameter values,
company standards — lives in `profiles/` and `standards/` and fills in
during later phases. v1 pilots the profile layer thinly (§10 P1c) so its
seams and review routing are proven, but v1 is judged as substrate, not by
guidance value it doesn't yet claim.

### Definition of done

1. **Full public content coverage** — CSF 2.0, 800-53r5, HIPAA mapping, and
   MCSB v2 ingested, schema-valid, cross-mapped, and reviewed (pipeline
   verified + per-framework spot-check).
2. **Unattended agent operation** — the discover → author → gate loop runs
   unattended for ≥ 4 consecutive weeks, correctly processing at least one
   real upstream change: canonical-layer syncs merge on green CI without
   human touch; anything else escalates to the human correctly. If no real
   upstream change lands in the window, a staged synthetic change (modified
   vendored snapshot on a fixture branch) counts as evidence for the loop
   mechanics; only the "real change" clause waits for upstream.

CIS 8.1 and Archer standards are **required later phases** but do not gate
done — CIS waits on license-terms verification, Archer on export access.

### Non-goals (v1)

- No consumption tooling (MCP server, skills, hooks, enforcement) until the
  repo is complete and reviewed — foundation first.
- No OSCAL authoring; OSCAL is an import (and potential export) format only.
- No heavyweight governance: a 2–3 person team runs on CODEOWNERS-per-layer
  plus one approving review (never the PR author). No approval matrices or
  multi-team ceremony yet; the layer seams are the growth path.
- No GRC-platform replacement; Archer stays the audit system of record, this
  repo syncs *from* it (one direction, later phase).

## 2. Format decision

**Working format: simplified YAML, one file per control.** Full OSCAL is the
interchange format at the boundaries, never the authored/consumed format.

Rationale (research, 2026-07-08): every surviving compliance-as-code project
converged on small per-control files and hid or dropped OSCAL (trestle hides
it behind markdown; Lula abandoned it as "too complex"); an OSCAL 800-53
control is 5–20× the tokens of its YAML equivalent — fatal for the
context-efficiency requirement. Provenance fields preserve native/OSCAL IDs
so OSCAL can be regenerated if an auditor or GRC tool demands it.

### Schema kernel (extensibility contract)

The JSON Schema defines a small **required kernel** every content file
carries — `id`, `framework`, `title`, `status`, `provenance` — plus
optional typed fields that frameworks use as their shape demands
(`statement`/`parameters`/`enhancements` are 800-53 shapes; CSF
subcategories and MCSB entries carry their own). **Schema changes are
additive-only**: the kernel never changes, and onboarding a framework may
add optional fields but never touches existing ones.

### Control file schema (sketch)

```yaml
# catalogs/nist-sp800-53/r5/ac/ac-2.yaml
id: ac-2
framework: nist-sp800-53
family: ac
title: Account Management
status: active            # active | withdrawn
# superseded_by: [ac-6]   # withdrawn controls: successor ids from the
#                         #   upstream withdrawal notice (free in OSCAL)
statement: |
  a. Define and document the types of accounts allowed ...
guidance: |
  Examples of system account types include individual, shared, group ...
parameters:
  - id: ac-02_odp.01
    label: prerequisites and criteria
enhancements:
  - id: ac-2.1
    title: Automated System Account Management
    statement: |
      Support the management of system accounts using ...
provenance:
  source: usnistgov/oscal-content
  source_version: "5.1.1+u5"
  oscal_uuid: <uuid>
  imported: "2026-07-08"   # from sources.lock, never the run date (§4)
  converter: tools/ingest/oscal_catalog.py
```

Enhancements live inline in the parent control's file (keeps 800-53 to ~320
files instead of ~1,200). **Catalog files are exempt from any small-file
line cap**: heavy controls (SC-7 has ~29 enhancements, SI-4 ~25) legitimately
run past 1,000 lines, which is still 50× cheaper than loading a framework.
`index.yaml` lists each control's enhancement ids and titles under its
parent, so an agent knows what a file contains before loading it. Withdrawn
controls stay present with `status: withdrawn` and `superseded_by` — mappings
and baselines may still reference them, and integrity warnings can suggest
repairs. Every framework directory carries an `index.yaml` plus a
`version.yaml` recording the exact upstream version (paths and
cross-references pin only the major revision — see §3).

### Mapping schema (sketch)

Typed relationship tuples per NIST IR 8477 — the vocabulary OLIR, CPRT, SCF,
and OSCAL 1.2's mapping model all share. One file per framework pair
(split per family/function if a file grows past ~300 lines).

**Mappings are shape-agnostic**: endpoints are any two catalogs in the repo,
including regulatory texts vendored as thin catalogs (citation ids + text).
The recipe must never assume CSF↔800-53 shapes — HIPAA Security Rule ↔
800-53 (P1c) is the first proof, and Archer standards will stress it again.

```yaml
# mappings/nist-csf-2.0__nist-sp800-53-r5/pr.yaml
source_framework: nist-csf-2.0
target_framework: nist-sp800-53@r5    # major revision only — patch bumps
                                      #   must not touch mapping files
provenance: {source: NIST CPRT crosswalk, imported: "2026-07-08"}
mappings:
  - source: pr.aa-01
    target: ac-2
    relationship: intersects-with
    rationale: optional one-liner     # optional; per-tuple provenance also
                                      #   allowed — auditors ask "why does
                                      #   this map" (matters for Archer)
```

## 3. Repository layout

Layers are separate directories because **review routing is computed from
which layer a diff touches** (see §5–6).

```
security-controls/
├── DESIGN.md
├── README.md
├── LICENSES/                    # per-source license + attribution files
├── schemas/                     # JSON Schema: required kernel + optional
│                                #   per-framework fields; additive-only
├── sources/                     # vendored upstream snapshots — the ONLY
│   │                            #   input converters read; fetched by
│   │                            #   discover, never hand-edited
├── sources.lock                 # per source: URL, upstream version, sha256,
│                                #   trust tier (§4)
├── catalogs/                    # LAYER: imported canonical text — generated,
│   │                            #   never hand-edited (regenerate-and-diff CI)
│   ├── nist-sp800-53/r5/        #   path pins MAJOR revision only; exact
│   │   ├── baselines/           #   version in version.yaml + provenance.
│   │   └── ...                  #   53B baselines = generated control-ID lists
│   ├── nist-csf/2.0/
│   ├── hipaa-security-rule/2013/#   thin catalog: safeguard citations
│   │                            #   (45 CFR §164.xxx) as mapping endpoints
│   ├── mcsb/v2/
│   └── cis/8.1/                 #   later, after license verification
├── mappings/                    # LAYER: cross-framework tuples (IR 8477)
├── profiles/                    # LAYER: org tailoring — applicability,
│   │                            #   parameter values (thin pilot in P1c)
├── standards/                   # LAYER: company standards ← Archer (later)
├── tools/
│   ├── ingest/                  # deterministic converters, one per source;
│   │   │                        #   all output via the shared canonical
│   │   │                        #   YAML serializer (§4)
│   │   ├── oscal_catalog.py     #   oscal-content JSON → YAML (800-53r5)
│   │   ├── cprt.py              #   CPRT JSON → YAML (CSF 2.0, crosswalk)
│   │   └── mcsb_docs.py         #   MCSB v2 artifact/docs → YAML
│   ├── gate/                    # deterministic PR classifier (§5) —
│   │                            #   paths + status diffs + lock tiers
│   └── validate/                # schema, mapping-integrity, regen-diff,
│                                #   index, golden fixtures
├── .claude/
│   └── agents/                  # discover.md, author.md, summarizer.md
└── .github/
    ├── CODEOWNERS               # per-layer human review routing
    └── workflows/               # validate.yml, gate.yml, discover.yml,
                                 #   staleness.yml (scheduled)
```

Versioning is two-axis: directory paths and all cross-references (mapping
headers, baseline lists) pin the **major revision only** (`r5`, `2.0`);
the exact upstream version (`5.1.1+u5`) lives in each framework's
`version.yaml`, `sources.lock`, and file provenance. This keeps a routine
patch bump (5.1.1 → 5.1.2) confined to `sources/` + `catalogs/` +
`sources.lock` — auto-mergeable per §6 — instead of renaming paths or
touching every mapping file. Repo-level semver tags version our own
releases. Upstream bumps arrive as agent-authored PRs.

## 4. Content pipeline (deterministic, not LLM transcription)

All canonical content is produced by converter scripts — re-runnable,
reviewable, faithful. Converters read **only** from vendored snapshots in
`sources/` (pinned by `sources.lock`); neither converters nor CI ever fetch
upstream — only discover does, by committing a new snapshot + lock update.
Agents orchestrate converters and summarize diffs; they do not transcribe
control text.

| Source | Input | License | Notes |
|---|---|---|---|
| 800-53r5 (+53B baselines) | usnistgov/oscal-content OSCAL JSON | Public domain | Cleanest; build first |
| CSF 2.0 | NIST CPRT JSON export | Public domain | No official OSCAL; CPRT is authoritative |
| CSF↔800-53 crosswalk | NIST CPRT/OLIR | Public domain | First mapping ingest |
| HIPAA Security Rule ↔ 800-53 | NIST SP 800-66r2 / OLIR | Public domain | HIPAA enters as a thin citation catalog + mapping (P1c); first non-CSF↔800-53 mapping shape |
| MCSB v2 | Official artifact if published (v1 shipped Excel/JSON via MicrosoftDocs/SecurityBenchmarks); else MS Learn docs | CC BY 4.0 | Prefer the artifact — it may earn `release-artifact` tier; if scraping, hash *extracted content*, never raw HTML (dynamic nav/timestamps) |
| CIS 8.1 | CIS WorkBench OSCAL | CC BY-NC-ND (licensed) | After license-terms verification; keep verbatim source + attribution beside transformed YAML |
| Company standards | Archer REST/report export | Company-owned | After export access provisioned |

### Trust tiers

Every `sources.lock` entry carries `trust: release-artifact | scraped`.
`release-artifact` = versioned, tagged upstream releases (NIST oscal-content,
CPRT exports, an official MCSB artifact). `scraped` = mutable web content
with no release discipline. The tier drives §6: **text changes to existing
controls from a `scraped` source never auto-merge** — a vandalized or
compromised page must not flow into canon without human eyes.

### Determinism rules (all converters)

Regenerate-and-diff CI (§7) only works if converters are bit-identical
across machines and days:

- Provenance dates (`imported`) come from `sources.lock` metadata — never
  the run date.
- All YAML output goes through one shared canonical serializer (key order,
  wrapping, unicode policy); Python and dependency versions are pinned by a
  lockfile and CI uses exactly that environment.
- `oscal_uuid` churns between upstream releases; semantic diff summaries
  must ignore provenance-only changes or every patch bump looks huge.
- Each converter has a **golden fixture**: a miniature vendored source slice
  plus expected YAML output, committed and tested on every PR touching
  `tools/` — instant objective signal during converter work.

### Adding a framework (extensibility requirement)

Onboarding a new framework — including standards unknown today — must be
**additive and friction-free**, never a redesign. The recipe, identical for
every framework:

1. Vendor the upstream artifact into `sources/` + add a `sources.lock`
   entry (with trust tier).
2. Write one converter module in `tools/ingest/` (+ golden fixture).
3. Generated output lands in a new `catalogs/<framework>/<rev>/` directory
   (controls + `index.yaml` + `version.yaml`).
4. Optional mapping files in `mappings/`.

Nothing else changes: the schema kernel is framework-agnostic (additive
optional fields only), and validators/CI discover frameworks dynamically
from the directory tree — no hardcoded framework lists anywhere. P1b (CSF)
proves the catalog recipe, P1c (HIPAA) proves the mapping recipe on a
non-control-shaped source, and any deviation either forces is a design bug
to fix then.

## 5. Agent maintenance model

Discrete tasks, minimal ceremony — and a hard rule: **no LLM in the merge
path**. Upstream text is attacker-influenceable (especially scraped
sources); a steerable classifier in front of auto-merge is a
prompt-injection hole. Every condition in §6 is mechanically decidable, so
the merge decision is code:

- **discover** (scheduled): polls upstream — oscal-content releases, CPRT
  endpoints, MCSB artifact/page content hashes. On change: commits the new
  snapshot to `sources/` + updates `sources.lock` (the snapshot diff *is*
  the upstream change, reviewable), opens an issue with a summary, and
  kicks author. Touches nothing outside `sources/` + the lock file (CI
  enforces the path scope).
- **author**: runs the relevant converter, regenerates YAML, opens a PR.
  Never merges.
- **merge gate** (deterministic script in `tools/gate/`, required status
  check — not an agent): classifies every PR from changed paths, control
  file adds/removals, `status:` field diffs, and `sources.lock` trust
  tiers, then applies the §6 table — auto-merge or escalate to human
  review. No model, no judgment, no attack surface.
- **summarizer** (LLM, advisory only): on escalated PRs, writes the
  human-facing semantic summary (controls added / withdrawn / text-changed /
  params-changed, provenance-only noise filtered). Has no merge authority;
  its output is a PR comment, never an input to the gate.

Separation of duties on GitHub Enterprise: two machine identities (GitHub
Apps) — the author identity opens PRs, the gate identity merges; branch
protection/rulesets enforce PR-only main, required status checks, and no
self-approval. CODEOWNERS routes layer diffs to human maintainers; human
review = one approving review from a maintainer other than the PR author.

**Runtime, phased:** prove the loop against a fixture branch (staged
synthetic upstream change), then deploy on GHE Actions with an org API key
and the GitHub App identities. Pin stable model IDs — never a preview
model; a 4-week unattended run must not die with a preview sunset. In
`.claude/agents/*.md` frontmatter: `claude-sonnet-5` for discover, author,
and summarizer (orchestration and summaries — the judgment-heavy merge
decision no longer exists as an LLM task). The 4-week unattended clock for
"done" starts on the Actions deployment.

## 6. Human-in-the-loop policy

Route by layer and trust tier, not by size — no paperwork for changes that
can't change meaning. **Every row is computed by the `tools/gate/`
classifier; an LLM never makes this call.**

| Change | Handling |
|---|---|
| `sources/` snapshot + lock update (discover) | Auto-merge on green CI |
| Canonical sync from `release-artifact` source, no control add/withdraw | Auto-merge on green CI |
| Regeneration/formatting/provenance-only diffs | Auto-merge on green CI |
| Text change to existing controls from a `scraped` source | Human review |
| Upstream adds/withdraws controls | Human review |
| Any `mappings/` change | Human review |
| Any `profiles/` or `standards/` change | Human review |
| Converter/schema/gate/CI code change | Human review |

Human review = one approving review from a maintainer other than the PR
author, routed by CODEOWNERS per layer.

## 7. Validation CI (hard gates, every PR)

Validators are framework-agnostic: they discover catalogs from the
directory tree, never from a hardcoded list.

1. JSON Schema validation of every YAML file (kernel + framework fields).
2. Mapping integrity — every mapping and baseline endpoint resolves against
   the referenced catalog. Enhancement ids (`ac-2.1`) have no file of their
   own: the resolver maps them to the parent control file. References to
   withdrawn controls warn and suggest `superseded_by` repairs.
3. Regenerate-and-diff — converters re-run against the vendored `sources/`
   snapshots must reproduce `catalogs/` exactly (catches hand-edits to
   canon; fully deterministic per §4 — CI never fetches upstream).
4. Snapshot integrity — `sources/` contents match the sha256s in
   `sources.lock`.
5. Index consistency — `index.yaml` matches the control files and inline
   enhancements present.
6. License/attribution file present for every source directory.
7. Golden-fixture tests — every converter reproduces its committed fixture
   output (runs on PRs touching `tools/`).
8. Staleness (warn-tier): last-discover-check age per source — runs on a
   **scheduled workflow**, not just PR CI, so a dead discover loop surfaces
   even when no PRs are flowing.

## 8. Review gate before tool access

Before any consumption tool may point at this repo ("fully created and
reviewed"):

- Human review of: all converter code, schemas, the gate classifier, CI
  workflows, agent definitions.
- Per-framework spot-check: random sample of control files diffed against
  the upstream source (sample size ~15/framework), plus targeted checks of
  known-tricky spots (parameter insertion in 800-53, withdrawn controls,
  HIPAA citation granularity, MCSB mapping blocks). Executed by the
  **content-verifier agent** (§11), which produces a discrepancy report;
  a maintainer reviews the report and the flagged files, and the report is
  attached to `REVIEW.md`.
- Sign-off recorded in the repo (`REVIEW.md` with date, commit, sample
  list), by a maintainer other than the primary author of the converters
  once the team exists.

## 9. Licensing & hosting posture

- Repo: **company GitHub Enterprise from the start**, private to the org.
  No personal-infrastructure phase — provenance stays clean for internal
  security review, and GHE provides the branch protection, rulesets,
  CODEOWNERS, and GitHub App identities that §5 depends on.
- Build order still leads with the simplest licensing surface:
  public-domain (NIST, HIPAA/800-66r2) and CC BY 4.0 (MCSB) content while
  the pipeline matures.
- CIS 8.1 lands only after its license terms are verified and recorded in
  `LICENSES/` (CC BY-NC-ND, company-licensed); its directory carries the
  CIS terms and a verbatim upstream copy alongside transformed YAML.
- Archer content lands after export access is provisioned; Archer remains
  the audit system of record.
- `LICENSES/` carries attribution per source — keep it airtight; it is the
  provenance story for any internal audit of this repo.

## 10. Phases

- **P0 — scaffold**: layout, schemas (kernel + extensions), validators,
  gate classifier, canonical serializer, golden-fixture harness, CI green
  on empty content; GHE branch protection + CODEOWNERS in place.
- **P1a — 800-53r5**: converter (+53B baselines), content-verifier agent,
  review gate per §8.
- **P1b — CSF 2.0**: CPRT converter + CSF↔800-53 crosswalk; first proof of
  the add-a-framework recipe.
- **P1c — HIPAA + profile pilot**: thin HIPAA Security Rule citation
  catalog + 800-66r2 mapping (first non-CSF↔800-53 mapping shape); thin
  org profile pilot (one family, a few ODP values) to exercise the
  `profiles/` schema and its review routing before anything depends on
  them.
- **P2 — MCSB v2**: official artifact if it exists, else docs converter
  with content-extraction hashing (`scraped` tier); mapping blocks feed
  `mappings/`; review gate.
- **P3 — agent loop**: discover/author/gate/summarizer on a fixture branch,
  then GHE Actions with GitHub App identities; 4-week unattended clock →
  **done**.
- **P4 (post-done)**: CIS 8.1 (license verified), Archer standards, then
  the consumption tools (MCP server, skills, hooks/enforcement) as
  follow-on projects with this repo as their foundation.

## 11. Build workflow & model plan

**Cadence: one phase per session**, driven by the main session (no
agent-per-phase). Each phase ends at an objective gate — CI green on GHE
plus the §8 spot-check for content phases — which is also the session
boundary. Handoff notes (`.remember/remember.md`) + git history carry
context between sessions; a human sits at every phase boundary.

Sessions: P0 scaffold · P1a 800-53r5 · P1b CSF 2.0 + crosswalk ·
P1c HIPAA + profile pilot · P2 MCSB v2 · P3 agent loop.

Per-session rhythm:
1. Catch up: `.remember/remember.md` + the relevant section here. This
   document is the plan — don't re-plan per session (exception: P3 gets a
   plan-mode pass first; it's new design, not execution of this doc).
2. Break the phase into tracked tasks.
3. Execute in the main session; delegate verbose side-work to subagents
   (validation runs → test-runner; upstream format spelunking → researcher)
   so raw payloads stay out of main context.
4. Verify on the objective signal only: validators pass, CI green on the
   pushed branch, regen-diff clean, golden fixtures green.
5. End session with handoff note updated.

**Models:**

- *Build sessions (interactive):* Fable 5 while preview access lasts —
  highest quality at the judgment points (schema decisions, gate-classifier
  edge cases, spot-check assistance). Not load-bearing: correctness comes
  from deterministic converters + CI hard gates, so a Sonnet 5 session
  reaches the same end state with a few more iterations on parser edge
  cases.
- *Unattended production loop (P3):* pin stable model IDs — never a preview
  model. `claude-sonnet-5` for discover/author/summarizer; the merge gate
  is a script and uses no model at all (the judgment-heavy step was
  removed from the merge path by design, §5).

**Build-time agents & skills** (distinct from the §5 product agents, which
are P3 deliverables):

- No agent-per-phase; the main session drives each phase and delegates
  verbose side-work to stock agents (test-runner for validation runs,
  researcher for upstream format spelunking).
- One custom agent: **content-verifier** — samples converted control files,
  fetches the upstream source, compares semantics field by field, emits the
  §8 discrepancy report. An agent rather than a skill because its bulky
  inputs (raw OSCAL/CPRT payloads) must stay out of main context. Created
  in P1a when the first real spot-check makes the procedure concrete;
  reused at every content gate (P1b, P1c, P2, later CIS).
- Skills: none hand-authored up front. The project verify skill
  auto-bootstraps via `/verify` in P0/P1 once `tools/validate/` exists
  (verify = schema validation + mapping integrity + regen-diff + index
  consistency + golden fixtures). The consumer-facing `consult-controls`
  skill (scan `index.yaml` first, load only needed control files) is a
  **P4 product deliverable** alongside the MCP server — building it earlier
  would violate the §8 gate. A phase-start skill encoding the session
  rhythm is deliberately deferred until re-explaining the rhythm becomes a
  real friction signal.
