# Phase 2: Generate PRD

## Step 1: Analyze the Feature

Before writing, determine:

- **Which OSAC services** are affected (BMaaS, CaaS, VMaaS, MaaS, Enclave)?
- **Which personas** are affected? Default to giving EACH persona its own
  heading. Only combine two personas under one heading (e.g.,
  `### Tenant Admin / Tenant User`) when they have genuinely identical
  capabilities AND neither has any unique story. When in doubt, keep
  separate — the gold standard almost always separates them.
- **Persona ownership test:** Before marking any persona "Not affected," ask:
  does this persona currently perform the manual process this feature automates
  or replaces? If yes, they are a primary affected persona — write stories
  about what changes for them. Example: if a feature replaces a static pool
  maintained by Cloud Infrastructure Admins, CIA is a primary persona even if
  the feature is "CaaS" scoped.
- **What is the user pain?** State it from the user's perspective.
- **What is the scope boundary?** What's in, what's explicitly out?
- **Lifecycle decomposition:** For any resource, pool, or capacity being
  introduced, enumerate all lifecycle operations a user can perform: create,
  list/view, update/configure, scale up, scale down, delete. Each operation
  that's in scope needs an In Scope bullet. A feature that creates resources
  on-demand almost always implies scale-up/down and status visibility.
- **Async status visibility (mandatory check):** If ANY resource in the
  feature is created, provisioned, deployed, or modified asynchronously,
  status/progress visibility is a REQUIRED In Scope item and MUST have a
  user story. Ask: "Can the user see the current state and failure reasons?"
  Resources that trigger async operations: ComputeInstance, ClusterOrder,
  BareMetalInstance, storage volumes, CSI driver deployment, GPU passthrough
  setup. If the feature creates or provisions any of these, add a user story:
  "As a {persona}, I want to see the current status and any failure reasons
  for {resource} so that I can track progress and troubleshoot issues."
- **What are the dependencies?** Other features that must land first.
- **Dependency direction:** Does this feature enable something downstream, or
  depend on something upstream? A feature that exposes data does NOT depend
  on the downstream consumer — the consumer depends on it.

## Step 1.5: Extract Requirements from Jira (MANDATORY)

**You MUST complete this step before Step 2. Write the file below BEFORE
writing any PRD content. If you skip this step, the PRD will fail review.**

Re-read the Jira Feature description line by line. Write a file called
`/tmp/scope-contract.md` with exactly three sections:

```
## In Scope (from Jira)
- [quote or paraphrase each capability the Jira explicitly requests]

## Out of Scope (from Jira)
- [quote anything the Jira explicitly defers or excludes]

## Not Mentioned
- Everything not listed above. Do NOT add these to the PRD.
```

Rules for the extraction:
- Use the Jira's own words. Do not rephrase, expand, or infer.
- Acceptance criteria items → In Scope
- Demo steps → In Scope (the capability they demonstrate)
- "Future work", "separate ticket", "out of scope" in Jira → Out of Scope
- If the Jira does not mention something, it goes in "Not Mentioned"

**In Step 2, every In Scope bullet in the PRD must come from the
"In Scope (from Jira)" list above. Every Out of Scope bullet must come
from the "Out of Scope (from Jira)" list. Adding items from "Not
Mentioned" is a scope creep failure.**

## Step 2: Write the PRD

Follow the template structure. Use the section guidance from
`context/section-guidance.md` for detailed per-section instructions.

### Problem Statement
- Lead with user pain, not the system gap.
- 2-4 sentences. If the problem is clear in 2, stop there.
- State the cost of inaction.
- **Pain only — no solutions.** The Problem Statement describes what's broken
  and why it matters. Do NOT describe what the feature introduces, how it
  works, or what the solution model is — that belongs in In Scope. A sentence
  starting with "This feature introduces..." or "The X eliminates..." is a
  solution description, not a problem statement.

### In Scope
- Bullet list of user-observable capabilities.
- **No scope creep.** Every In Scope item must trace to the Jira input
  (description, acceptance criteria, or linked issues). Do NOT add
  capabilities the Jira ticket does not mention — even if they seem
  logical. If the Jira says "CRUD for X", scope is CRUD for X, not
  CRUD plus monitoring plus migration plus integration with Y.
- Do NOT restate user stories. In Scope adds boundary information that stories
  alone wouldn't convey ("works for both new and existing clusters" is a
  boundary; "tenants can create volumes" duplicates a story).
- If there is nothing beyond what user stories convey, keep to 2-4 bullets.
- Describe at a high level — not detailed requirements.
- **Status visibility:** For every resource or operation that is created or
  modified asynchronously, verify that status tracking is addressed. Can the
  user see the current state and any failure reasons? If a resource is
  provisioned on-demand, progress and error visibility is an implied In Scope
  requirement — don't omit it.
- **Billing, metering, and cost management** are platform-wide concerns.
  Default to Out of Scope unless billing IS the feature's primary purpose
  or the Jira Definition of Done explicitly requires billing integration.

### Out of Scope
- **Optional.** Only include what a reader would plausibly assume is included.
- Each item must pass the **boundary proximity test**: would a reviewer ask
  "is this included?" If not, the item is too distant.
- **Match the Jira source.** If the Jira ticket explicitly mentions something
  as out of scope or deferred, include it here. If the Jira mentions a related
  capability handled by a different ticket, that is an Out of Scope item with
  the responsible ticket noted.
- **Do NOT invert scope.** If the Jira ticket says something is in scope, do
  not move it to Out of Scope. If the Jira says something is deferred or out
  of scope, do not move it to In Scope. When unsure, check the Jira wording.
- For features involving shared physical infrastructure (bare metal hosts,
  GPUs, storage backends), explicitly address the tenant data boundary: what
  happens to data and configuration between assignments? If host sanitization,
  disk wipe, or credential rotation is not in scope for this feature, state it
  as Out of Scope with the responsible service noted (e.g., "Host sanitization
  between tenants — BMaaS responsibility").

### User Stories
- One story per distinct user goal. If a story has "and", split it.
- Ground in concrete artifacts and scenarios — name what users interact with.
- Do NOT write stories about platform behavior. "I want tenant isolation to
  be enforced" is not a user story — it's a platform invariant. "I want to
  view only my tenant's instances" is a user story.

### Assumptions
- Optional. Omit if no unverified assumptions exist.
- Do NOT put API contracts, interface specs, or design details here.

### Dependencies
- Optional. Omit if no external dependencies exist.
- Get the direction right. Name specific capabilities, not just Jira keys.

## Design Leakage — Apply These Tests to Every Statement

Reviewers reject PRDs that contain implementation details. Apply these smell
tests:

- **PM test:** Could a Product Manager verify this by using the product?
  If no → design leakage.
- **Swap test:** Would this statement change if the implementation changed?
  If yes → it's design.
- **Code test:** Does this name something only visible in source code?
  If yes → design leakage.

### Examples of Design Leakage (Do NOT Include)

| Design Leakage | User-Facing Alternative |
|----------------|------------------------|
| "Exponential backoff with 5 retries" | "The system retries failed operations" |
| "BareMetalInstanceReady condition" | "The instance status reflects readiness" |
| "InfraEnv per cluster" | omit — internal architecture |
| "Deep disk wipe and network state reset" | "Hosts are securely sanitized before reuse" |
| "MAC normalization to IEEE format" | omit — internal formatting |
| "Metadata propagated within 60 seconds" | omit unless Jira specifies an SLA |
| "Synchronization every 10 minutes" | omit — implementation timing |
| "The controller uses AAP to install" | "Storage is automatically available" |
| "Finalizer prevents deletion until..." | "Resources are cleaned up on deletion" |
| "AwaitingHardwareDiscovery status" | omit — internal condition name |

### Platform Vocabulary (Acceptable — NOT Design Leakage)

- OpenShift, Hosted Control Planes
- ClusterOrder, ComputeInstance, BareMetalInstance, Tenant
- VirtualNetwork, Subnet, SecurityGroup, PublicIP, StorageClass
- Keycloak, OPA, kubectl, Helm
- BMaaS, CaaS, VMaaS, MaaS, Enclave

## OSAC Personas

| Persona | Role |
|---------|------|
| **Cloud Provider Admin** | Tenant onboarding, quotas, global catalogs, super-user |
| **Cloud Infrastructure Admin** | Core infrastructure, network/storage integrations |
| **Tenant Admin** | Org config, users, IDP, org-scoped catalogs |
| **Tenant User** | Self-service provisioning, lifecycle management |

Each affected persona gets a `### {Persona}` heading with at least one user
story. Unaffected personas get "Not affected by this feature." in one line.

### Persona-Story Alignment Check

After writing all user stories, verify each story's capability matches the
persona's role:

| Capability type | Correct persona |
|----------------|-----------------|
| Tenant onboarding, quotas, global catalogs, cross-tenant visibility | Cloud Provider Admin |
| Infrastructure operations, hardware lifecycle, sanitization, network/storage backends | Cloud Infrastructure Admin |
| Org config, org-scoped catalogs, IDP, org users | Tenant Admin |
| Self-service provisioning, resource lifecycle, click-ops | Tenant User |

A sanitization or hardware lifecycle story under Cloud Provider Admin is a
misattribution — move it to Cloud Infrastructure Admin.

## Size Calibration

Match output depth to feature complexity. When in doubt, write less.

- **Simple feature** (1-2 capabilities): 15-40 lines
- **Medium feature** (3-5 capabilities): 40-70 lines
- **Complex feature** (5+ capabilities): 70-100 lines

One reviewer said of a 120-line PRD for a simple feature: "This doesn't need
to be 120 lines. Maybe 12? Shorter is better."

Consolidation rules:
- One user story per distinct user goal. Consolidate identical persona stories.
- Skip dimensions that don't apply — no "N/A" lines.
- Do not repeat information across sections.

## Patterns from Top-Scoring PRDs

**What 10/10 PRDs do:**
- Problem statement leads with user pain AND strategic motivation (security,
  compliance, adoption blocker — not just operational convenience)
- In Scope covers EVERY capability from the Jira Definition of Done — no omissions
- In Scope includes failure behavior (what happens when things go wrong)
- In Scope mentions tenant isolation for new resources
- Out of Scope contains items at the feature boundary — closely related but
  deferred. NOT distant, unrelated capabilities nobody would expect.
- When two personas have identical capabilities, they are combined under one
  heading (e.g., "### Tenant Admin / Tenant User") with a note explaining they
  share the same scope. **Combined persona test:** "Does the Tenant Admin have
  any capability in this feature that the Tenant User does not?" If no, combine.
- Out of Scope items pass the **boundary proximity test**: "Would a reviewer
  plausibly ask 'is this included?'" If not, the item is too distant.
- User stories cover 3-4 personas with concrete scenarios
- Dependencies name specific capabilities needed, not just Jira keys
- Assumptions are specific and verifiable
- Language is precise — no "appropriate", "efficient", "standard" without specifics

**What 5/10 PRDs get wrong:**
- Claim "all resources" but omit some
- Mix implementation details with requirements
- User stories are generic ("manage X") instead of specific ("create X with Y")
- Contradictory scope statements
- List completed work as in-scope

## What NOT to Do

- Do NOT ask clarifying questions — generate the best PRD from available info
- Do NOT add FR-N/NFR-N requirement IDs — OSAC uses user stories, not numbered FRs
- Do NOT add Risks, Acceptance Criteria, or Open Questions sections
- Do NOT prescribe implementation — "the controller uses AAP" is design leakage
- Do NOT use vague language — "handle edge cases appropriately" → name the edge cases
- Do NOT repeat the same information in Problem Statement and In Scope

## Source Traceability

Add `[Jira: {KEY}]` markers only when a requirement comes from a non-obvious
source (linked issue, comment). Add `[Assumption]` for requirements not directly
stated in the Jira source. Most statements trace to the primary feature — don't
tag every one.
