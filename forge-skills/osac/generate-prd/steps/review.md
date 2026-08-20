# Phase 3: Self-Review

**IMPORTANT: Do NOT output any review text, scores, or reasoning. Perform
this review silently — evaluate each criterion internally, note what needs
fixing, then proceed directly to Phase 4 (Revise) or Phase 5 (Output).
Your text output must contain ONLY the final PRD, never review commentary.**

Score your PRD draft against the 5-criterion rubric below. Be strict — first
drafts rarely merit 10/10. The average merged PRD scores 8-9/10 after human
review rounds.

## Scoring Process

For each criterion, evaluate internally and decide the score. Do NOT write
the scores or reasoning as text output.

### 1. WHAT — Clear user-facing need? (0-2)

Check:
- Does the PRD describe a new product capability (not just content/docs)?
- Are OSAC services identified (BMaaS, CaaS, VMaaS, MaaS, Enclave)?
- Are affected personas identified with per-persona user stories?
- Each affected persona must have at least one `As a <persona>...` story.
  Mentioning a persona in prose without a story does not count.

Score:
- **0** = Vague, system internals, no personas, or no per-persona user stories
- **1** = Partially clear but mixed with implementation or missing personas
- **2** = Clear, specific, user-observable. Each affected persona has stories.

Calibration:
- W=0: "Implement CSI driver installation via AAP playbook" — system action, no user need
- W=1: "Tenants can manage secrets" — right direction but generic, no use cases
- W=2: "Tenant users can retrieve cluster kubeconfig and admin password via
  the secrets API. Tenant admins can store OIDC client secrets for IDP
  integration." — names concrete artifacts

### 2. WHY — Business justification? (0-2)

Check:
- Is there a clear problem statement with user pain?
- Is the cost of inaction described?
- Is there concrete evidence (not just "users need this")?

Score:
- **0** = No justification or circular reasoning
- **1** = Generic justification, plausible but no evidence
- **2** = Concrete justification with pain, impact, or strategic tie

Calibration:
- Y=1: "Tenants cannot run stateful workloads without manual storage
  configuration." — gap described but no impact
- Y=2: "Tenants cannot run stateful workloads until someone manually
  configures storage, and there is no visibility into whether storage is
  available. This blocks CaaS adoption." — pain + consequence + tie

### 3. User-Facing Focus — Free from design leakage? (0-2)

Check:
- Does the PRD name controllers, reconcilers, finalizers, playbooks?
- Does it describe internal conditions or reconciliation logic?
- Does it specify CRD field names, SLA numbers not from Jira, or cleanup
  mechanics?
- Platform vocabulary (ClusterOrder, ComputeInstance, etc.) is acceptable.

Smell tests:
- Could a PM verify this by using the product?
- Would this change if the implementation changed?
- Does this name something only visible in code?

Score:
- **0** = Reads like a design document
- **1** = Mostly user-focused but some design leakage
- **2** = Only user-observable outcomes

Calibration:
- UF=0: "The storage controller places a finalizer on each ClusterOrder.
  On deletion, it triggers osac-delete-tenant-cluster-storage." — finalizers,
  controller names, playbook names
- UF=1: "Storage is automatically provisioned on CaaS clusters. The
  controller uses AAP to install the CSI driver." — good outcome, but
  "controller uses AAP" is implementation
- UF=2: "When a CaaS cluster is provisioned and ready, persistent storage
  is automatically available without manual configuration." — pure outcome

### 4. Right-Sized — Focused scope? (0-2)

Check:
- How many independent capabilities are described?
- Could each ship on its own and provide value?
- Capabilities that require each other are one feature.

Score:
- **0** = Bundles 3+ independent capabilities
- **1** = Bundles 1-2 separable capabilities
- **2** = Focused — capabilities require each other

### 5. Testability — Verifiable requirements? (0-2)

Check:
- Can each user story be verified by a PM using the product?
- Are there vague terms ("appropriate", "efficient") without specifics?
- Are there requirements describing system internals?

Score:
- **0** = Requirements describe activities or internals
- **1** = Some testable, some vague or internal
- **2** = Every requirement PM-verifiable

## Pass/Fail

- **PASS**: Total >= 7/10 AND no zeros on any criterion
- **FAIL**: Total < 7 OR any zero (automatic fail regardless of total)

## Inline Deterministic Checks

Also verify these concrete checks against your draft:

1. **Section check:** Count `## ` headings. Must have exactly: Problem
   Statement, In Scope, Out of Scope, User Stories, Assumptions (optional),
   Dependencies (optional). No other `## ` headings allowed (no Risks,
   Acceptance Criteria, Terminology, Milestone Scoping).

2. **Persona check:** All four names must appear in the User Stories section:
   Cloud Provider Admin, Cloud Infrastructure Admin, Tenant Admin, Tenant User
   (each with a story or "Not affected" note).

3. **Leakage check:** Search for these terms (case-insensitive). Any match
   is a failure:
   - reconciler, reconciliation, finalizer, playbook, env var, AAP job,
     CRD field, osac-operator, osac-aap, ansible role
   - "controller" (unless in "Hosted Control Planes")

4. **Length check:** Count non-blank lines. Must be 15-120. Target is 40-80.

5. **Persona ownership check:** For each persona marked "Not affected," verify
   the feature does not automate or replace a process they currently perform.
   If it does, they need user stories — mark this as a review failure.

6. **Persona-story alignment check:** For each user story, verify the
   capability matches the persona's role definition. Infrastructure operations
   under Cloud Provider Admin, or tenant management under Cloud Infrastructure
   Admin, is a misattribution.

7. **Problem Statement solution check:** The Problem Statement must not contain
   sentences describing what the feature introduces or how it works. Search for
   "introduces", "eliminates", "provides", "enables" used to describe the
   feature itself (not the user pain). Any match is a failure.

8. **Async status check:** Scan In Scope for ANY of these async triggers:
   provisioning, deployment, creation of ComputeInstance, ClusterOrder,
   BareMetalInstance, storage volumes, CSI driver, GPU passthrough, or any
   phrase like "automatically available", "on-demand", "when ready". If ANY
   trigger is found, verify that BOTH conditions are met:
   (a) In Scope includes a status/progress visibility bullet, AND
   (b) User Stories includes a story about seeing current state and failure
   reasons. Missing either is a failure.

9. **Jira completeness check:** Re-read the original Jira input line by line.
   For each concrete requirement, acceptance criterion, capability, or
   explicit scope statement in the Jira description:
   (a) Verify it appears somewhere in the PRD (In Scope, User Stories, or
   Out of Scope with justification).
   (b) If the Jira says something is out of scope or deferred, verify it
   is in your Out of Scope section — not accidentally in In Scope.
   List any gaps. Missing a Jira requirement is a failure.

10. **Scope creep check:** For each In Scope bullet, verify it traces to
    something in the Jira input. If an In Scope item was NOT mentioned in
    the Jira ticket (description, acceptance criteria, or linked issues),
    it is scope creep — remove it or move it to Out of Scope. Do NOT
    invent capabilities the Jira does not request.

11. **Persona separation check:** Verify that Tenant Admin and Tenant User
    have SEPARATE headings in User Stories unless they have genuinely
    identical capabilities in this feature. Same for Cloud Provider Admin
    and Cloud Infrastructure Admin. If any two personas are combined under
    one heading, verify that neither has ANY unique capability — if one
    does, split them into separate headings.

## Verdict

If all checks pass → proceed to Phase 5 (Output).
If any check fails → proceed to Phase 4 (Revise). Note which criteria scored
low and which deterministic checks failed.
