# security-controls — Design

**Status:** v6 · 2026-07-15 — **published, build-ready**; execution
starts at P0 (§12). v6 = independent security-architecture review round:
gate executes from base ref (§7), head-SHA approvals (§8), normative
reference grammar (§3), per-phase exit checklists (§12), Actions-runner
prerequisite (P0), approval dates derived from PRs — `provenance.approved`
dropped (§3/§6), referenced-control text changes escalate to human review
(§8), spec-gap closures (`sources.lock` format, staleness data source,
branch convention, vacuous-pass rule, draft-status field optionality);
plus an independent cold-read verification round: gate workflow trust
mechanism (`pull_request_target`, identity-pinned check), authored-vs-
regenerated `mappings/` gate distinction, tag-ruleset release allowlist,
loop secrets scoping, referenced-control scan moved to P0, `version.yaml`
schema, protection activation ordering, P3 prerequisite + source-recipe
boxes, tooling-gate recording point, `corp-std` path form, `.gitignore`,
content-verifier egress fallback. Carries forward every v5 decision. v5 = executability round (sources
bootstrap path, schema file classes, gate precedence + decision-table
fixtures, phased GHE protection activation, pinned toolchain, per-phase
human prerequisites, DESIGN.md as the only committed spec — no committed
CLAUDE.md) plus a publish-readiness pass (content-class boundaries,
`corp` version.yaml exemption, workflow phasing, release semver rule).
**Owners:** maintainer team of 2–3 (blameworthy lead); **two maintainers
from the start** — §8's non-author review requires it, so onboarding the
second maintainer is a P0 human prerequisite (§12). If the team ever
drops to one: requirement changes merge only as `status: draft`
(promotion to `official` waits for a non-author approving review), and
`profiles/`, `mappings/`, and `tools/` changes wait for the second
maintainer — no solo merges there.

## 1. Goal

> **Every security decision at the company — by an engineer, an AI
> assistant, or an auditor — is answered from one authoritative, versioned
> set of official company requirements: clear, prescriptive,
> platform-neutral, parameterized, and traceably mapped to the industry
> frameworks they address.**

Short form: **official company security requirements as code — followed by
builders, provable to auditors.**

The product is the `requirements/` layer. Everything else is a means,
judged by how directly it serves the product:

| Element | Role |
|---|---|
| `requirements/` — parameterized official requirements | **The product** |
| `mappings/` + inline `maps_to` tuples | The proof mechanism — audits, crosswalks, CSF roll-ups |
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
  `maps_to` mappings to framework controls. CI generates the proof:
  coverage/gap reports (framework controls with no covering requirement),
  auditor crosswalks ("every requirement mapped to AC-2, with resolved
  parameter values"), and CSF 2.0 roll-ups — % views per category and
  function for leadership. Two numbers, never conflated: **coverage %**
  (subcategory has ≥1 mapped requirement — computable from the repo, day
  one) and **implementation %** (requirement verifiably followed — needs
  assessment data from Archer or scanning tools; a later input that slots
  into the same roll-up).
- **Context-efficient for machines.** One small YAML file per unit; agents
  load only what a task needs, discovered via `index.yaml`, never a whole
  framework or domain. The small-file cap: non-catalog content files stay
  roughly under 400 lines (catalog files are exempt, §3).
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

- A requirement's `maps_to` tuples point *up* at framework controls —
  compliance traceability. **Technical controls** are the distinct second
  mapping direction, pointing *down*: measurable checks (Prisma/Wiz
  policies, config baselines) that inherit the requirement's parameters
  and return pass/fail (`tls_min_version = TLS 1.3 → pass`).
- A requirement is **implemented** when every technical control mapped to
  it passes; implemented requirements roll up through `maps_to` and the
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
   `maps_to` tuples on every merge to main.
3. **Substrate as pulled** — every `maps_to` endpoint resolves against a
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

Two classes with opposite rules. Layer directories belong to exactly
one — except `mappings/`, which holds both, and regenerate-and-diff tells
them apart: files a converter reproduces are canon, the rest are
authored. `sources/` (vendored input, §5) and code — `schemas/`,
`tools/`, `.github/` — sit outside both classes; §8 routes their changes.

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

Content files divide into two schema classes, each validated against its
own JSON Schema in `schemas/` (§9 check 1):

- **Unit files** — catalog entries and requirements — carry the required
  kernel: `id`, `framework`, `title`, `status`, `provenance`, plus
  optional typed fields per framework shape. **Additive-only**: the
  kernel never changes; onboarding a framework may add optional fields
  but never touches existing ones.
- **Structural files** — profiles, mapping files, `index.yaml`,
  `version.yaml`, `sources.lock` — do not carry the kernel; each shape
  has its own schema (the examples and field lists in this document are
  normative), extended under the same additive-only contract.

### Reference grammar (normative)

One grammar for every cross-reference; validators parse exactly this,
and any other surface form in a content file is a validation failure:

- **Framework slug**: lowercase kebab, version-free — `nist-sp800-53`,
  `nist-csf`, `hipaa-security-rule`, `corp`, `corp-std`.
- **Reference form** (`maps_to` targets, mapping endpoints,
  `source_framework`/`target_framework`): `<slug>@<rev>/<unit-id>` for a
  unit, `<slug>@<rev>` for a framework; `<rev>` is the major revision
  only (`@r5`, `@2.0`, `@2013`), per §4. `corp` and `corp-std` are
  unversioned (they version with repo releases, §4) and omit `@<rev>`:
  `corp/ns-1`, `corp-std/crypto-001`.
- **Path form**: revision as directory, no `@` —
  `catalogs/<slug>/<rev>/…`; `requirements/<domain>/…` for `corp`;
  `catalogs/corp-std/` with **no revision directory** for `corp-std`
  (unversioned slug — it versions with Archer export snapshots, so it
  *does* carry `version.yaml` + a `sources.lock` entry like any
  generated catalog; only `corp` is exempt).
- **Filename form** (mapping pair directories, where `@` and `/` are
  unavailable): `@` becomes `-` —
  `mappings/nist-csf-2.0__nist-sp800-53-r5/`.
- **Unit ids**: lowercase, framework-native (`sc-8`, `pr.aa-01`, `ns-1`);
  enhancement ids suffix the parent (`sc-8.1`).

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
maps_to:
  - target: nist-sp800-53@r5/sc-8
    relationship: subset-of
  - target: nist-csf@2.0/pr.ds-02
    relationship: intersects-with
implementation:
  azure: |
    Enforce TLS {{tls_min_version}} minimum on App Service, Storage, ...
  aws: ...
  gcp: ...
  onprem: ...
provenance:
  origin: authored           # authored | derived-from-archer
```

Rules: the `requirement` statement never names a platform (the company runs
Azure, AWS, GCP, and on-prem — platform "how" lives only in
`implementation:`); values appear only as `{{parameter}}` references;
`maps_to` targets are ordinary IR 8477 tuples validated by mapping
integrity (§9); ids are stable and never reused; retired requirements stay
present with `status: retired` + `superseded_by`. `maps_to` and
`implementation` MAY be absent while `status: draft`; both are required
for `official` (schema-enforced, §9 check 1). Approval identity and date
are **never stored in unit files** — they derive from the approving PR's
review and merge record (§6), so a file can never disagree with GitHub's
audit trail.

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
when full-text catalogs arrive — and **ODP tailoring assignments reference
corp parameters, never restate values**: an ODP entry says
`param: tls_min_version`, not `value: "TLS 1.3"`, so
`profiles/corp/parameters.yaml` remains the only file where a value
literal exists and a requirement can never disagree with the 800-53
tailoring it maps to. Parameter integrity (§9 check 3) extends to ODP
references when tailoring lands. Parameter names are `noun_qualifier`
(`tls_min_version`); before adding one, scan the profile for an existing
parameter covering the same decision — semantic duplicates are exactly
what human review on `profiles/` exists to catch.

### Catalog schema (substrate — thin and full)

Catalogs enter **thin-first**: id, title, family/function lineage — enough
for `maps_to` endpoints to resolve, indexes to be scannable, and roll-ups
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
loading a framework. Every framework directory (including `requirements/`
domains) carries `index.yaml` — one line per unit: id, title, **status**,
enhancement ids + titles. Catalog directories also carry `version.yaml`
(exact upstream version); `requirements/` does not — the `corp` framework
versions with the repo's release tags (§4), never a hand-maintained
counter. `version.yaml` is a structural file (§3), normative example:

```yaml
# catalogs/nist-sp800-53/r5/version.yaml
framework: nist-sp800-53
revision: r5
source_version: "5.1.1+u5"   # exact upstream version, from sources.lock
```

`revision` is omitted for unversioned slugs (`corp-std`, §3 grammar).
The index is the agent entry point: scan it, load only the files the task
needs; status in the index means an agent filters `official` from `draft`
or `withdrawn` without opening a single unit file. Indexes are always
generated: converters emit catalog indexes, and the `requirements/` index
is regenerated by `tools/render/` in the same PR as any content change — never
hand-maintained (§9 check 6 fails drift either way).
Withdrawn controls stay present — mappings may still reference them, and
integrity checks suggest `superseded_by` repairs.

### Mapping schema (imported crosswalks)

Typed tuples per NIST IR 8477 (shared by OLIR, CPRT, SCF, OSCAL 1.2); the
relationship vocabulary is exactly IR 8477's five values —
`equal | subset-of | superset-of | intersects-with | no-relationship`.
One file per framework pair, split per family/function past ~300 lines.
Endpoints are any two catalogs in the repo — including `corp` and thin
regulatory catalogs (HIPAA citations). Requirements' `maps_to` blocks are
the authored siblings of these files, validated identically.

```yaml
# mappings/nist-csf-2.0__nist-sp800-53-r5/pr.yaml
source_framework: nist-csf@2.0
target_framework: nist-sp800-53@r5    # reference form, major rev only (§3)
provenance: {source: NIST CPRT crosswalk, imported: "2026-07-09"}
mappings:
  - {source: pr.aa-01, target: ac-2, relationship: intersects-with}
```

Crosswalks assist authoring (suggest candidate `maps_to` targets) and
future inference; **roll-ups and coverage reports compute from explicit
`maps_to` tuples only** — inferred/transitive coverage is a possible
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
├── schemas/                     # JSON Schema: unit kernel + per-framework
│                                #   fields + structural shapes (§3);
│                                #   additive-only
├── sources/                     # vendored upstream snapshots — the ONLY
│                                #   input converters read; written only
│                                #   by tools/ingest/fetch.py (§5), never
│                                #   hand-edited
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
├── .claude/agents/              # discover.md, author.md, summarizer.md;
│                                #   content-verifier.md arrives P2 (§13)
└── .github/
    ├── CODEOWNERS               # per-layer human review routing
    └── workflows/               # validate.yml + gate.yml (P0),
                                 #   release.yml (P1), discover.yml +
                                 #   staleness.yml (scheduled, P5)
```

Versioning is two-axis: paths and cross-references pin the **major
revision** (`r5`, `2.0`); exact upstream versions live in `version.yaml`,
`sources.lock`, and provenance — a patch bump stays confined to `sources/`
+ `catalogs/` + regenerated crosswalks in `mappings/` + lock,
auto-mergeable per §8 (the authored-vs-regenerated `mappings/`
distinction there is what keeps this true). Repo-level semver tags version
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
  Numbering is deterministic: `release.yml` auto-increments the minor
  version (first release `v0.1.0`); major bumps are human-triggered via
  workflow dispatch, reserved for declared milestones (`v1.0.0` = the §1
  definition of done met).
- Per-requirement change history is git history: every requirements/
  profiles change is a reviewed PR, so who/what/when/approval is already
  recorded. If GRC later needs an effective date as data, the renderer
  derives it from the approving PR's merge date — never a hand-edited field.
- Consumers and auditors cite `requirement id + release tag`, never commit
  SHAs.
- A GHE ruleset restricts tag creation/deletion to maintainers **plus the
  release workflow's identity** (`release.yml` creates the tags — without
  that allowlist entry the first P1 release fails tag-denied) — release
  history is audit-grade immutable.

## 5. Import pipeline (deterministic, not LLM transcription)

All generated canon is produced by converter scripts — re-runnable,
reviewable, faithful. Converters read **only** vendored snapshots in
`sources/` (pinned by `sources.lock`); converters and validation CI
never fetch upstream — **all import-pipeline upstream I/O happens inside
`tools/ingest/fetch.py`**, whether run interactively by a maintainer or
by the scheduled discover workflow (P5); the §13 content-verifier's
read-only spot-check fetch is the sole exception, and it writes nothing. Snapshots enter through exactly
one tool:
`tools/ingest/fetch.py` fetches an upstream artifact (or ingests a
human-supplied download via `--from-file`, for egress-restricted
instances) and writes snapshot + `sources.lock` entry together in one
commit. Pre-P5, a maintainer runs it interactively — this is how P1's
sources arrive; from P5, the discover agent runs the same module on a
schedule. Framework text is never transcribed or paraphrased by an LLM.

`sources.lock` is YAML (structural schema, §3): one entry per source
slug with `url`, `version`, `sha256`, `trust` (tier, below), `fetched`
(snapshot date — the date converters stamp into provenance), and
`last_checked` (refreshed by every discover poll from P5, even when
nothing changed; §9 check 10 reads its age).

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
`maps_to` reconciliation.

### Trust tiers

Every `sources.lock` entry carries `trust: release-artifact | scraped`.
`release-artifact` = versioned, tagged upstream releases. `scraped` =
mutable web content. The tier drives §8: **text changes to existing catalog
entries from a `scraped` source never auto-merge** — a vandalized or
compromised page must not flow into canon without human eyes.
Tier assignments are made here, not per-session: usnistgov/oscal-content
and CIS WorkBench artifacts are `release-artifact`; CPRT/OLIR JSON exports
and the Archer API export are `scraped` (no tagged upstream release
exists for them).

### Determinism rules (converters and renderer alike)

Regenerate-and-diff CI (§9) requires bit-identical output across machines
and days:

- Provenance dates come from `sources.lock` metadata — never the run date.
- All YAML/Markdown output goes through one shared canonical serializer
  (key order, wrapping, unicode policy). Pinned toolchain (decided
  2026-07-15): Python 3.12, `uv` with committed `uv.lock`, `ruamel.yaml`
  behind the canonical serializer, `jsonschema` for schema validation,
  `pytest` for fixtures and gate tests; CI runs exactly that environment
  via uv.
- `oscal_uuid` churns between upstream releases; semantic diff summaries
  ignore provenance-only changes or every patch bump looks huge.
- Each converter has a **golden fixture**: a miniature vendored source
  slice + expected output, committed and tested on every PR touching
  `tools/` — instant objective signal during converter work.

### Adding a framework (additive recipe)

Identical for every framework, thin or full; any deviation a new framework
forces is a design bug to fix then:

1. Vendor the artifact via `tools/ingest/fetch.py` into `sources/` +
   `sources.lock` entry (trust tier).
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
   framework text; crosswalks suggest candidate `maps_to` targets. AI
   drafting is welcome here — this is exactly the judgment-heavy work the
   interactive session is for.
3. **Parameterize**: any value an auditor could ask "why this number?"
   about becomes a `{{parameter}}` citing its backing Archer standard in
   the profile (§3); no backing standard = a standards gap raised to the
   standards owners.
4. **Validate**: schema, parameter resolution, mapping integrity, render.
5. **Approve**: the author sets `status: official` in the PR itself; the
   PR merges only with an approving review from a maintainer other than
   the author, covering the head SHA (§8) — `requirements/` and
   `profiles/` never auto-merge. Reviewer identity and approval date live
   in the PR record, never in the file; if an effective date is ever
   needed as data, the renderer derives it from the approving PR's merge
   date (§4).

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

- **discover** (scheduled): polls upstream via the §5 fetch module
  (oscal-content releases, CPRT endpoints; later Archer). On change:
  opens a snapshot + `sources.lock` PR under the author App identity —
  main is PR-only, and the snapshot diff *is* the upstream change —
  opens an issue, kicks author. CI enforces its path scope
  (`sources/` + lock only).
- **author** (loop mode): runs converters, regenerates catalogs + rendered
  output, opens a PR. Never merges. (Requirement *drafting* is interactive
  only, §6 — the unattended author touches generated canon exclusively.)
- **impact flag** (deterministic script): when a catalog change touches
  control ids referenced by any requirement's `maps_to`, it lists the
  affected requirements on the PR/issue — maintainers see exactly which
  official requirements an upstream change threatens. No model needed.
  Its scan is also a gate input: the §8 row escalating
  referenced-control text changes to human review uses this same
  deterministic check.
- **merge gate** (deterministic script in `tools/gate/`, required status
  check): classifies every PR from changed paths, add/withdraw diffs,
  `status:` diffs, and lock trust tiers; applies the §8 table. No model,
  no judgment, no attack surface. Ships with a decision-table fixture
  suite — every §8 row, mixed-layer diffs, and unclassified paths each
  have a committed case (§9 check 8). **The gate workflow always
  executes gate code from the base branch (main), never the PR head** —
  a PR touching `tools/gate/` or `.github/` is judged by the incumbent
  gate, so no PR can weaken the gate that decides its own merge. The
  mechanism, not just the property — a default `pull_request` trigger
  takes even the *workflow file* from the PR ref, silently voiding it:
  `gate.yml` uses the `pull_request_target` trigger (workflow definition
  and checkout from the base ref); the gate job never executes anything
  from the PR head — it classifies from changed paths and diffs read via
  the GitHub API; and the required check is identity-pinned so a
  PR-defined workflow can't go green under the same check name — **at P0
  via a required-workflows ruleset** (path-pinned to `gate.yml` on main;
  the App identities don't exist until P5), **from P5 via the gate
  App's check identity**. (The §9 validators run from the PR head as usual — they gate
  content correctness, not merge authority.)
- **summarizer** (LLM, advisory only): human-facing semantic summary on
  escalated PRs, provenance noise filtered. No merge authority; output is
  a comment, never a gate input.

Separation of duties on GHE: two machine identities (GitHub Apps) — author
opens PRs, gate merges; branch protection enforces PR-only main, required
checks, no self-approval. CODEOWNERS routes layer diffs to humans.

Secrets scoping for the unattended loop (LLM agents process
attacker-influenceable upstream text, so credentials are least-privilege
per workflow): discover fetches **only URLs registered in
`sources.lock`** — an injected instruction cannot turn it into an
arbitrary-URL exfiltration channel; the org API key and App credentials
are per-workflow Actions secrets, never shared across jobs; the
summarizer runs with a comment-only token — it can write PR comments and
nothing else.

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
| Catalog text change to a control referenced by any `maps_to` — **any trust tier** | Human review + impact flag |
| Upstream adds/withdraws controls | Human review + impact flag |
| Any `requirements/` or `profiles/` change | **Human review, always** |
| Authored `mappings/` change | Human review |
| Converter/renderer/schema/gate/CI code change | Human review |

Two classifier rules close the table: a PR matching multiple rows gets
the **most restrictive** handling (human review beats auto-merge), and a
path matching no row defaults to human review. Regenerated imported
crosswalks in `mappings/` are generated canon (§2) and follow the
regeneration and trust-tier rows, not the authored-mappings row — the
gate tells them apart the same way §2 does: a converter reproduces them
(regenerate-and-diff), so §4's patch-bump auto-merge stays true when a
source ships crosswalk updates.

Human review = one approving review from a maintainer other than the PR
author, **covering the PR's head SHA** — an approval that predates the
latest push never counts (stale-approval race). Routed by CODEOWNERS per
layer.

The referenced-control row is a deliberate tightening over pure trust
tiers: even a tagged upstream release (`release-artifact`) can be
compromised or wrong, and text changes to controls that official
requirements cite are exactly where that damage lands — so those changes
always get human eyes, while text churn in uncited substrate still flows
on tier rules alone.

**Activation is phased** (decided 2026-07-15): GHE cannot path-scope
review requirements except via CODEOWNERS, so P0 configures the simple
safe state — branch protection requires one non-author approving review
plus green required checks on every PR, with **"require approval of most
recent push" enabled** so stale approvals are dismissed; the gate runs as
a required check from P0 (in classify-only mode: it goes green when
classification succeeds, and branch protection carries the review
requirement) but its auto-merge lanes stay dormant. Activation ordering:
required-check protection is switched on **immediately after the
scaffold PR merges** — the workflows must exist on main before they can
be required, or the first PR deadlocks waiting on checks that never
report. At P5, when bot PRs
first exist, the blanket review requirement is lifted and the gate
becomes the review enforcer: its required check goes green only when the
matched row's conditions hold — for human-review rows, a non-author
approving review of the head SHA, verified by the gate itself — with
CODEOWNERS routing reviewers per layer. The switch is part of P5's
plan-mode pass (§13).

## 9. Validation CI (hard gates, every PR)

Framework-agnostic: validators discover catalogs from the directory tree.
Every check passes vacuously on an empty content tree — P0's "CI green on
empty content" gate depends on it. That includes checks 4 and 9: they
no-op until the converters/renderer they exercise exist (P1) — P0 ships
no stub renderer to satisfy them.

1. JSON Schema validation of every YAML file against its file-class
   schema (§3: kernel for unit files, structural schemas otherwise).
2. Mapping integrity — every `maps_to` target, crosswalk tuple, and
   baseline entry resolves against its catalog; enhancement ids resolve to
   the parent control file; references to withdrawn controls warn with
   `superseded_by` suggestions.
3. Parameter integrity — every `{{parameter}}` reference resolves in the
   profile; every profile parameter is referenced (orphans warn); ODP
   tailoring entries reference profile parameters, never value literals
   (activates when tailoring lands, P2+).
4. Regenerate-and-diff — converters and renderer re-run against `sources/`
   + authored content must reproduce `catalogs/` and `rendered/` exactly.
5. Snapshot integrity — `sources/` matches `sources.lock` sha256s.
6. Index consistency — `index.yaml` matches files and inline enhancements.
7. License/attribution file present for every source directory.
8. Golden fixtures — every converter reproduces its committed fixture,
   and the gate classifier passes its §8 decision-table suite (PRs
   touching `tools/`).
9. Coverage report generation succeeds; the report itself is warn-tier
   (gaps are work queue, not build failures).
10. Staleness (warn-tier, **scheduled workflow**): per-source
    `last_checked` age read from `sources.lock` (§5) — a dead discover
    loop surfaces even when no PRs flow.

## 10. Review gates

Per layer, recorded in `REVIEW.md` (date, commit, reviewer, sample list):

- **Tooling gate (P0/P1)**: human review of converters, renderer, schemas,
  gate classifier, CI workflows, agent definitions. Recorded **once, at
  P1**, covering P0+P1 tooling together — P0's exit needs only green CI
  and its merged non-author-reviewed PRs, no separate `REVIEW.md` entry.
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

Phases list **human prerequisites** where they exist — admin/access steps
the agent cannot perform. The session confirms them at catch-up (§13 step
1) and stops early if one is missing, rather than improvising.

Each phase carries a normative **exit checklist** — the complete
deliverable set, gathered here so nothing hides in a cross-reference.
§13 step 2 turns the checklist into the session task list; the phase
gate is every box green plus the relevant §10 gate.

- **P0 — scaffold**: layout, schemas (unit kernel + requirement + profile
  + catalog + mapping + structural), canonical serializer, validators,
  gate classifier + its §8 decision-table fixture suite, golden-fixture
  harness, `tools/ingest/fetch.py`, CI green on empty content; GHE branch
  protection in the P0 state (§8 activation) + CODEOWNERS +
  tag-protection ruleset (§4 release model). *Human prerequisites: repo
  live on company GHE with admin rights for protection rules; **GHE
  Actions runners available to the repo and able to install the pinned
  toolchain** (uv + Python 3.12 + locked deps — via egress or an internal
  mirror; `sources/` egress is solved separately by `--from-file`, §5);
  second maintainer onboarded (§1).*
  *Exit checklist:* ▢ §4 directory layout + `.gitignore` (local
  CLAUDE.md, `.remember/`, `.venv` — §13 requires the first two stay
  uncommitted) ▢ `schemas/` — kernel, requirement, profile, catalog,
  mapping, and structural shapes incl. `index.yaml`, `version.yaml`,
  `sources.lock` ▢ canonical serializer + committed `uv.lock` ▢ §9
  checks 1–9 implemented, passing vacuously on empty content ▢ gate
  classifier + referenced-control `maps_to` scan (gate-internal, §7 —
  the P5 impact flag reuses it) + full §8 decision-table fixture suite
  ▢ golden-fixture harness ▢ `tools/ingest/fetch.py` incl. `--from-file`
  ▢ `validate.yml` + `gate.yml` (`pull_request_target`, gate from base
  ref, required-workflows-ruleset-pinned check, §7) ▢ CODEOWNERS
  ▢ branch protection P0 state incl. approval-of-most-recent-push,
  enabled right after the scaffold PR merges (§8)
  ▢ tag-protection ruleset incl. release-workflow allowlist (§4)
  ▢ `REVIEW.md` created ▢ CI green on GHE.
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
  *Exit checklist:* ▢ 800-53r5 + CSF 2.0 + CPRT crosswalk vendored via
  `fetch.py` (lock entries + `LICENSES/`) ▢ three converters, each with a
  golden fixture ▢ thin catalogs + `index.yaml` + `version.yaml`
  ▢ `profiles/corp/parameters.yaml` with first parameters ▢ 3–5
  network-security requirements at `status: official`, non-author
  approved ▢ `tools/render/`: `rendered/requirements/` + coverage/gap +
  CSF roll-up + crosswalk reports ▢ generated `requirements/` index
  ▢ `release.yml` + first Release `v0.1.0` with stamped artifacts
  ▢ tooling gate (§10) recorded in `REVIEW.md` ▢ CI green.
- **P2 — full-text catalogs (pulled)**: 800-53r5 full converter (+53B
  baselines) and CSF full text; content-verifier agent + catalog gates.
  Triggered by P1's drafting experience — expected almost immediately, but
  it must be pulled, not scheduled.
  *Exit checklist:* ▢ 800-53r5 full-text converter + 53B baselines ▢ CSF
  2.0 full text ▢ `.claude/agents/content-verifier.md` (§13) ▢ catalog
  gates (§10) run, discrepancy reports attached to `REVIEW.md` ▢ CI green.
- **P3 — requirements scale-out**: remaining v1 domains, driven by the
  coverage report as work queue; HIPAA thin citation catalog + 800-66r2
  mapping lands here (healthcare compliance story: requirements traceable
  to Security Rule citations). *Human prerequisite: v1 domain scope
  confirmed in `REVIEW.md` (P1's async leadership decision landed) —
  without it "remaining domains" is unenumerable.*
  *Exit checklist:* ▢ remaining v1 domains authored + approved ▢ HIPAA
  source vendored via `fetch.py` (lock entry + `LICENSES/`) ▢ HIPAA
  converter + golden fixture ▢ HIPAA thin citation catalog + 800-66r2
  mapping ▢ per-domain sign-offs in `REVIEW.md` ▢ CI green.
- **P4 — Archer intake**: export company Policy & Standards +
  cross-references via REST/Content API; standards enter as a thin
  citation catalog (the HIPAA treatment) so profile `standard:` citations
  become resolvable mapping endpoints; cross-references seed `maps_to`
  reconciliation with authored requirements (gated on §14 in-company
  checks; skippable/reorderable if access stalls — authored requirements
  don't wait on it). *Human prerequisites: Archer API access; §14
  answers.*
  *Exit checklist (after plan-mode pass, §13):* ▢ §14 answers recorded in
  `REVIEW.md` ▢ Archer converter + golden fixture ▢ `corp-std` thin
  citation catalog ▢ profile `standard:` citations resolve as mapping
  endpoints ▢ cross-references seeded into `maps_to` reconciliation
  ▢ CI green.
- **P5 — maintenance loop**: discover/author/gate/impact-flag/summarizer
  on a fixture branch, then GHE Actions with App identities; §8
  auto-merge lanes activated (plan-mode pass first, §13) → **done**
  per §1. *Human prerequisites: two GitHub App identities created; org
  API key provisioned.*
  *Exit checklist (after plan-mode pass, §13):* ▢ `.claude/agents/`
  discover, author, summarizer committed, pinned `claude-sonnet-5`
  ▢ impact-flag PR/issue commenting wrapper (reuses the P0 gate-internal
  scan, §7) ▢ fixture-branch synthetic upstream change
  processed end-to-end ▢ `discover.yml` + `staleness.yml` scheduled
  ▢ App identities + org API key wired ▢ blanket review lifted, gate
  enforcing §8 incl. head-SHA approvals ▢ §1 definition of done verified
  and recorded in `REVIEW.md`.
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
wording is load-bearing.

**Portability & precedence:** this document is the single source of truth
and must execute standalone on any Claude instance — it is the **only
committed spec file**. No CLAUDE.md is committed (decided 2026-07-15): a
managed enterprise instance may inject or mandate its own CLAUDE.md
content, so a committed digest would drift or conflict with it. A session
MAY generate a local CLAUDE.md digest of the hard rules from this
document if the harness benefits; it is gitignored, never committed, and
never authoritative — this document wins on any conflict.
`.remember/remember.md` is session state — gitignored, never
committed (bootstrap rule in step 1 below
if absent). If the
hosting environment injects enterprise rules that conflict with this
document: environment policy wins on security/data-handling; a *process*
conflict is surfaced to the human, never improvised around. Named harness
features — subagents (test-runner, researcher, Explore), the `/verify`
skill, effort settings — describe the reference setup, not requirements:
where one is absent, run that work inline in the session, use
`tools/validate/` directly as the verify loop, and treat effort guidance
as advisory. Model IDs vary by hosting platform (first-party vs
Bedrock/Vertex naming): "pin `claude-sonnet-5`" means the platform's
stable Sonnet 5 ID, never a preview. Where egress is restricted, upstream
artifacts enter via `fetch.py --from-file` from a human-supplied download
(§5) — hash-pinned in `sources.lock` either way.

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
   and every DESIGN.md section it references. No work before both. If the
   handoff note doesn't exist (fresh clone, new instance), reconstruct
   state from `git log` plus which §12 phase gates are green in CI, write
   `.remember/remember.md` from that, and only then start.
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
   on GitHub; an unpushed commit is invisible. Convention: main is
   PR-only (§8); work on a phase branch (`p0-scaffold`,
   `p1-product-slice`), one PR per coherent reviewable deliverable
   group — a phase is typically 1–3 PRs, never one per micro-task.
7. **End**: update the handoff note, stating plainly what is
   done-and-green vs. in-progress.

### Guardrails (hard)

- **Never redesign mid-phase.** If executing a step seems to require
  deviating from this document, stop and surface it to the human — a
  forced deviation is a design bug to fix in the design, not an obstacle
  to improvise around.
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
  upstream (where egress is restricted: compares against the hash-pinned
  vendored artifact, or a human-supplied fresh download per §5
  `--from-file`, which additionally guards against vendoring corruption),
  compares field-by-field, emits the §10 discrepancy report.
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
