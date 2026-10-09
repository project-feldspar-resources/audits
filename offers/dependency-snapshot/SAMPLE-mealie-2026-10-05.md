# Dependency & Advisory Snapshot — sample deliverable

> This is a **specimen** produced as a fulfilment dry run for the $49 deterministic
> dependency/advisory snapshot offer. No buyer; public repo; point-in-time. It shows exactly
> what a buyer receives. The raw scanner JSON accompanies every real delivery.

**Repository:** mealie-recipes/mealie (public)
**Commit scanned:** `46497a95ef183ba011e2e4d950ee0ce618acb124`
**Scanned at:** 2026-10-05T16:07:09Z (UTC)
**Engine:** feldspar-discovery-scan (deterministic, dependency-free; OSS: github.com/project-feldspar-resources/feldspar-scan)
**Data sources:** OSV.dev (dependency advisories), public GHSA/CVE records, deterministic config/secret rules

---

## 1. What this is (and is not)

This is a **dependency and advisory snapshot**: a point-in-time inventory of your resolved
dependencies cross-referenced against public vulnerability databases, plus a lightweight config and
secret-pattern surface. Every item traces to a public advisory or a deterministic rule, so you can
re-run the open-source engine and reproduce it yourself.

It is **not** a source security review: no new vulnerability discovery, no exploit or proof-of-concept,
no authorization/tenant-isolation analysis, no compliance attestation. Those are separate services.

---

## 2. Dependency advisories (the priority list)

Three dependencies carry known published advisories at the scanned commit. **Note the upgrade column:
all three currently have no patched release listed in OSV** (the latest published version is still the
last-affected version), so the action today is mitigate/monitor, not upgrade.

| # | Package | Ecosystem | Installed | Advisory | CVSS (v3.1) | Class | Fixed version available? |
|---|---------|-----------|-----------|----------|-------------|-------|--------------------------|
| 1 | braces | npm | 3.0.3 | GHSA-vfj7-8cjw-p6xm | 7.5 High (A:H) | Stack-exhaustion DoS via deeply nested patterns | No — last-affected is 3.0.3 (current) |
| 2 | node-forge | npm | 1.4.0 | GHSA-86w9-cpqp-85rv | 7.5 High (I:H) | RSA PKCS#1 v1.5 verification accepts extra nested DigestAlgorithm elements (signature forgery) | No — last-affected is 1.4.0 (current) |
| 3 | nltk | PyPI | 3.10.3 | GHSA-8mgp-746c-j5xp | 6.8 Medium (AC:H, C:H) | Model-artifact APIs bypass path checks and touch files outside allowed roots | No — last-affected is 3.10.3 (current) |

Location in the tree:
- `frontend/pnpm-lock.yaml` — braces 3.0.3, node-forge 1.4.0
- `uv.lock` — nltk 3.10.3

**Triage notes (the value this adds over a raw `pnpm audit` / `pip-audit`):**
- The scanner's coarse severity flags all three "high"; by CVSS the nltk item is **Medium**, and its
  attack complexity is High. Prioritise braces and node-forge.
- **node-forge** is the one to watch: a signature-verification bypass is an integrity risk, not just
  availability. Check whether node-forge is in a reachable code path (TLS/crypto helpers) or only a
  transitive build-time dependency; the snapshot tells you it is present, not whether it is reachable.
- **braces** is a transitive micromatch/chokidar-family dependency; the DoS needs attacker-controlled
  glob patterns to reach it, which most apps do not expose.
- Because none has a fixed release yet, the realistic actions are: pin and monitor the advisory for a
  patched version, or apply the advisory's documented mitigation if one exists.

---

## 3. Secret-pattern surface (informational)

The engine flagged **20 hardcoded-credential patterns**. **Under the false-positive policy below, all 20
are non-issues**: every hit is in a test file, test fixture, CI workflow, or frontend example config —
never in shipped runtime code.

Representative locations: `tests/unit_tests/validator_tests/`, `tests/utils/api_routes/__init__.py`,
`tests/integration_tests/.../test_group_ai_providers.py`, `.github/workflows/test-backend.yml`,
`tests/e2e/docker/docker-compose.yml`, `frontend/nuxt.config.ts`.

**False-positive policy:** the secret surface is a pattern match, not a verified leak. Matches in paths
containing `test`, `tests`, `fixture`, `e2e`, `.github/workflows`, or example/config files are reported
as informational and should be treated as noise unless a value is a real production credential. No real
secret was identified in this repository.

---

## 4. Config lint (informational)

| # | File | Item | Severity |
|---|------|------|----------|
| 1 | `docker/Dockerfile` | No `USER` instruction — container runs as root | Low |

Add a non-root `USER` to the Dockerfile to reduce blast radius if the container is compromised.

---

## 5. Coverage and caveats

- **Covered:** resolved dependencies in `frontend/pnpm-lock.yaml` and `uv.lock` against OSV; deterministic
  config and secret-pattern rules across the tree.
- **Point-in-time:** valid only for commit `46497a9`. New advisories are published daily; re-run to refresh.
- **Not covered:** reachability analysis (is the vulnerable code path actually used?), source-level
  security review, runtime/behavioural testing, and any manifest the engine does not yet parse.
- **Reproducible:** `python3 scan.py <repo> --json out.json` with the open-source engine reproduces this.

---

## 6. Bottom line

mealie carries **three known dependency advisories (2 High, 1 Medium), none with a published fix yet**,
and **one low config hardening item**. The secret surface is clean (all matches are test/fixture noise).
The priority is monitoring node-forge and braces for patched releases and confirming node-forge
reachability.
