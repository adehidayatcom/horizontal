# 05 - Frontend and UI Layer Audit

## Frontend readiness summary

Overall readiness: **partial**.

The frontend layer is substantially documented and mostly aligned with the selected implementation stack, but there are unresolved naming drift, route-model alignment gaps, and contract coverage ambiguities that should be resolved before coding.

### Findings

#### Finding ID: FUI-001
- Severity: high
- Affected documents:
  - `docs/frontend/frontend_architecture.md` (sections: **Tech stack and constraints**, **Data and integration boundaries**)
  - `docs/ui/modernize_shell_guide.md` (section: **Shell and navigation structure**)
- Summary: Core stack alignment is explicit and mostly implementation-ready.
- Evidence from documents:
  - Frontend architecture explicitly names Next.js App Router, TypeScript, MUI/Modernize, SWR, Formik/Yup, and Supabase interaction boundaries via internal API route layer.
  - Modernize shell guide defines shell composition and MUI-based layout conventions.
- Why it matters for implementation:
  - Confirms baseline architectural direction and prevents accidental divergence (e.g., direct Supabase client calls for operational business data).
- Recommended action:
  - Keep as canonical for frontend implementation baseline; link this stack section from all page-level UI specs.
- Decision needed: no

#### Finding ID: FUI-002
- Severity: high
- Affected documents:
  - `docs/frontend/frontend_architecture.md` (section: **Routing topology**)
  - `docs/frontend/navigation_and_period_setup_ui.md` (sections: **Periode setup flow**, **Step routes and navigation**)
- Summary: Route hierarchy is documented but not fully synchronized across architecture and period setup documents.
- Evidence from documents:
  - Architecture route map emphasizes high-level route groups.
  - Period setup document describes specific setup/checklist/closing flows with route-level behavior that is not consistently mirrored in the architecture map.
- Why it matters for implementation:
  - Mismatch can lead to duplicated route trees, incorrect layout nesting, and broken breadcrumb/guard behavior in App Router.
- Recommended action:
  - Publish one canonical route matrix (segment, layout owner, guard, actor, module ownership) and reference it from both documents.
- Decision needed: yes

#### Finding ID: FUI-003
- Severity: medium
- Affected documents:
  - `docs/frontend/frontend_component_contracts.md` (section: **Component naming and DTO mapping**)
  - `docs/ui/admin_dashboard_uiux.md` (sections: **Dashboard widgets**, **Workflow action panels**)
- Summary: Some UI interaction specs are detailed, but explicit backend contract references are inconsistent per action.
- Evidence from documents:
  - Component contracts define integration pattern, but several dashboard actions are described from UX perspective without direct contract IDs/endpoints or RPC mapping.
- Why it matters for implementation:
  - UI engineers may infer endpoints/queries, increasing risk of contract drift and inconsistent error handling.
- Recommended action:
  - Add a mandatory “backend contract reference” field per actionable UI element (query contract ID, command/API contract ID, expected status model).
- Decision needed: yes

#### Finding ID: FUI-004
- Severity: medium
- Affected documents:
  - `docs/frontend/frontend_architecture.md` (section: **Reference and dependency links**)
  - `docs/frontend/component_patterns.md` (section: **Cross-doc references**)
- Summary: Some document references appear stale/non-canonical (e.g., support path references).
- Evidence from documents:
  - References include links that do not match current truth-layer path conventions.
- Why it matters for implementation:
  - Agents and developers can consume outdated decisions and miss current conflict resolution protocol.
- Recommended action:
  - Normalize cross-references to truth-layer canonical paths (`docs/truth/*`) and remove stale aliases.
- Decision needed: no

## Route/page coverage

### Findings

#### Finding ID: FUI-005
- Severity: high
- Affected documents:
  - `docs/frontend/frontend_architecture.md` (section: **App Router structure**)
  - `docs/ui/reseller_uiux.md` (sections: **Reseller dashboard and workflow pages**)
  - `docs/ui/admin_dashboard_uiux.md` (sections: **Admin navigation and dashboard**) 
- Summary: Route coverage is broad for admin and reseller areas, but page-to-workflow ownership is not always explicit.
- Evidence from documents:
  - Route groups and UI pages are documented.
  - Several workflows span multiple pages without explicit owner module or boundary per route segment.
- Why it matters for implementation:
  - In App Router, unclear ownership causes duplicated fetching logic, inconsistent cache invalidation, and cross-module coupling.
- Recommended action:
  - Create route ownership table: route segment, owning module, required actor, required period state, primary read model, primary write contracts.
- Decision needed: yes

#### Finding ID: FUI-006
- Severity: medium
- Affected documents:
  - `docs/frontend/navigation_and_period_setup_ui.md` (section: **Period setup navigation state**) 
  - `docs/ui/modernize_shell_guide.md` (section: **Sidebar/menu behavior**) 
- Summary: Navigation behavior is described, but canonical rule for period-aware menu gating is fragmented.
- Evidence from documents:
  - Period setup doc defines gating semantics.
  - Shell guide defines generic menu behavior without always restating period dependency model.
- Why it matters for implementation:
  - Users may access pages in invalid lifecycle states unless guards are uniformly implemented at route and UI levels.
- Recommended action:
  - Define one guard contract for menu visibility + route entry checks tied to period state machine.
- Decision needed: yes

## UI action to backend contract mapping

### Findings

#### Finding ID: FUI-007
- Severity: blocker
- Affected documents:
  - `docs/ui/admin_dashboard_uiux.md` (sections: **Operational actions**, **Review and verification interactions**)
  - `docs/frontend/frontend_component_contracts.md` (section: **Action handlers and API boundary**) 
  - Related contract baseline: `docs/contracts/*` (as referenced by frontend docs)
- Summary: Not every high-impact UI action has an explicit backend contract reference in the frontend/UI layer docs.
- Evidence from documents:
  - UI specs include approval/review/bulk workflow actions.
  - Frontend component contract explains pattern but does not consistently bind each action to a named contract artifact.
- Why it matters for implementation:
  - This is a direct blocker for safe implementation because developers may invent commands, payloads, or transitions.
- Recommended action:
  - Add action-contract matrix per screen: `ui_action`, `actor`, `precondition`, `contract_id`, `payload schema ref`, `success state`, `failure states`.
- Decision needed: yes

#### Finding ID: FUI-008
- Severity: high
- Affected documents:
  - `docs/ui/reseller_uiux.md` (sections: **Setoran/payment interactions**, **Order/status interactions**)
  - `docs/frontend/frontend_component_contracts.md` (section: **Mutation and validation patterns**)
- Summary: Validation and error mapping for mutation actions are partially specified and not always standardized.
- Evidence from documents:
  - UI flows describe user actions and expected outcomes.
  - Uniform mapping between backend validation errors and Indonesian UI feedback is not complete per action.
- Why it matters for implementation:
  - Inconsistent validation display can break user trust and cause support load.
- Recommended action:
  - Define one error taxonomy mapping: contract error code -> Indonesian UI notification/form error target.
- Decision needed: yes

## Role-based UI consistency

### Findings

#### Finding ID: FUI-009
- Severity: high
- Affected documents:
  - `docs/ui/admin_dashboard_uiux.md` (section: **Admin-only capabilities**)
  - `docs/ui/reseller_uiux.md` (section: **Reseller capabilities**) 
  - `docs/frontend/frontend_architecture.md` (section: **Auth/authorization boundary**) 
  - Related permission truth: `docs/contracts/*` RLS/permission artifacts (by reference)
- Summary: Actor separation is clearly intended, but UI visibility rules are not fully traced to permission/RLS contract identifiers.
- Evidence from documents:
  - Admin and reseller screens are separated at UX level.
  - Explicit mapping of each conditional UI element to permission keys/RLS policy references is incomplete.
- Why it matters for implementation:
  - UI may expose actions that backend rejects, or hide actions that backend allows, causing role inconsistency.
- Recommended action:
  - Add role visibility matrix: component/action -> role(s) -> permission key -> RLS/API contract reference.
- Decision needed: yes

## UI language consistency

### Findings

#### Finding ID: FUI-010
- Severity: medium
- Affected documents:
  - `docs/frontend/component_patterns.md` (section: **Naming patterns**) 
  - `docs/ui/admin_dashboard_uiux.md` (multiple UI sections)
  - `docs/ui/reseller_uiux.md` (multiple UI sections)
- Summary: Language rule intent is clear, but technical terms and UI labels occasionally drift.
- Evidence from documents:
  - Docs generally preserve Indonesian for user-facing labels.
  - Some mixed usage appears where system concepts and UI wording are not explicitly paired (internal status vs displayed status text).
- Why it matters for implementation:
  - Without explicit pairing, engineers may leak internal English enums to UI or localize inconsistently.
- Recommended action:
  - Add bilingual mapping table per domain term: `system_term (English)` <-> `UI label (Indonesian)` and enforce it in component contracts.
- Decision needed: yes

#### Finding ID: FUI-011
- Severity: low
- Affected documents:
  - `docs/ui/admin_dashboard_uiux.md` (section duplication around setoran progress)
- Summary: Duplicate/overlapping sections may cause future divergence in label and state wording.
- Evidence from documents:
  - Similar dashboard subtopic appears more than once with overlapping intent.
- Why it matters for implementation:
  - Duplicate spec text increases maintenance risk and inconsistent UI copy.
- Recommended action:
  - Consolidate duplicate sections into one canonical block with explicit state/label table.
- Decision needed: no

## Missing UI states

### Findings

#### Finding ID: FUI-012
- Severity: high
- Affected documents:
  - `docs/ui/admin_dashboard_uiux.md` (sections: **Data panels and action flows**) 
  - `docs/ui/reseller_uiux.md` (sections: **Operational pages**) 
  - `docs/frontend/frontend_component_contracts.md` (section: **State handling conventions**)
- Summary: Loading/empty/error states are present in many places, but “success state” behavior is not consistently formalized per action.
- Evidence from documents:
  - State handling appears in narrative form.
  - Some actions define failure/empty/loading but do not define post-success UI behavior (toast, redirect, table refresh, status chip transition).
- Why it matters for implementation:
  - Inconsistent success handling affects usability and can produce stale UI in SWR cache flows.
- Recommended action:
  - Require state matrix per action: `loading`, `empty`, `error`, `success`, `offline` (if applicable), with trigger and UI response.
- Decision needed: yes

#### Finding ID: FUI-013
- Severity: medium
- Affected documents:
  - `docs/frontend/navigation_and_period_setup_ui.md` (section: **Setup progression and recovery**) 
  - `docs/ui/modernize_shell_guide.md` (section: **Global feedback patterns**)
- Summary: Recovery states for partially completed period setup and interrupted flows are not fully standardized in shell-level guidance.
- Evidence from documents:
  - Period flow defines gating steps.
  - Shell-level fallback UX for interrupted/partially valid states is not always explicit.
- Why it matters for implementation:
  - Users may get stuck without clear recovery CTA across route transitions.
- Recommended action:
  - Define a global recovery UX contract for blocked period states (banner, CTA target, reason code).
- Decision needed: yes

## Blockers before coding

### Blockers

1. **FUI-007 (blocker)**: Missing explicit UI-action to backend-contract mapping for all critical operational actions.
2. **FUI-002 (high, blocking in practice)**: Route hierarchy not fully synchronized between architecture and period setup flow docs.
3. **FUI-005 (high, blocking in practice)**: Route/page ownership and workflow boundaries are incomplete for App Router module implementation.
4. **FUI-009 (high, security-risk blocker)**: Role-based UI visibility not fully traced to permission/RLS contract references.

### Open questions

1. **OPEN QUESTION OQ-FUI-01**
   - Should `docs/frontend/frontend_architecture.md` become the sole canonical route map, with `navigation_and_period_setup_ui.md` only adding behavioral overlays, or should a separate centralized route matrix be introduced under truth/contracts?
2. **OPEN QUESTION OQ-FUI-02**
   - What is the canonical artifact for per-action backend mapping in frontend docs: embed in each UI doc or maintain one shared action-contract registry?
3. **OPEN QUESTION OQ-FUI-03**
   - Which permission key naming convention is canonical for UI conditional rendering when mapped to RLS/API policies?
4. **OPEN QUESTION OQ-FUI-04**
   - What is the final canonical bilingual mapping source for `internal status (English)` to `display status (Indonesian)` across modules?

## Recommended changes

1. Publish a single **route ownership matrix** with App Router segment ownership, guards, actor, and period-state dependencies.
2. Publish a **UI action-contract matrix** covering all mutating and approval actions.
3. Publish a **role visibility matrix** tied to permission/RLS references.
4. Publish a **state behavior matrix** (loading/empty/error/success/offline) per key action and screen.
5. Publish a **system-term to UI-label glossary** (English system terms, Indonesian UI labels) and reference it from frontend/component docs.
6. Normalize stale cross-doc references to current truth-layer paths.
