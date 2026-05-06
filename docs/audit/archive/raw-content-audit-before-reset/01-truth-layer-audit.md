# Executive summary
The truth layer has a clear backbone and is mostly usable by Codex agents, but it is not fully implementation-safe yet. The hierarchy is explicit in `AGENTS.md` and reinforced by `docs/00-start-here.md`, and the three core truth artifacts (`decision_log`, `canonical_system_brief`, `open_questions_register`) are generally aligned. However, there are structural blocker-level issues: stale truth-path references (`support/*`), and unresolved/deferred decisions that can block first implementation wave if touched. Language policy (English system terms vs Indonesian UI labels/entities) is not explicitly codified in the truth layer and remains an OPEN QUESTION.

# Source-of-truth hierarchy
## Current hierarchy (explicit)
1. `docs/truth/01-decision_log.md`
2. `docs/truth/02-canonical_system_brief.md`
3. `docs/product/prd.md`
4. `docs/contracts/schema_mapping.md`
5. `docs/contracts/business_contracts.md`
6. `docs/contracts/query_contracts.md`
7. `docs/contracts/integration_contract_pack.md`
8. `docs/modules/*.md`
9. `docs/execution/*.md`

Source: `AGENTS.md` section `## Source of truth order`.

## Agent usability check
- Usable: yes.
- Evidence:
  - `AGENTS.md` has explicit ordered list and conflict rule (`Check decision_log first`, then update `open_questions_register`).
  - `docs/00-start-here.md` provides “Read this first” sequence, prioritizing `AGENTS.md` and truth docs.
- Structural risk:
  - `docs/00-start-here.md` sequence places `canonical_system_brief` before `decision_log`, while `AGENTS.md` puts `decision_log` first. Not a hard conflict, but it can cause different reading order by agents.

# Blocker decisions
## B-01 (blocker)
- Topic: Truth path integrity for decision references
- Evidence:
  - `docs/truth/03-open_questions_register.md` rows repeatedly reference `truth/01-decision_log.md` and related `support/*` paths.
  - Actual truth files are under `docs/truth/*`.
- Why blocker:
  - Agents may fail to resolve references or rely on wrong paths, reducing trust in “resolved” status traceability.
- Decision needed: yes
- OPEN QUESTION:
  - Should all truth references be normalized to `docs/truth/*` immediately before implementation starts?

## B-02 (blocker when in scope)
- Topic: Deferred product/UX decisions still open
- Evidence:
  - `docs/truth/03-open_questions_register.md`:
    - `PQ-004` status `deferred`
    - `UX-003` status `deferred`
- Why blocker:
  - If first implementation wave touches period duplication or admin report drill-down, behavior is undefined at truth level.
- Decision needed: yes (conditional)
- OPEN QUESTION:
  - Are these deferred items formally out-of-scope for wave 1 acceptance, with explicit guardrails in execution plans?

# Ambiguous truth sources
## A-01
- Ambiguity: Reading order mismatch between onboarding doc and authority order.
- Evidence:
  - `AGENTS.md`: decision log first.
  - `docs/00-start-here.md`: canonical brief first, then decision log.
- Implementation risk:
  - Agents may interpret summary before decision overrides and miss conflict resolution precedence.

## A-02
- Ambiguity: Resolved status traceability points to stale locations.
- Evidence:
  - `docs/truth/03-open_questions_register.md` “Dokumen Terkait” includes `truth/01-decision_log.md` in many resolved items.
- Implementation risk:
  - “Resolved” claims are harder to verify automatically, increasing risk of silent drift.

## A-03
- Ambiguity: `02-canonical_system_brief.md` claims final integrated truth, but conflict rule lives in `AGENTS.md` and decision authority in `01-decision_log.md`.
- Evidence:
  - `docs/truth/02-canonical_system_brief.md` intro states “Source of Truth” summary role.
  - `AGENTS.md` explicitly says check `decision_log` first for conflicts.
- Implementation risk:
  - New contributors may treat brief as final for all details and ignore latest decision entries.

# Language consistency issues
## L-01
- Issue: Truth layer does not explicitly codify bilingual policy (English for system terms, Indonesian for UI labels/entities shown to users).
- Evidence:
  - `AGENTS.md`, `docs/00-start-here.md`, `docs/truth/01-decision_log.md`, `docs/truth/02-canonical_system_brief.md`, and `docs/truth/03-open_questions_register.md` do not define this language rule as a formal standard.
- Risk:
  - Inconsistent naming across API/schema/auth docs vs frontend labels.
- Decision needed: yes
- OPEN QUESTION:
  - Where should language policy be canonically defined (truth doc vs style guide), and what enforcement rule applies?

## L-02
- Issue: Mixed Indonesian/English is present but uncontrolled.
- Evidence examples:
  - System term present: `Internal API Routes`, `RLS`, `status_pesanan` (technical).
  - UI-facing entities/labels present in Indonesian: `Periode`, `Setoran`, `Riwayat Koreksi`.
- Risk:
  - Lack of explicit mapping between system term and UI label may cause contract/UI drift.

# Recommended changes
1. Add a short “Truth Consumption Protocol” section in `docs/truth/` (or `AGENTS.md`) that states:
- mandatory read order,
- conflict resolution rule,
- precedence of decision log over summaries.

2. Normalize all stale references from `support/*` to `docs/truth/*` and correct contract path mentions to current locations.

3. Add explicit scope gating for deferred truth items (`PQ-004`, `UX-003`) in execution entry criteria:
- either marked “out of wave 1 scope”,
- or promoted to `open` with owner/date.

4. Introduce a canonical language policy note in truth layer:
- System term: English (architecture, API, database, auth, permission, validation, workflow, implementation).
- UI label/frontend-facing entity: Indonesian.
- Require explicit mapping when terms differ.

5. Add a compact glossary mapping table at truth layer boundary to prevent naming drift.
Example format:
- System term: `order_status`
- UI label: `Status Pesanan`

# Open questions
1. Should `docs/00-start-here.md` read order be aligned exactly with `AGENTS.md` (decision log first), or is current sequence intentional?
2. Are `PQ-004` and `UX-003` formally excluded from wave 1 implementation scope?
3. Should stale truth references (`support/*`) be treated as release-blocking for documentation sign-off?
4. Where is the canonical location for bilingual language policy and term mapping enforcement?
