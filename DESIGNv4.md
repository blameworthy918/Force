# security-controls — Design

**Status:** draft v4 · 2026-07-09 — rewritten around the requirements-as-product
goal. Supersedes v3's substrate-first framing; carries forward every v2/v3
decision that survives the inversion (deterministic pipeline, no-LLM merge
path, trust tiers, additive recipes, GHE governance).
**Owners:** maintainer team of 2–3 (blameworthy lead); solo until teammates
onboard.

## 1. Goal

> **Every security decision at the company — by an engineer, an AI
> assistant, or an auditor — is answered from one authoritative, versioned
> set of official company requirements: clear, prescriptive,
> platform-neutral, parameterized, and traceably mapped to the industry
> frameworks they satisfy.**

Short form: **official company security requirements as code — followed by
builders, provable to auditors.**

The product is the `requirements/` layer. Everything else is a means,
judged by how directly it serves the product:

| Element | Role |
|---|---|
| `requirements/` — parameterized official requirements | **The product** |
| `mappings/` + inline `satisfies` tuples | The proof mechanism — audits, crosswalks, CSF roll-ups |
| `catalogs/` — framework content | Substrate: thick enough for endpoints to resolve and requirements to be derived, no thicker |
| `profiles/` — parameter values | The single place values change (values originate in Archer standards, §3) |
| Converters, gate, agent loop | Maintenance cost control — the product stays current without headcount |
| Archer export | Input feed: Policy & Standards citations, parameter backing, cross-references |
| Consumption tools (MCP, skills, hooks) | Distribution — post-product |

### Authority model (decided 2026-07-09)

| Artifact | Authoritative home | Owner |
|---|---|---|
| Policy & Standards | **Archer** | policy/standards owners (GRC side) |
| Requirements — the *what* | **this repo** | Product Security (implementation owners) |
| Technical controls — the *how*, measured | security tooling (Prisma, Wiz, …), post-done | Product Security |

The chain: a Policy mandates, a Standard sets values, a Requirement states
what must be done (parameterized from Standards), technical controls
measure whether it is done. This repo is authoritative for exactly one
tier — requirements — citing up (parameters cite standards) and mapping
down (technical controls, post-done). GRC is a **consumer** of the
rendered reports for now; the reserved growth seam, if the teams decide
to collaborate, is CODEOWNERS review for GRC on the compliance surface
(`mappings/`, report semantics).

**Terminology rule:** the word *control* is always qualified in this
repo — **framework control** (catalog entry: SC-8) vs **technical
control** (measurable pass/fail check in tooling). An unqualified
"control" in any file is a doc bug.

### Goals

- **Official requirements people can follow.** Small domain files
  (MCSB-style sections and numbering: `ns-1`), each with a security
  principle, a prescriptive platform-neutral statement, and per-platform
  implementation guidance (Azure / AWS / GCP / on-prem). Values are never
  hardcoded: statements reference parameters resolved from the org profile,
  so a policy change (TLS 1.3 → 1.4) is a one-line, human-reviewed diff.
- **Compliance as a query, not a project.** Every requirement carries typed
  `satisfies` mappings to framework controls. CI generates the proof:
  coverage/gap reports (framework controls with no covering requirement),
  auditor crosswalks ("every requirement satisfying AC-2, with resolved
  parameter values"), and CSF 2.0 roll-ups — % views per category and
  function for leadership. Two numbers, never conflated: **coverage %**
  (subcategory has ≥1 mapped requirement — computable from the repo, day
  one) and **implementation %** (requirement verifiably followed — needs
  assessment data from Archer or scanning tools; a later input that slots
  into the same roll-up).
- **Context-efficient for machines.** One small YAML file per unit; agents
  load only what a task needs, discovered via `index.yaml`, never a whole
  framework or domain.
- **Maintained by agents, governed by code.** Agents do discrete tasks
  (discover, regenerate, flag impact, draft); **merge decisions are made by
  deterministic code, never by an LLM** (§7). Official requirement content
  is always human-approved.
- **Resilient to change.** New frameworks, domains, mapping shapes, and
  maintainers onboard via the existing additive recipes (§5–6) without
  restructuring. Schema changes are additive-only.

### Implementation measurement model (recorded now, built post-done)

How implementation % will eventually compute — recorded so v1 schemas
don't foreclose it, and deliberately **not built in v1**: we are
pre-author; nothing here gates P0–P5.

- A requirement's `satisfies` tuples point *up* at framework controls —
  compliance traceability. **Technical controls** are the distinct second
  mapping direction, pointing *down*: measurable checks (Prisma/Wiz
  policies, config baselines) that inherit the requirement's parameters
  and return pass/fail (`tls_min_version = TLS 1.3 → pass`).
- A requirement is **implemented** when every technical control mapped to
  it passes; implemented requirements roll up through `satisfies` and the
  CSF hierarchy to subcategory / category / function implementation %.
- Schema seam reserved: a future `measured_by:` list on requirements (or
  a checks layer) — additive per the kernel contract. The likely first
  feed is exporting rendered requirements as Prisma/Wiz custom frameworks
  (§12 P6).

### Definition of done (v1)

1. **Official requirements live** — the v1 domain scope (chosen in P1 with
   security leadership and recorded in `REVIEW.md`) fully authored:
   parameterized, platform-neutral, human-approved, rendered.
2. **Proof generated, not assembled** — CI produces the coverage/gap
   report, CSF category/function roll-up, and auditor crosswalks from
   `satisfies` tuples on every merge to main.
3. **Substrate as pulled** — every `satisfies` endpoint resolves against a
   catalog (thin citation catalogs suffice; full text only where
   derivation or consumers demanded it, per §12 phases).
4. **Maintenance loop deployed** — discover + impact-flagging running on
   GHE Actions, having correctly processed at least one upstream change.
   A staged synthetic change (modified vendored snapshot on a fixture
   branch) counts as evidence for loop mechanics; a real upstream change
   closes the clause when one lands. *(The v3 "4 consecutive unattended
   weeks" gate is dropped: the product is human-approved authored content,
   not a mirror of churning upstream — a calendar gate no longer buys
   assurance proportional to its delay.)*

Archer intake and CIS 8.1 remain **required later phases** but do not gate
done — Archer waits on in-company checks (§14), CIS on license
verification. MCSB is out entirely (decided 2026-07-09): our requirements
are generated, not reused from Microsoft's — only its file format lives
on, absorbed into the requirement schema (§3).

### Non-goals (v1)

- No consumption tooling (MCP server, skills, enforcement hooks) until the
  layers it would read have passed their review gates and the requirement
  set covers enough domains to be worth distributing (post-P3 judgment
  call, recorded in `REVIEW.md`).
- No full-framework transcription for its own sake: catalogs earn thickness
  from demand (§12), not completeness pride.
- No OSCAL authoring; OSCAL is an import (and potential export) format only.
- No heavyweight governance: 2–3 people on CODEOWNERS-per-layer plus one
  approving review (never the PR author). Layer seams are the growth path.
- No GRC-platform replacement and no measurement in v1: Archer remains
  authoritative for Policy & Standards (§1 authority model);
  technical-control measurement is post-done. This repo may someday push
  rendered requirements *to* Archer — a post-done question.
- No exception/waiver workflow in v1 — noted as the first likely `profiles/`
  extension when real consumers hit it.

## 2. Content classes

Two classes with opposite rules; every directory belongs to exactly one:

- **Generated canon** — `catalogs/`, `rendered/`, imported crosswalks in
  `mappings/`: produced by deterministic converters/renderers, **never
  hand-edited** (regenerate-and-diff CI fails hand edits). To change it,
  fix the tool and regenerate.
- **Authored, governed** — `requirements/`, `profiles/`, authored mapping
  files: written by humans or AI-drafted in interactive sessions, **always
  merged through human approval**. AI may draft; it never merges (§8).

## 3. Format & schemas

**Working format: simplified YAML, one file per unit.** Full OSCAL is an
interchange format at the boundaries, never authored or consumed.
Rationale (research 2026-07-08): surviving compliance-as-code projects
converged on small per-control files and hid or dropped OSCAL; an OSCAL
800-53 control is 5–20× the tokens of its YAML equivalent. Provenance
fields preserve native/OSCAL IDs so OSCAL can be regenerated on demand.

### Schema kernel (extensibility contract)

Every content file carries the required kernel — `id`, `framework`,
`title`, `status`, `provenance` — plus optional typed fields per framework
shape. **Additive-only**: the kernel never changes; onboarding a framework
may add optional fields but never touches existing ones.

**Company requirements are a first-class framework** (slug `corp`,
placeholder for the company slug). They live in `requirements/` but are, in
the data model, just another catalog: they get `index.yaml`, versioning,
and can serve as mapping endpoints like any framework. No special cases —
every existing mechanism (mapping integrity, indexes, roll-ups) applies
uniformly, and future shapes (a subsidiary's requirement set, a
business-unit overlay) arrive as additional catalogs, not redesigns.

### Requirement schema (the product)

```yaml
# requirements/network-security/ns-1.yaml
id: ns-1
framework: corp
domain: network-security
title: Encrypt data in transit
status: official            # draft | official | retired
# superseded_by: [ns-4]     # retired requirements name their successor
principle: |
  Protect data traversing networks from interception and tampering.
requirement: |
  All data in transit must be encrypted using {{tls_min_version}} or
  higher. Unencrypted protocols ({{prohibited_protocols}}) are prohibited.
satisfies:
  - target: nist-sp800-53@r5/sc-8
    relationship: subset-of
  - target: nist-csf-2.0/pr.ds-02
    relationship: intersects-with
implementation:
  azure: |
    Enforce TLS {{tls_min_version}} minimum on App Service, Storage, ...
  aws: ...
  gcp: ...
  onprem: ...
provenance:
  origin: authored           # authored | derived-from-archer
  approved: "2026-08-01"     # date of the approving human review
```

Rules: the `requirement` statement never names a platform (the company runs
Azure, AWS, GCP, and on-prem — platform "how" lives only in
`implementation:`); values appear only as `{{parameter}}` references;
`satisfies` targets are ordinary IR 8477 tuples validated by mapping
integrity (§9); ids are stable and never reused; retired requirements stay
present with `status: retired` + `superseded_by`.

### Profile schema (parameters)

```yaml
# profiles/corp/parameters.yaml
tls_min_version:
  value: "TLS 1.3"
  standard: corp-std/crypto-001   # backing Archer standard — free-text
                                  #   citation until P4 vendors the
                                  #   standards citation catalog
  last_reviewed: "2026-07-09"
prohibited_protocols:
  value: "SSL, TLS 1.0/1.1, unencrypted HTTP/FTP/Telnet"
  standard: corp-std/crypto-001
```

**Values originate in Archer standards** (decided 2026-07-09, §1 authority
model): the standard changes first; the profile mirrors it with a
citation. A needed value with no backing standard is a *standards gap* to
raise with the standards owners — never a value invented here. **Known
gap until P4:** nothing detects an Archer standard changing underneath a
mirrored value — drift is watched manually (periodic check by the
maintainers, plus asking standards owners to notify) until the discover
loop covers Archer at P4. One place
values change; the renderer (§6) resolves references into the
human-readable views. 800-53 ODP tailoring lands in the same profile layer
when full-text catalogs arrive.

### Catalog schema (substrate — thin and full)

Catalogs enter **thin-first**: id, title, family/function lineage — enough
for `satisfies` endpoints to resolve, indexes to be scannable, and roll-ups
to compute. Full text is a per-framework upgrade of the same converter when
demand pulls it (requirement drafting at scale needs control text in-repo;
consumers ask "what does SC-8 actually say"). Thin output is a strict
subset of full output — same kernel, same file paths, no migration.

```yaml
# catalogs/nist-sp800-53/r5/sc/sc-8.yaml (thin)
id: sc-8
framework: nist-sp800-53
family: sc
title: Transmission Confidentiality and Integrity
status: active              # active | withdrawn (+ superseded_by)
enhancements:               # ids+titles even in thin form — mapping
  - {id: sc-8.1, title: Cryptographic Protection}   #   endpoints resolve
provenance:
  source: usnistgov/oscal-content
  source_version: "5.1.1+u5"
  imported: "2026-07-09"    # from sources.lock, never the run date (§5)
  converter: tools/ingest/oscal_catalog.py
```

Full form adds `statement`, `guidance`, `parameters`, enhancement
statements — inline in the parent file (keeps 800-53 at ~320 files, not
~1,200). **Catalog files are exempt from small-file caps**: SC-7 (~29
enhancements) legitimately runs past 1,000 lines — still 50× cheaper than
loading a framework. Every framework directory carries `index.yaml`
(ids, titles, enhancement ids) and `version.yaml` (exact upstream version).
Withdrawn controls stay present — mappings may still reference them, and
integrity checks suggest `superseded_by` repairs.

### Mapping schema (imported crosswalks)

Typed tuples per NIST IR 8477 (shared by OLIR, CPRT, SCF, OSCAL 1.2). One
file per framework pair, split per family/function past ~300 lines.
Endpoints are any two catalogs in the repo — including `corp` and thin
regulatory catalogs (HIPAA citations). Requirements' `satisfies` blocks are
the authored siblings of these files, validated identically.

```yaml
# mappings/nist-csf-2.0__nist-sp800-53-r5/pr.yaml
source_framework: nist-csf-2.0
target_framework: nist-sp800-53@r5    # major revision only (§4)
provenance: {source: NIST CPRT crosswalk, imported: "2026-07-09"}
mappings:
  - {source: pr.aa-01, target: ac-2, relationship: intersects-with}
```

Crosswalks assist authoring (suggest candidate `satisfies` targets) and
future inference; **roll-ups and coverage reports compute from explicit
`satisfies` tuples only** — inferred/transitive coverage is a possible
later enhancement, kept out of v1 so the leadership numbers stay
explainable.

## 4. Repository layout

Layers are separate directories because **review routing is computed from
which layer a diff touches** (§7–8).

```
security-controls/
├── DESIGN.md
├── README.md
├── REVIEW.md                    # sign-offs: scope, gates, spot-checks
├── LICENSES/                    # per-source license + attribution
├── schemas/                     # JSON Schema: kernel + per-framework
│                                #   optional fields; additive-only
├── sources/                     # vendored upstream snapshots — the ONLY
│                                #   input converters read; fetched by
│                                #   discover, never hand-edited
├── sources.lock                 # per source: URL, version, sha256, trust tier
├── requirements/                # LAYER · THE PRODUCT — official company
│   └── network-security/        #   requirements, one file each, by domain
│                                #   (Policy & Standards stay in Archer, §1)
├── profiles/                    # LAYER — org parameter values (+ later:
│                                #   800-53 ODP tailoring, exceptions)
├── catalogs/                    # LAYER — generated framework substrate
│   ├── nist-sp800-53/r5/        #   paths pin MAJOR revision only; exact
│   ├── nist-csf/2.0/            #   version in version.yaml/provenance
│   └── hipaa-security-rule/2013/#   thin citation catalog (P3)
├── mappings/                    # LAYER — imported crosswalks (IR 8477)
├── rendered/                    # generated: resolved human-readable
│   ├── requirements/            #   requirement docs (params substituted)
│   └── reports/                 #   coverage, CSF roll-up, crosswalks
├── tools/
│   ├── ingest/                  # deterministic converters, one per source,
│   │                            #   golden fixture each; shared serializer
│   ├── render/                  # parameter resolution + report generation
│   ├── gate/                    # deterministic PR classifier (§7)
│   └── validate/                # schema, mapping integrity, regen-diff,
│                                #   params, index, fixtures, coverage
├── .claude/agents/              # discover.md, author.md, summarizer.md
└── .github/
    ├── CODEOWNERS               # per-layer human review routing
    └── workflows/               # validate.yml, gate.yml, discover.yml,
                                 #   staleness.yml (scheduled), release.yml
```

Versioning is two-axis: paths and cross-references pin the **major
revision** (`r5`, `2.0`); exact upstream versions live in `version.yaml`,
`sources.lock`, and provenance — a patch bump stays confined to `sources/`
+ `catalogs/` + lock, auto-mergeable per §8. Repo-level semver tags version
our own releases; the `corp` framework versions with the repo itself.

**Release and history management is delegated to GHE** — no hand-maintained
version counters or changelogs in content files:

- `release.yml` tags and publishes a GitHub Release on every merge to main
  that touches the product layers (`requirements/`, `profiles/`,
  `mappings/`), attaching the rendered reports (crosswalks, coverage,
  roll-ups) as immutable artifacts — "requirements as of vX.Y.Z" without
  reading git. The workflow stamps repo version + commit into the attached
  artifact copies at release time (committed `rendered/` files stay
  unstamped — they are regen-diff-checked and can't contain their own SHA).
- Per-requirement change history is git history: every requirements/
  profiles change is a reviewed PR, so who/what/when/approval is already
  recorded. If GRC later needs an effective date as data, the renderer
  derives it from the approving PR's merge date — never a hand-edited field.
- Consumers and auditors cite `requirement id + release tag`, never commit
  SHAs.
- A GHE ruleset restricts tag creation/deletion to maintainers — release
  history is audit-grade immutable.

## 5. Import pipeline (deterministic, not LLM transcription)

All generated canon is produced by converter scripts — re-runnable,
reviewable, faithful. Converters read **only** vendored snapshots in
`sources/` (pinned by `sources.lock`); neither converters nor CI ever fetch
upstream — only discover does, by committing snapshot + lock together.
Framework text is never transcribed or paraphrased by an LLM.

| Source | Input | License | Phase / notes |
|---|---|---|---|
| 800-53r5 | usnistgov/oscal-content OSCAL JSON | Public domain | P1 thin; P2 full (+53B baselines) |
| CSF 2.0 | NIST CPRT JSON export | Public domain | P1 thin (hierarchy needed for roll-ups); full with P2 |
| CSF↔800-53 crosswalk | NIST CPRT/OLIR | Public domain | P1 — authoring aid + integrity anchor |
| HIPAA Security Rule | 45 CFR citations + SP 800-66r2/OLIR mapping | Public domain | P3 — thin citation catalog; healthcare compliance story |
| Company Policy & Standards | Archer REST/Content API (JSON; expect HTML-in-fields cleanup, ID lookups) | Company-owned | P4 — gated on §14 checks; enters as a thin citation catalog (parameter backing + mapping endpoints); industry-framework content never comes from Archer |
| CIS 8.1 | CIS WorkBench OSCAL | CC BY-NC-ND (company-licensed) | Deferred — after license-terms verification; verbatim source + attribution kept beside YAML |

**MCSB v2 is not a source at all** (decided 2026-07-09). It is itself
derived from 800-53/CIS/CSF mappings — a peer of our requirements layer,
not an input to it — and the company generates its own requirements rather
than reusing Microsoft's. Its assessment tooling (Defender for Cloud) is
not in use here; the company runs Prisma Cloud and Wiz, which measure
against custom frameworks directly (§12 P6). MCSB's contribution survives
only as the format that inspired the requirement schema (§3).

**Frameworks always come from upstream, never from Archer** (decided
2026-07-09): Archer's copies lag, export fidelity is uncontrolled, and its
content lineage carries UCF licensing risk (§14). Archer uniquely holds
the company's own Policy & Standards — the tier above this repo in the §1
authority model — and their cross-references. What we pull: standards
citations backing parameter values, and cross-references to seed
`satisfies` reconciliation.

### Trust tiers

Every `sources.lock` entry carries `trust: release-artifact | scraped`.
`release-artifact` = versioned, tagged upstream releases. `scraped` =
mutable web content. The tier drives §8: **text changes to existing catalog
entries from a `scraped` source never auto-merge** — a vandalized or
compromised page must not flow into canon without human eyes.

### Determinism rules (converters and renderer alike)

Regenerate-and-diff CI (§9) requires bit-identical output across machines
and days:

- Provenance dates come from `sources.lock` metadata — never the run date.
- All YAML/Markdown output goes through one shared canonical serializer
  (key order, wrapping, unicode policy); Python and dependencies pinned by
  lockfile; CI uses exactly that environment.
- `oscal_uuid` churns between upstream releases; semantic diff summaries
  ignore provenance-only changes or every patch bump looks huge.
- Each converter has a **golden fixture**: a miniature vendored source
  slice + expected output, committed and tested on every PR touching
  `tools/` — instant objective signal during converter work.

### Adding a framework (additive recipe)

Identical for every framework, thin or full; any deviation a new framework
forces is a design bug to fix then:

1. Vendor the artifact into `sources/` + `sources.lock` entry (trust tier).
2. One converter module in `tools/ingest/` (+ golden fixture).
3. Output lands in `catalogs/<framework>/<rev>/` (+ `index.yaml`,
   `version.yaml`).
4. Optional crosswalks in `mappings/`.

Validators and CI discover frameworks from the directory tree — no
hardcoded framework lists anywhere.

## 6. Authoring pipeline (the product path)

Requirements are drafted in interactive sessions — by a maintainer, or
AI-assisted with the human steering — never by the unattended loop:

1. **Scope**: v1 domains chosen with security leadership (P1), recorded in
   `REVIEW.md`; each domain gets a directory and id prefix (`ns-`, `dp-`,
   `im-`…) — MCSB's domain conventions are the naming reference.
2. **Draft**: from company knowledge, Archer content (once cleared), and
   framework text; crosswalks suggest candidate `satisfies` targets. AI
   drafting is welcome here — this is exactly the judgment-heavy work the
   interactive session is for.
3. **Parameterize**: any value an auditor could ask "why this number?"
   about becomes a `{{parameter}}` citing its backing Archer standard in
   the profile (§3); no backing standard = a standards gap raised to the
   standards owners.
4. **Validate**: schema, parameter resolution, mapping integrity, render.
5. **Approve**: PR reviewed by a maintainer other than the author —
   `requirements/` and `profiles/` never auto-merge (§8). `status: official`
   is set by the approving review, and `provenance.approved` records it.

`tools/render/` deterministically produces `rendered/requirements/`
(parameters substituted, per-domain Markdown for humans) and
`rendered/reports/` (coverage/gap, CSF roll-up, auditor crosswalks).
Rendered output is generated canon: regenerated in the same PR, covered by
regenerate-and-diff. A parameter change therefore *shows* its blast radius
in the PR diff — that visibility is a feature.

## 7. Agent maintenance model

Discrete tasks, minimal ceremony — and the hard rule survives unchanged:
**no LLM in the merge path**. Upstream text is attacker-influenceable; a
steerable classifier in front of auto-merge is a prompt-injection hole.
Every §8 condition is mechanically decidable, so the merge decision is code:

- **discover** (scheduled): polls upstream (oscal-content releases, CPRT
  endpoints; later Archer, MCSB). On change: commits snapshot +
  `sources.lock` update (the snapshot diff *is* the upstream change),
  opens an issue, kicks author. CI enforces its path scope
  (`sources/` + lock only).
- **author** (loop mode): runs converters, regenerates catalogs + rendered
  output, opens a PR. Never merges. (Requirement *drafting* is interactive
  only, §6 — the unattended author touches generated canon exclusively.)
- **impact flag** (deterministic script): when a catalog change touches
  control ids referenced by any requirement's `satisfies`, it lists the
  affected requirements on the PR/issue — maintainers see exactly which
  official requirements an upstream change threatens. No model needed.
- **merge gate** (deterministic script in `tools/gate/`, required status
  check): classifies every PR from changed paths, add/withdraw diffs,
  `status:` diffs, and lock trust tiers; applies the §8 table. No model,
  no judgment, no attack surface.
- **summarizer** (LLM, advisory only): human-facing semantic summary on
  escalated PRs, provenance noise filtered. No merge authority; output is
  a comment, never a gate input.

Separation of duties on GHE: two machine identities (GitHub Apps) — author
opens PRs, gate merges; branch protection enforces PR-only main, required
checks, no self-approval. CODEOWNERS routes layer diffs to humans.

**Runtime, phased:** prove the loop on a fixture branch (staged synthetic
upstream change), then GHE Actions with an org API key and the App
identities. Pin stable model IDs — never a preview model. In
`.claude/agents/*.md`: `claude-sonnet-5` for discover, author, summarizer
(orchestration and prose; no judgment-bearing merge step exists).

## 8. Human-in-the-loop policy

Routed by layer and trust tier, not by size. **Every row is computed by
`tools/gate/`; an LLM never makes this call.**

| Change | Handling |
|---|---|
| `sources/` snapshot + lock update (discover) | Auto-merge on green CI |
| Catalog sync from `release-artifact` source, no add/withdraw | Auto-merge on green CI |
| Regeneration/formatting/provenance-only diffs (incl. `rendered/`) | Auto-merge on green CI |
| Catalog text change from a `scraped` source | Human review |
| Upstream adds/withdraws controls | Human review + impact flag |
| Any `requirements/` or `profiles/` change | **Human review, always** |
| Any `mappings/` change | Human review |
| Converter/renderer/schema/gate/CI code change | Human review |

Human review = one approving review from a maintainer other than the PR
author, routed by CODEOWNERS per layer.

## 9. Validation CI (hard gates, every PR)

Framework-agnostic: validators discover catalogs from the directory tree.

1. JSON Schema validation of every YAML file (kernel + framework fields).
2. Mapping integrity — every `satisfies` target, crosswalk tuple, and
   baseline entry resolves against its catalog; enhancement ids resolve to
   the parent control file; references to withdrawn controls warn with
   `superseded_by` suggestions.
3. Parameter integrity — every `{{parameter}}` reference resolves in the
   profile; every profile parameter is referenced (orphans warn).
4. Regenerate-and-diff — converters and renderer re-run against `sources/`
   + authored content must reproduce `catalogs/` and `rendered/` exactly.
5. Snapshot integrity — `sources/` matches `sources.lock` sha256s.
6. Index consistency — `index.yaml` matches files and inline enhancements.
7. License/attribution file present for every source directory.
8. Golden fixtures — every converter reproduces its committed fixture
   (PRs touching `tools/`).
9. Coverage report generation succeeds; the report itself is warn-tier
   (gaps are work queue, not build failures).
10. Staleness (warn-tier, **scheduled workflow**): last-discover-check age
    per source — a dead discover loop surfaces even when no PRs flow.

## 10. Review gates

Per layer, recorded in `REVIEW.md` (date, commit, reviewer, sample list):

- **Tooling gate (P0/P1)**: human review of converters, renderer, schemas,
  gate classifier, CI workflows, agent definitions.
- **Requirements gate (per domain)**: every requirement file human-approved
  by a non-author maintainer (this is §8 policy, so it is continuous, not
  an event); domain sign-off in `REVIEW.md` when its v1 set is complete.
- **Catalog gate (per framework, at full-text upgrade)**: spot-check ~15
  files against upstream plus known-tricky spots (parameter insertion,
  withdrawn controls, HIPAA citation granularity). Executed by the
  **content-verifier agent** (§13), which emits a discrepancy report a
  maintainer reviews; report attached to `REVIEW.md`. Thin catalogs need
  only the pipeline checks — ids/titles are trivially verifiable.

## 11. Licensing & hosting

- **Company GitHub Enterprise from the start**, private to the org — clean
  provenance for internal security review; GHE supplies the branch
  protection, rulesets, CODEOWNERS, and App identities §7 depends on.
- Build order leads with the simplest licensing surface: public-domain
  NIST/HIPAA content while the pipeline matures.
- **Nothing UCF-derived ever lands in this repo** (UCF ToS prohibits
  copying/disclosure outside adopted internal policies; §14).
- CIS 8.1 only after license terms are verified and recorded in
  `LICENSES/`; MCSB (CC BY 4.0) attribution recorded if/when ingested.
- Archer remains the audit system of record; `LICENSES/` carries
  attribution per source — it is the provenance story for any internal
  audit of this repo.

## 12. Phases (one per session)

- **P0 — scaffold**: layout, schemas (kernel + requirement + profile +
  catalog + mapping), canonical serializer, validators, gate classifier,
  golden-fixture harness, CI green on empty content; GHE branch protection
  + CODEOWNERS + tag-protection ruleset (§4 release model).
- **P1 — product slice, end-to-end**: thin 800-53r5 + CSF 2.0 catalogs and
  the CPRT crosswalk (three small converters); org profile with first
  parameters; **network-security domain with 3–5 official requirements**;
  renderer + coverage/gap + CSF roll-up + crosswalk reports; `release.yml`
  publishing the first tagged Release with stamped report artifacts (§4).
  *This proves the entire value chain — author → validate → map → render →
  report — before any heavy substrate work.* The v1 domain scope needs security
  leadership's sign-off, but that is **async, not a session dependency**:
  P1 proceeds on network-security as a provisional slice; the scope
  decision lands in `REVIEW.md` whenever leadership confirms it.
- **P2 — full-text catalogs (pulled)**: 800-53r5 full converter (+53B
  baselines) and CSF full text; content-verifier agent + catalog gates.
  Triggered by P1's drafting experience — expected almost immediately, but
  it must be pulled, not scheduled.
- **P3 — requirements scale-out**: remaining v1 domains, driven by the
  coverage report as work queue; HIPAA thin citation catalog + 800-66r2
  mapping lands here (healthcare compliance story: requirements traceable
  to Security Rule citations).
- **P4 — Archer intake**: export company Policy & Standards +
  cross-references via REST/Content API; standards enter as a thin
  citation catalog (the HIPAA treatment) so profile `standard:` citations
  become resolvable mapping endpoints; cross-references seed `satisfies`
  reconciliation with authored requirements (gated on §14 in-company
  checks; skippable/reorderable if access stalls — authored requirements
  don't wait on it).
- **P5 — maintenance loop**: discover/author/gate/impact-flag/summarizer
  on a fixture branch, then GHE Actions with App identities → **done**
  per §1.
- **P6 (post-done)**: consumption tools (MCP server, `consult-controls`
  skill, enforcement hooks); CIS 8.1 ingestion (license verified);
  assessment-data intake for implementation % — the company's CNAPP tools
  (Prisma Cloud, Wiz) support custom compliance frameworks, so the natural
  path is exporting rendered `corp` requirements as a custom framework
  definition and letting those tools measure implementation against our
  own requirements directly; Archer push-back (rendered requirements →
  Archer) as a possible follow-on.

## 13. Build workflow & model plan

**Build model: Claude Sonnet 5 (`claude-sonnet-5`) for all interactive
build sessions** (decided 2026-07-09 — Fable access ends). The design
already assumed this could happen: correctness never rests on the model —
it comes from deterministic converters, golden fixtures, and CI hard
gates. Sonnet 5's profile fits (near-Opus coding/agentic quality; strict,
literal instruction-following), with three watch-points, each mitigated
below: judgment-dense prose (requirement drafting) leans harder on the
human review §8 already mandates; parser/edge-case work may take more
iterations (fixtures give the objective signal); and literal
instruction-following makes **this document the executable spec** — its
wording and CLAUDE.md's are load-bearing.

**Effort:** default `high`; use `xhigh` for judgment-dense work (schema
kernel, gate-classifier edge cases, requirement drafting). If reasoning
looks shallow on a hard problem, raise effort before prompting around it.
If a phase still stalls (repeated failed iterations on one problem), end
the session cleanly at a task boundary and re-run that task on Opus 4.8.

**Cadence: the phase is the unit of scope; the phase gate is the unit of
done.** Prefer one phase per session, but a phase MAY split across
sessions at a task boundary with an updated handoff — never rush a gate
to fit a session. Each phase ends at an objective gate (CI green on GHE
plus the relevant §10 gate); a human sits at every phase boundary. This
document is the plan — don't re-plan per session (exceptions: P4 and P5
get a plan-mode pass first; they involve new integration surface, not
execution of this doc).

### Per-session execution protocol

1. **Catch up**: read `.remember/remember.md`, then the phase's §12 entry
   and every DESIGN.md section it references. No work before both.
2. **Task list**: break the phase into tracked tasks; every task's
   completion criterion is an **objective check** (validator passes,
   fixture green, regen-diff clean) — never self-assessment.
3. **Fixtures first**: for converter/renderer work, commit the golden
   fixture (input slice + expected output) before writing the tool;
   iterate until it passes.
4. **Fan out side-work** so raw payloads stay out of main context:
   validation/CI runs → test-runner; upstream format spelunking (raw
   OSCAL/CPRT payloads) → researcher; multi-file code questions →
   Explore. Independent units (separate converters, separate validators)
   may run as parallel subagents. **Judgment work — schema decisions,
   requirement drafting, anything in an authored layer — stays in the
   main session with the human.**
5. **Verify on objective signal only**: validators pass, fixtures green,
   regen-diff clean, CI green on the pushed branch. Not done until green.
6. **Commit and push at every green milestone** — the maintainer reviews
   on GitHub; an unpushed commit is invisible.
7. **End**: update the handoff note, stating plainly what is
   done-and-green vs. in-progress.

### Guardrails (hard)

- **Never redesign mid-phase.** If executing a step seems to require
  deviating from this document or a CLAUDE.md hard rule, stop and surface
  it to the human — a forced deviation is a design bug to fix in the
  design, not an obstacle to improvise around.
- **Escalate ambiguity, don't guess** — sessions are interactive; ask
  when a decision isn't derivable from this doc, the code, or convention.
- **Scope discipline** — build exactly what the phase names; no extra
  features, abstractions, or error handling beyond the task.

**Unattended loop (P5):** pinned `claude-sonnet-5` for
discover/author/summarizer; the merge gate and impact flag are scripts
and use no model at all. Never a preview model in the loop.

**Build-time agents & skills** (distinct from §7 product agents, which are
P5 deliverables):

- No agent-per-phase; stock agents for side-work (test-runner, researcher).
- One custom agent: **content-verifier** — samples converted files, fetches
  upstream, compares field-by-field, emits the §10 discrepancy report.
  An agent (not a skill) because its bulky inputs must stay out of main
  context. Created in P2 with the first full-text catalog; reused at every
  catalog gate.
- Skills: none hand-authored up front. The project verify skill
  auto-bootstraps via `/verify` once `tools/validate/` exists. The
  consumer-facing `consult-controls` skill is a **P6 product deliverable**.
  A phase-start skill is deferred until re-explaining the rhythm becomes a
  real friction signal.

## 14. Open questions (gate P4 Archer intake — nothing else)

Public research (2026-07-09) settled feasibility: Archer's REST/Content API
exports records as JSON with OData paging (~1000/page practical; SOAP caps
at 250/page); cross-reference fields carry the authoritative-source ↔
Control Standards links, so company standards export *with* their framework
mappings. Expect cleanup: HTML embedded in text-area fields;
cross-references return internal IDs needing secondary lookups. Archer is
migrating Authoritative Sources → Citations/Obligations applications (2026
compliance core) — the converter targets whichever model the instance runs.
UCF is *not* bundled in current Archer (separate Network Frontiers license;
its ToS prohibits extraction), so exposure is an in-company question.

Answerable only in the company instance/contract:

1. **Control Standards Library provenance** — company-authored (clean to
   export) vs Archer-shipped library (largely ISO-based;
   reuse-outside-Archer terms live in the master agreement)? Pre-2015
   Archer content was historically UCF-sourced — if seeded that long ago,
   check the Network Frontiers agreement.
2. **UCF exposure** — is the UCF Common Controls Hub integration in use?
   If yes, identify and exclude UCF-derived records.
3. **API access & instance model** — REST/Content API enablement, SaaS
   throttling, Authoritative Sources vs Citations/Obligations.

None of these gate P0–P3: the product path runs on authored requirements
and public-domain substrate until Archer is cleared.
