# quadrano-gtm-template Charter

**Version 1.0.0** · adopted 2026-10-25 · ratified when merged to `main` · see the [amendment log](#amendment-log)

This Charter is the rulebook for all work in `quadrano-gtm-template`. Where the Charter and anyone's habits disagree, the Charter wins.

It comes in two parts that must always agree:

- **`.kittify/charter/charter.yaml`** is the machine-readable form that Spec Kitty loads into agent context: governance settings plus the numbered project directives `QDR-001` to `QDR-012`.
- **This document** is the narrative form. Spec Kitty reads its *Policy Summary*, *Terminology Canon*, *Code Review Checklist* and *Regression Vigilance* sections verbatim into implement and review prompts, so keep those headings exactly as they are.

## Policy Summary

- Provenance (QDR-001): everything is original or from compatible public sources; nothing from any non-public third-party source; prior ventures are never named in commits, code, docs or Spec Kitty artefacts.
- The repo is a public, Apache-2.0 GTM Custom Tag Template (QDR-002): `template.tpl`, `metadata.yaml`, template tests, a README with the permission justifications, a CHANGELOG and SemVer release tags. The template configures and loads the hosted script; it never embeds the runtime, compiler or builder.
- The template asks for the least privilege in GTM's sandbox (QDR-003): `inject_script` limited to `https://cdn.quadrano.com/v1/*`, `access_globals` limited to writing `_quadrano`, and `logging` in debug mode only — each with a written justification in the README.
- Every push passes the minimal gates (QDR-004): template tests, no secrets, provenance, in that order. Nothing heavier is required here.
- No secrets in the repo (QDR-005): no GTM container IDs, keys or tokens in code, config, tests or commits.
- The template depends only on the frozen published-artefact contract (QDR-006); changing the contract is a major amendment with Product Owner sign-off.
- The `.tpl` carries template tests — at least one success and one failure or edge case per behaviour (QDR-007).
- If the Charter doesn't cover it, stop and ask the Product Owner (QDR-011); after three failed fix attempts, block the work package with a structured report.

## Roles

| Role | Who | Responsibilities | Never |
|---|---|---|---|
| **Architect** | AI agent | Writes specifications and plans, breaks work into work packages, reviews work and re-runs the gates, proposes Charter amendments | Writes production code |
| **Implementer** | AI agent | Builds one work package at a time in its own worktree, follows the Charter literally, records evidence | Improvises outside the Charter, proposes amendments |
| **Product Owner** | Barry | Approves specifications. Personally approves changes to the published-artefact contract, the template's sandbox permissions and the provenance rule. Merges to `main`. | — |

Plans and implementation are left to the agents.

## Provenance (QDR-001)

Quadrano is original work. All code, text and assets are original, or come from public sources under licences compatible with commercial use, with attribution where the licence requires it. Nothing may come from any non-public codebase or document belonging to a third party, including former employers and former ventures.

Commits, code, comments, documentation and Spec Kitty artefacts (interview answers, specs, plans, glossary entries, activity logs) never name prior ventures. This includes passing references: describe lessons generically ("experience shows that…").

The provenance gate enforces this. It scans every tracked file, every file version introduced by the commits being pushed, and every commit message against a denylist that is deliberately **held outside the repository**: a local file under the user profile, and a GitHub Actions secret in CI. Holding it outside means the guard never writes the protected names into the repo itself. If the denylist is missing, empty or unreadable, the check fails rather than passing silently. Output never repeats a matched term.

Only the Product Owner may change this rule.

## Repo shape, licence and scope (QDR-002)

| Concern | Approved |
|---|---|
| Repo | `Quadrano-Ltd/quadrano-gtm-template`, **public**, licensed **Apache-2.0** (ADR 0011) |
| Contents | `template.tpl`, `metadata.yaml`, template tests inside the `.tpl`, a `README` that documents the sandbox permissions and their justifications, a `CHANGELOG`, and SemVer release tags. The web app serves the compiled `.tpl` for download. |
| The template itself | A GTM Custom Tag Template that configures and loads the hosted Quadrano script. It never embeds or re-implements `@quadrano/runtime`, `@quadrano/compiler` or the builder; those live in `quadrano-web`. |
| Gallery | Community Template Gallery submission is planned for **after the talk**, not on the critical path — attendees import the `.tpl` file directly. |

This repo is a parallel lane in Horizon 1. It has no backend, no accounts, no database and no build system of its own, and it does not import or depend on the quadrano-web codebase. Anything not listed here needs a Charter amendment.

## GTM sandbox constraints (QDR-003)

The template runs inside customers' GTM containers under GTM's sandbox, so it asks for the least privilege and nothing more. Every permission is justified in writing in the `README`, as the build plan's M06 requires:

| Permission | Limited to | Justification (recorded in the README) |
|---|---|---|
| `inject_script` | `https://cdn.quadrano.com/v1/*` | The template loads the hosted bundle for the chosen spec ID and nothing else; the origin is pinned to the CDN so no other script can be injected. |
| `access_globals` | writing `_quadrano` | The template sets the config global `window._quadrano` and nothing else; it reads and writes no other global. |
| `logging` | debug only | Diagnostic messages go to GTM's debug log, never to the customer's Console output, which belongs to the runtime's format v1. |

The template writes `window._quadrano` exactly once with the four contract fields only, in the order `specId`, `mode`, `dataLayerName`, `logLevel` (`mode` is always `'console'`), then injects `https://cdn.quadrano.com/v1/{specId}.js`. The spec-ID field is validated as 12 characters of lowercase base32 (`a`–`z`, `2`–`7`) — the contract's format, implemented once by `@quadrano/schema`. No permission beyond these three, and no change to them, without an amendment.

## Quality gates (QDR-004)

Every push must pass these gates, in this order. The single entry point runs them all and stops at the first failure.

1. template tests — the `.tpl` tests run inside GTM's sandbox (QDR-007)
2. no-secrets — no secret-like value, and no GTM container ID, anywhere in the repo (QDR-005)
3. provenance — the check described in QDR-001

The exact commands are fixed at specification time and recorded in `AGENTS.md`. Nothing heavier is required in this repo. Agents never bypass a hook and never weaken a gate to get a change through.

## No secrets in the repo (QDR-005)

No GTM container ID, API key, token or other secret appears in code, config files, template tests or commits. Secrets for CI or deployment live in environment and secret values, never in the repository. The no-secrets gate fails a push that introduces one.

## Published-artefact contract dependency (QDR-006)

The template depends only on the frozen published-artefact contract v1.0 (signed off 2026-10-23), not on how `quadrano.com` is built — those are separate repos and may change underneath us. The contract fixes:

- the URL scheme `https://cdn.quadrano.com/v1/{specId}.js` and `/v1/{specId}/{version}.js`
- the `specId` format: 12 characters of lowercase base32 (`a`–`z`, `2`–`7`)
- the `window._quadrano` config global and its four fields: `specId`, `mode` (`'console'`), `dataLayerName`, `logLevel`
- Console output format v1: every message, level and rendering rule
- the runtime guarantees G-1 to G-12 (never throws, pass-through, no mutation, no network, idempotent install, second-Specification warning, one global, config fallbacks, missing data layer, replay order, snippet/hosted agreement, nonce)

This repo must not redefine any of them. Changing the contract is a **major** amendment with Product Owner sign-off. Until then it is the shape of truth.

## Template tests (QDR-007)

The `.tpl` carries template tests that run inside GTM's sandbox. Every behaviour gets at least one success case and one failure or edge case. The tests exercise:

- the spec-ID field validation (12 characters of lowercase base32)
- the log-level field (`all` / `errors`) and the data-layer-name field (a JavaScript identifier)
- the sandbox permissions: `inject_script` restricted to the CDN, `access_globals` restricted to `_quadrano`
- the config global the template writes: four fields, the right order, `mode` always `'console'`

A behaviour without a test is not done.

## Verification that proves something (QDR-008)

A passing test suite counts as evidence only once someone has seen it fail.

- Each required assertion is paired with a **deliberate break** in the template that must make it fail. The implementer runs each pair and records the result.
- Assertions match the requirement's exact wording. A check that can never fail is removed.
- Reviewers re-run the gates themselves rather than trusting the implementer's report.

## Terminology Canon

These terms are used exactly, and with these meanings, in code, the `.tpl`, the README, the CHANGELOG, specifications and plans. The Spec Kitty glossary holds the same definitions.

| Term | Meaning | In this repo |
|---|---|---|
| **Event Definition** | The rules for one data layer event, identified by its `event` value: its Attributes and what each must satisfy. The unit of reuse. | referenced, not built here |
| **Attribute** | One key in a pushed event object, together with its rules (required, type, allowed values, pattern, …) | referenced, not built here |
| **Data Layer Specification** ("Specification" for short) | A named set of Event Definitions for one website or group of websites; the unit of publishing (one spec ID, one script, one GTM tag) | The spec ID the template's field accepts |
| **Event Catalogue** | Quadrano-curated Event Definitions, read-only; entries are *copied* into an Event Library | referenced, not built here |
| **Event Library** | An Organisation's own Event Definitions, *referenced* by its Specifications | referenced, not built here |
| **Version** | An immutable, numbered snapshot of a Specification taken at publish; what the CDN serves | referenced, not built here |
| **GTM Custom HTML** | A GTM tag *type*: a free-form `<script>` that runs on the page. The delivery used by Quadrano Local's snippet. **Not** this repo's Custom Tag Template. | contrasted with the template |
| **snippet** | The Quadrano Local Custom HTML snippet: one GTM Custom HTML tag that sets `window._quadrano` and injects the bundle inline, under the contract's §6 rules. | not built here; the template is its hosted counterpart |
| **container** | The customer's GTM container, which the template is installed into. The template cannot update containers already in the field. | the deployment target |

**Rules:**

- Don't introduce synonyms for the six canonical terms: no "schema" for a Specification, no "tracking plan", "rule set" or "spec" in UI text. "Spec ID" is the one sanctioned short form.
- Use "Attribute", not "parameter", "property" or "field", except when quoting a vendor's own documentation.
- Spell "Catalogue" the British way, everywhere.
- "JSON Schema" names only the technical format an Event Definition compiles to; it is never a UI noun.
- Keep the three template-specific terms distinct: **GTM Custom HTML** is a tag type, **snippet** is the Quadrano Local delivery, and **container** is where either installs. The template is neither the snippet nor Custom HTML — it is a GTM **Custom Tag Template** (`.tpl`).
- The canonical set for this repo is the six terms above plus the three template-specific terms. A new domain term needs a glossary entry and a Charter patch amendment before it appears in code.

## Commits, branches and releases (QDR-009)

- Commits are atomic and follow Conventional Commits, scoped to the work package, e.g. `feat(WP01): add log-level field validation`. Agent-authored commits end with an `Agent: <name>` trailer.
- Stage only the work package's own deliverables; never use blanket staging.
- No direct commits or pushes to `main`. Every change reaches `main` through a pull request with green gates.
- Spec Kitty creates and names work-package branches (`kitty/mission-*`).
- Releases use SemVer version tags, and the `CHANGELOG` is updated in the same change. Gallery submission waits until after the talk.

## Escalation (QDR-011)

If the Charter or the briefing does not cover a situation, or a needed change isn't permitted, the implementer stops and escalates to the Product Owner instead of improvising.

After **three** failed attempts to fix a failing gate or test, the work package moves to *blocked* with a structured error report. The report gives:

- what was attempted
- the exact failing command and its output
- the suspected cause
- what is needed to unblock it

## Code Review Checklist

Implementers check every item before moving a work package to review. Reviewers verify every item independently, re-running commands rather than reading reports.

1. **Gates:** the minimal gate set passes locally (template tests, no-secrets, provenance) (QDR-004).
2. **Scope:** the diff touches only the work package's own deliverables — `template.tpl`, `metadata.yaml`, template tests, README, CHANGELOG. Nothing outside this repo's surface (QDR-002).
3. **Provenance:** nothing copied from a non-public source, and no prior venture named anywhere in the diff or the commit messages (QDR-001).
4. **No secrets:** no GTM container ID or keys in code, config, tests or commits; they live in environment/secret values (QDR-005).
5. **Sandbox:** permissions are exactly `inject_script` (`https://cdn.quadrano.com/v1/*`), `access_globals` (write `_quadrano` only) and `logging` (debug only), each justified in the README; `window._quadrano` is written once with the four contract fields in order (QDR-003).
6. **Contract:** nothing in the published-artefact contract changed; the spec-ID validation and config global match contract v1.0 (QDR-006).
7. **Template tests:** each behaviour has at least one success and one failure or edge case, and each required assertion has a recorded deliberate-break pairing (QDR-007, QDR-008).
8. **Terminology:** the Terminology Canon is used exactly; no new synonyms (QDR-010).
9. **Commits:** Conventional Commits scoped to the work package, with the `Agent:` trailer; atomic; no blanket staging (QDR-009).

## Regression Vigilance

- **Never weaken a test to pass.** A failing test means either the change is wrong or a requirement changed. Escalate if it's the latter.
- **Guard the published-artefact contract.** It is frozen; every template change is checked against the contract's URL scheme, spec-ID format, config global and Console output format v1 (QDR-006).
- **Guard the sandbox.** A permission that grows beyond the three named grants, or a `window._quadrano` write that drifts from the four contract fields, is a regression and a potential break for every container in the field.
- **Terminology cutovers are bulk edits.** Renaming a canonical term follows Spec Kitty's bulk-edit occurrence classification, and covers the `.tpl`, README, CHANGELOG and the glossary in one mission. A half-renamed term is a regression.
- **Bugs get a test first.** A fixed defect gets a template test that failed before the fix. If a class of defect recurs, close it by construction (a field-validation rule, a permission check) rather than patching each case.

## How the Charter is enforced

Written rules alone don't reliably change agent behaviour, so wherever possible each rule is backed by a mechanism. When a rule changes, its mechanism must change in the same pull request.

| Rule | Enforced by |
|---|---|
| Quality gates (QDR-004), template tests (QDR-007), provenance (QDR-001) | The gate runner — the single entry point fixed at specification time and recorded in `AGENTS.md`, plus a git pre-push hook / CI once the tooling is established. The provenance denylist lives outside the repository and the check fails closed when it is missing. |
| No secrets (QDR-005) | The no-secrets gate, environment/secret values for real credentials, and Code Review Checklist item 4. |
| Sandbox constraints (QDR-003) | The `.tpl`'s `metadata.yaml` permission declarations, the written justifications in the README, and template tests that assert the three grants; Code Review Checklist item 5. |
| Published-artefact contract (QDR-006) | The frozen contract v1.0; template tests and Code Review Checklist item 6; Product Owner sign-off for any change. |
| Repo shape, licence and scope (QDR-002) | The Apache-2.0 LICENSE, the fixed file set (`template.tpl`, `metadata.yaml`, tests, README, CHANGELOG), Code Review Checklist item 2, and review — anything new needs a Charter amendment. |
| Verification discipline (QDR-008), escalation (QDR-011) | The *Code Review Checklist* and *Regression Vigilance* sections above, which Spec Kitty injects into implement and review prompts. |
| Commits, branches and releases (QDR-009) | Git branch protection on `main`, Conventional Commits, SemVer tags, and review. |
| Terminology (QDR-010) | The Terminology Canon above, `charter.yaml`'s QDR-010, and the Spec Kitty glossary. |
| Changing the Charter (QDR-012) | The amendment process itself: a versioned change with an amendment-log row and a matching enforcing-mechanism change in the same pull request, reviewed by the Product Owner before it reaches `main`. |

## Changing the Charter (QDR-012)

- Only the Architect proposes amendments; implementers never do.
- Amendments are versioned like software: **patch** for a clarification or correction, **minor** for a new rule, **major** for a rule changed or removed.
- Every amendment:
  - updates the version and date at the top of this document
  - keeps `charter.yaml` and this document in agreement
  - explains its reasoning in the commit message
  - adds a row to the amendment log
  - changes the enforcing mechanism in the same pull request, or states that none applies
- Like every change to `main`, an amendment ships through a pull request with green gates.
- Exceptions to any rule need an amendment; there are no ad hoc deviations.

## Amendment log

| Version | Date | Type | Summary |
|---|---|---|---|
| 1.0.0 | 2026-10-25 | — | Initial Charter: roles, provenance, repo shape/licence/scope, GTM sandbox constraints, quality gates, no secrets, published-artefact contract dependency, template tests, verification and evidence, Terminology Canon (six canonical terms plus GTM Custom HTML, snippet, container), commits/branches/releases, escalation, enforcement mechanisms and amendment process |
