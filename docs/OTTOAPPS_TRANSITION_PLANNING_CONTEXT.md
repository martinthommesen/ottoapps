# OttoApps transition planning context

> **Status:** Authoritative planning brief for the OttoApps transition  
> **Date:** 2026-08-10  
> **Target repository:** `martinthommesen/ottoapps`  
> **Source repositories:** `martinthommesen/smalt` and `martinthommesen/otto`  
> **Intended audience:** Coding agents and maintainers preparing the transition plan  
> **Authorization boundary:** This document authorizes investigation and planning. It does **not** authorize implementation, publication, repository archival, data deletion, package deprecation, or production cutover without a separately approved execution plan.

## 1. Executive mandate

OttoApps is the clean-sheet successor to Smalt and the exclusive home for Atlassian and ServiceNow MCP functionality currently spread across `martinthommesen/smalt` and `martinthommesen/otto`.

The end state is a public, acquisition-quality product family with exactly two public MCP server identities:

- `oa-atlassian`
- `oa-servicenow`

These two servers must retain every useful capability and safety property from Smalt, absorb every Atlassian and ServiceNow integration responsibility currently implemented inside Otto, and then grow beyond both source projects into full replacements and improvements over any official Atlassian or ServiceNow MCP servers.

The architecture must be:

- harness-agnostic;
- model-agnostic;
- hosting-agnostic;
- authentication-provider-agnostic;
- transport-flexible within the MCP standard;
- framework-independent below a thin FastMCP 4 adapter;
- safe for local, enterprise, and remotely hosted use;
- suitable for single-tenant and, later, multi-tenant deployment;
- auditable, least-privileged, and secure by default.

OttoApps is **not** merely a rename of Smalt, a collection of compatibility wrappers, or a proxy in front of official vendor MCP servers. It is a new product with its own architecture, lifecycle, release train, security model, and product identity.

## 2. The non-negotiable repository boundary

This is the most important requirement in the transition.

### 2.1 OttoApps becomes the exclusive owner of MCP and vendor-integration functionality

After cutover, `martinthommesen/ottoapps` must own and implement all of the following for Atlassian and ServiceNow:

- MCP server construction and lifecycle;
- MCP tool, resource, prompt, completion, and capability registration;
- MCP transport handling, including local and remotely hosted modes;
- MCP protocol negotiation and compatibility handling;
- MCP client implementation used by first-party consumers such as Otto;
- inbound client authentication and authorization;
- caller identity, role, scope, and tenant resolution;
- vendor API clients and all direct vendor network communication;
- Atlassian REST, GraphQL, or other supported API access;
- ServiceNow REST, Table API, IRE, scripted API, scoped-app, or other supported API access;
- vendor authentication, token acquisition, refresh, storage, rotation, and credential selection;
- instance, site, origin, account, project, space, table, class, and other tenant or allowlist enforcement;
- vendor query construction and validation, including JQL, CQL, ServiceNow encoded queries, structured queries, and pagination;
- vendor request shaping, rate limiting, retry policy, timeout handling, response-size enforcement, and error mapping;
- vendor payload parsing and normalization, including ADF, Confluence storage formats, ServiceNow records, attachments, and rich text;
- tool schema discovery, schema fingerprints, compatibility manifests, and registration-parity verification;
- write gates, previews, confirmations, one-shot confirmation spending, mutation ledgers, reconciliation, and audit trails;
- source-side redaction, content classification, provenance, attachment handling, and bounded result shaping;
- source-specific health checks, diagnostics, live tests, replay fixtures, and sandbox verification;
- compatibility logic for legacy Smalt contracts during the migration period;
- any first-party OttoApps client library or generated client contract used by Otto or other consumers.

This ownership rule applies even when code is not literally an MCP server. A ServiceNow REST adapter, a CQL builder, an Atlassian payload parser, or an OAuth refresher is part of the MCP product's vendor-integration implementation and therefore belongs in OttoApps.

### 2.2 Only the agent itself remains in Otto

After cutover, `martinthommesen/otto` must contain only the concerns that make Otto an agent. Those may include:

- Webex ingress, egress, delivery, and user-experience behavior;
- conversation state and memory;
- agent orchestration and turn coordination;
- model-provider selection and model invocation;
- model egress policy;
- agent-level tool selection and planning;
- agent-level grounding, evidence assembly, citation rendering, and answer composition;
- agent-specific authorization, rate limiting, and abuse controls;
- agent-level telemetry and operational audit;
- generic source or tool-provider interfaces that contain no MCP or vendor implementation;
- minimal composition code that injects an OttoApps-owned client implementation into the agent.

Otto must not own or directly implement Atlassian, ServiceNow, or MCP behavior after the transition.

### 2.3 Forbidden content in Otto after cutover

The transition is incomplete while any of the following remains in the Otto repository, except in historical migration documentation or isolated tests proving removal:

- a FastMCP dependency or FastMCP server/client implementation;
- an MCP transport, session, negotiation, tool-discovery, or schema-validation implementation;
- direct calls to Atlassian, Confluence, Jira, ServiceNow, or their official/community MCP servers;
- a hosted Atlassian Rovo MCP binding;
- a self-hosted `mcp-atlassian` binding or fallback;
- a ServiceNow REST or Table API client;
- JQL, CQL, ServiceNow encoded-query, or vendor-specific query-building logic;
- vendor response parsers or vendor-specific record normalization;
- Atlassian or ServiceNow credentials, token refreshers, instance URLs, site URLs, API tokens, OAuth client secrets, or vendor scopes;
- vendor-specific health probes, contract discovery, capability fingerprints, retries, pagination, or response-size enforcement;
- direct fallback paths that bypass OttoApps;
- duplicated copies of OttoApps tool schemas or source contracts maintained by hand;
- source mutation gates, confirmation logic, source audit state, or source-side attachment handling.

A direct fallback from Otto to a vendor API or official MCP server is specifically prohibited. Rollback must occur by deploying a previous OttoApps version, not by retaining permanent bypass code inside Otto.

### 2.4 Permitted integration seam between Otto and OttoApps

The recommended end state is:

```text
Webex user
    │
    ▼
Otto agent
    │  generic agent/source interface
    ▼
OttoApps-owned client package or external client adapter
    │  standard MCP
    ├──────────────► oa-atlassian
    └──────────────► oa-servicenow
                         │
                         ▼
                  vendor APIs only
```

The concrete MCP client implementation used by Otto should be built, tested, versioned, and released from the OttoApps repository, for example as a small `ottoapps-client` distribution or equivalent generated client artifact. Otto may define a narrow structural protocol that the client satisfies, but Otto must not implement MCP transport or vendor semantics itself.

The composition root in Otto may:

- read OttoApps endpoint addresses;
- read a caller credential used to authenticate Otto to OttoApps;
- construct or inject the OttoApps-owned client;
- expose agent-level façades when those façades encode conversational behavior rather than vendor behavior.

The composition root must not receive upstream Atlassian or ServiceNow credentials.

### 2.5 Dependency and trust direction

The dependency direction must remain one-way:

```text
Otto ──uses──► OttoApps client/protocol ──calls──► OttoApps servers ──call──► vendors
```

The following are prohibited:

- OttoApps importing code from Otto;
- OttoApps depending on Otto's agent models or Webex runtime;
- Otto and OttoApps sharing a mutable source tree;
- Otto becoming the owner of generated OttoApps contracts;
- vendor credentials flowing through Otto;
- OttoApps calling back into Otto to complete a vendor operation.

Cross-repository contracts must be versioned and published by OttoApps. They must not rely on two repositories changing atomically.

## 3. Product identity and public surface

The working product identity is:

| Concern | Name |
| --- | --- |
| Repository and product family | `ottoapps` / OttoApps |
| Atlassian MCP server | `oa-atlassian` |
| ServiceNow MCP server | `oa-servicenow` |
| Python namespace | `ottoapps` or another explicitly approved namespace |
| Optional administration CLI | `oa` |
| Optional first-party client package | `ottoapps-client` or an explicitly approved equivalent |

The exact package, PyPI, container, and registry names must be checked for availability before publication. Repository naming does not by itself guarantee package-name or trademark availability.

There must be exactly two public MCP server products. Internally mounted providers, domain modules, role profiles, replicas, or restricted deployments do not count as additional products. An administrative CLI, health endpoint, migration utility, or client library is allowed, but a third audit MCP server is not.

The current Smalt `mcp-audit` capability should become one or both of:

- appropriately authorized administrative tools inside `oa-atlassian` and `oa-servicenow`;
- non-MCP administration through the `oa` CLI or an operational API.

It must not survive as a third public MCP endpoint.

## 4. Source repositories and transition inputs

Before producing an execution plan, the coding agent must record exact commit SHAs for all three repositories and inspect repository-specific instructions, open pull requests, open issues, generated artifacts, release metadata, and CI requirements. Relative statements such as “current main” are not sufficient in the final plan.

### 4.1 Smalt: legacy product and executable specification

Repository: `martinthommesen/smalt`

Smalt is the principal source of existing MCP functionality and safety behavior. Its current architecture includes a private shared runtime and a family of separate MCP server packages covering:

- ServiceNow lookup;
- ServiceNow ITSM;
- ServiceNow generic table reads;
- ServiceNow generic table writes;
- ServiceNow approvals;
- ServiceNow CMDB writes through IRE;
- ServiceNow knowledge authoring and feedback;
- ServiceNow control-app audit access;
- Confluence core operations;
- Confluence permission administration;
- constrained Confluence API access;
- Jira;
- local audit querying.

Smalt also contains important non-tool behavior that must be treated as product functionality, including:

- fail-closed environment parsing;
- one-origin and credential-boundary enforcement;
- caller-selected allowlists;
- origin-pinned bounded HTTP;
- proxy and private-CA support;
- stable typed tool errors;
- response redaction;
- write previews and confirmation gates;
- persistent one-shot confirmation spending;
- audit-before-mutation semantics;
- mutation ledgers and uncertain-outcome handling;
- identity pinning and drift detection;
- generated capability manifests and registration parity;
- packed-tarball replay tests;
- live sandbox verification;
- an optional ServiceNow scoped control application.

The generated Smalt capability catalog is a starting parity contract, not the final OttoApps product design. Every declared tool, resource, prompt, environment contract, error code, policy rule, and observable behavior must be inventoried. The planner must distinguish:

1. behavior that must remain byte-for-byte or semantically compatible;
2. behavior that should be preserved but improved behind a compatible contract;
3. behavior that should be deliberately deprecated;
4. missing vendor functionality that OttoApps must add.

Smalt PR #486, the closed and unmerged FastMCP 4 prototype, is a useful design and test reference. It must not be blindly cherry-picked. Its architecture, assumptions, dependency versions, security gaps, and prototype shortcuts must be revalidated against the approved OttoApps design.

### 4.2 Otto: agent repository containing integration responsibilities that must be extracted

Repository: `martinthommesen/otto`

Otto currently mixes the agent with Atlassian and ServiceNow integration behavior. The planning agent must inspect, at minimum, the current equivalents of:

- `src/otto/integrations/confluence/**`;
- `src/otto/integrations/servicenow/**`;
- shared integration bases and evidence contracts;
- vendor-specific credential code under `src/otto/credentials/**`;
- vendor and MCP configuration in the main config model;
- application composition in `src/otto/app.py`;
- model-facing source tools in `src/otto/runtime/agent.py`;
- contract verification, health probes, and startup preflight;
- tests, fixtures, deployment configuration, environment examples, and documentation;
- direct and transitive MCP dependencies in `pyproject.toml` and lock files.

Known current concerns include a hosted Atlassian MCP integration, a self-hosted Atlassian fallback, an Otto-owned ServiceNow REST adapter, vendor query builders, payload parsing, source contract verification, and model-facing source tools. All implementation responsibility for those integrations must move to OttoApps.

The planner must classify each Otto artifact as one of:

- **Move:** functionality belongs in OttoApps and should be ported or reimplemented there;
- **Replace:** Otto should consume a new OttoApps contract instead of retaining the implementation;
- **Retain:** the artifact is genuinely agent-specific and contains no MCP or vendor behavior;
- **Delete after cutover:** temporary or superseded code with no continuing owner;
- **Split:** agent behavior remains in Otto while source behavior moves to OttoApps.

The default classification for anything vendor-specific is **Move** or **Replace**, not Retain.

### 4.3 OttoApps: clean successor

Repository: `martinthommesen/ottoapps`

OttoApps should begin with a deliberate architecture rather than importing either source repository wholesale. It must preserve valuable behavior without inheriting accidental package boundaries, historical naming, direct harness coupling, or obsolete deployment assumptions.

The new repository should become the sole source of truth for:

- OttoApps product architecture;
- server and client code;
- capability manifests;
- vendor schemas and adapters;
- security policy and threat models;
- deployment artifacts;
- compatibility and migration tooling;
- release notes and support policy;
- official-server comparison matrices;
- all future Atlassian and ServiceNow MCP development.

## 5. Definition of success

Success should be measured in four explicit levels.

### Level 0: source inventory complete

- Every Smalt capability and invariant is recorded.
- Every Atlassian, ServiceNow, and MCP concern in Otto is recorded.
- Exact source SHAs and provenance are pinned.
- Every open issue and pull request is classified by destination.
- No functionality is assumed to be irrelevant merely because it is not a registered tool.

### Level 1: source parity

- OttoApps has semantic parity with all supported Smalt capabilities.
- OttoApps absorbs all vendor-integration behavior required by Otto.
- Differential and contract tests prove parity or record an approved deviation.
- Existing source behavior remains available through the two new server identities.

### Level 2: safe production cutover

- Otto uses only OttoApps for Atlassian and ServiceNow.
- Otto contains no MCP implementation, no vendor integration, no vendor credentials, and no bypass path.
- Operational state, confirmations, audit, attachments, and rollback are production-ready.
- Local and remote deployment modes pass their support matrices.
- Smalt can be frozen without removing a capability from users.

### Level 3: official-server replacement and superiority

- OttoApps matches the useful supported surface of the current official Atlassian and ServiceNow MCP offerings, measured against dated and cited comparison matrices.
- OttoApps exceeds those offerings in safety, deployment independence, policy control, auditability, extensibility, agent usability, and enterprise operability.
- The comparison is maintained continuously as vendor products change.

Source parity and safe cutover are prerequisites. They must not be confused with complete vendor-platform coverage.

## 6. Architectural principles

### 6.1 FastMCP is an adapter, not the domain architecture

FastMCP 4 is the selected MCP delivery framework, but business logic must not live primarily in decorators, request objects, or framework-specific component classes.

The dependency direction should be:

```text
FastMCP 4 adapter
        │
        ▼
application operations and capability modules
        │
        ▼
policy, identity, audit, confirmation, state, and result contracts
        │
        ▼
vendor clients and transport abstractions
```

The domain and application layers must be testable without starting an MCP server. Replacing FastMCP, adding a CLI, adding a REST façade, or embedding OttoApps in another host should not require rewriting vendor operations.

FastMCP-specific code should be limited to concerns such as:

- server construction;
- component registration;
- MCP input/output adaptation;
- request-context extraction;
- transport startup;
- framework authorization hooks;
- capability negotiation and optional framework extensions.

### 6.2 Two servers, many internal modules

The two public servers should compose focused internal modules.

`oa-servicenow` should ultimately include modules equivalent to and broader than:

- identity and lookup;
- ITSM;
- generic table read;
- generic table write;
- approvals;
- CMDB and IRE;
- knowledge;
- control and authoritative audit;
- attachments and exports;
- instance metadata and schema discovery;
- future licensed platform domains.

`oa-atlassian` should ultimately include modules equivalent to and broader than:

- Jira;
- Confluence;
- Confluence permissions;
- constrained generic Atlassian API access;
- Jira Service Management;
- Assets;
- Compass;
- Bitbucket, when in approved scope;
- identity, administration, and site metadata;
- future supported Atlassian products.

Internal modules may have independent policies, credentials, tests, and deployment profiles, but they must mount into one of the two public server identities without creating a new product endpoint.

### 6.3 Vendor-complete, not vendor-shaped

OttoApps should not expose a one-to-one wrapper for every REST endpoint. It should provide:

- high-quality task-level tools that perform deterministic validation and orchestration;
- lower-level tools where agents genuinely need them;
- guarded generic escape hatches for custom tables, custom fields, or APIs;
- stable, agent-friendly schemas independent of incidental vendor response shapes.

A high-level operation may call several vendor APIs internally, provided its behavior is explicit, bounded, auditable, and testable.

### 6.4 Standard MCP first

Every essential capability must work over standard MCP without requiring a proprietary harness or FastMCP-only extension.

Optional enhancements may use negotiated capabilities, but there must be a safe fallback for clients that lack them. In particular:

- essential writes must not depend exclusively on interactive elicitation support;
- confirmation can use a portable preview/challenge/execute contract;
- structured output should include a text-compatible fallback where required;
- resources and prompts must not be the only way to access essential data if common clients cannot consume them;
- tool search or role-filtered discovery may optimize large catalogs, but authorized tools must remain directly callable by name.

### 6.5 Fail closed

Missing, contradictory, or ambiguous configuration must fail startup or refuse the operation. It must never silently widen access, select a default tenant, infer a production write acknowledgment, or fall back to a more privileged credential.

### 6.6 Preserve security semantics before porting mutations

No mutation module should be ported to production use until the shared security runtime is complete enough to preserve:

- policy classification;
- preview behavior;
- explicit arming;
- caller-bound confirmation;
- one-shot spend;
- audit-before-outbound-write;
- uncertain-outcome recording;
- reconciliation;
- safe retry and idempotency rules.

## 7. Recommended repository and package shape

The exact structure should be finalized through an ADR, but the plan should preserve these boundaries. A Python workspace could resemble:

```text
ottoapps/
├── apps/
│   ├── oa-atlassian/
│   └── oa-servicenow/
├── packages/
│   ├── core/
│   ├── policy/
│   ├── state/
│   ├── audit/
│   ├── auth/
│   ├── mcp-fastmcp/
│   ├── client/
│   ├── atlassian/
│   │   ├── common/
│   │   ├── jira/
│   │   ├── confluence/
│   │   └── ...
│   └── servicenow/
│       ├── common/
│       ├── lookup/
│       ├── itsm/
│       ├── cmdb/
│       └── ...
├── contracts/
│   ├── capability-manifest/
│   ├── schemas/
│   └── compatibility/
├── tests/
│   ├── unit/
│   ├── contract/
│   ├── differential/
│   ├── integration/
│   ├── security/
│   ├── compatibility/
│   └── live/
├── deployments/
├── docs/
└── tools/
```

A single Python distribution with two entry points may be acceptable for operators, but the first-party client should be separable enough that Otto does not need to install server-only dependencies. Possible approaches include:

- a workspace containing separate `ottoapps-server` and `ottoapps-client` distributions;
- one namespace with independently packaged subprojects;
- generated client code and schemas published as a small artifact.

The planning agent must compare these choices using installation size, dependency isolation, release coupling, typing, compatibility, and supply-chain risk. It must not solve package boundaries by reintroducing thirteen public servers.

## 8. Capability contracts and tool-surface strategy

### 8.1 Build an authoritative capability manifest

OttoApps needs a machine-readable manifest that is the source of truth for:

- server ownership;
- module ownership;
- tool, resource, and prompt names;
- descriptions and annotations;
- input and output schemas;
- access classification;
- required scopes and roles;
- tenant and allowlist dimensions;
- confirmation requirements;
- audit classification;
- stability and deprecation status;
- source provenance;
- compatibility aliases;
- test coverage and live-verification status.

Generated documentation, client bindings, registration-parity tests, role profiles, and comparison reports should derive from this manifest where practical.

### 8.2 Preserve names during source-parity migration

Existing `snow_*`, `confluence_*`, and `jira_*` tool names should be treated as compatibility contracts during the first migration unless a documented security or correctness defect requires a break.

The new server names do not require every tool to gain an `oa_` prefix. The server boundary already identifies the product. Avoid a simultaneous architecture rewrite and wholesale tool rename.

Any later naming convergence should provide:

- a versioned policy;
- aliases where safe;
- generated migration documentation;
- a deprecation window;
- contract tests for both old and new names;
- an explicit removal release.

### 8.3 Keep the visible catalog manageable

Two servers may contain hundreds of capabilities. Catalog size must be controlled through standards-compatible mechanisms such as:

- caller authorization and component visibility;
- role profiles;
- server-side tags and groups;
- tool-search capabilities with direct-call fallback;
- selective mounting at deployment time;
- predictable pagination and schema summaries.

Do not solve catalog size by recreating one process per domain. Do not expose unauthorized tool names and then rely only on execution-time denial if the framework can hide them safely.

### 8.4 Stable result and error contracts

Results should be structured, bounded, and explicit. A common envelope should be considered for:

- schema version;
- operation and source identity;
- request and correlation identifiers;
- tenant or connection identity without secrets;
- status;
- structured data;
- pagination;
- warnings and partial-result indicators;
- provenance;
- policy or audit receipts;
- uncertainty and reconciliation state.

Error codes must be stable enough for agents and clients to distinguish:

- invalid input;
- policy refusal;
- authentication or authorization failure;
- not found;
- conflict;
- rate limit;
- upstream unavailable;
- timeout;
- response too large;
- uncertain mutation outcome;
- internal invariant violation.

MCP protocol errors should be reserved for protocol failures. Ordinary tool failures should be typed tool results without leaking credentials, sensitive record values, or raw upstream bodies.

## 9. OttoApps client contract and Otto integration

### 9.1 Client ownership

The concrete client used by Otto must live in OttoApps. It should own:

- MCP connection lifecycle;
- transport selection;
- protocol negotiation;
- authentication to OttoApps;
- capability discovery;
- contract and schema-version checks;
- request correlation;
- timeout and cancellation behavior;
- safe retry behavior for reads;
- result-envelope decoding;
- typed error conversion;
- compatibility handling during rolling upgrades.

It must not own Otto's conversation or model behavior.

### 9.2 Otto's retained interface

Otto may retain a small generic interface, for example conceptually:

```python
class EnterpriseAppsGateway(Protocol):
    async def call(self, operation: str, arguments: Mapping[str, object]) -> ToolResult: ...
```

The real interface should be strongly typed where practical, but it must not import vendor SDKs or implement MCP. Structural typing can allow the OttoApps client to satisfy Otto's interface without OttoApps importing Otto.

Agent-level façades may remain in Otto only when they add agent behavior such as:

- combining multiple authorized results into a conversational workflow;
- applying model-egress policy;
- rendering citations;
- selecting a source based on user intent;
- limiting context passed to a model.

Vendor field names, query syntax, pagination, payload parsing, and source policy remain OttoApps responsibilities.

### 9.3 No duplicated source schema

Otto must not hand-maintain copies of OttoApps schemas. The plan should choose one of:

- runtime MCP discovery plus pinned schema fingerprints;
- generated typed bindings published by OttoApps;
- a versioned contract artifact consumed in Otto tests and builds.

The chosen design must support independent deployment and rolling upgrades. A mismatch must fail clearly rather than silently dropping fields or widening access.

### 9.4 Network-level enforcement

The production design should make the repository boundary enforceable in infrastructure:

- Otto should have no network route or egress allowlist entry for Atlassian or ServiceNow origins;
- only OttoApps should hold vendor credentials;
- Otto should reach only its model provider, Webex, OttoApps endpoints, and other explicitly agent-owned services;
- deployment tests should prove that Otto cannot bypass OttoApps;
- OttoApps should pin outbound vendor origins and reject caller-supplied arbitrary URLs.

## 10. Authentication, identity, authorization, and tenancy

### 10.1 Keep the two authentication planes separate

OttoApps has two independent identity planes:

1. **Caller to OttoApps:** who is invoking an MCP capability and what may that caller see or do?
2. **OttoApps to vendor:** which Atlassian or ServiceNow identity and credential performs the upstream action?

A caller token must never be mistaken for an upstream credential, and an upstream credential must never grant caller authorization by itself.

Audit records should preserve both identities when available:

- remote caller subject;
- tenant;
- OttoApps server and capability;
- upstream acting identity;
- policy decision;
- operation outcome.

### 10.2 Inbound authentication must be pluggable

The architecture should support adapters for appropriate deployment modes, such as:

- local stdio process trust;
- OAuth/OIDC bearer tokens;
- JWT validation against configured issuers;
- machine-to-machine client credentials;
- mutual TLS;
- trusted reverse-proxy identity assertions with strict network and signature controls;
- enterprise identity providers such as Entra ID, Okta, Keycloak, Auth0, Cloudflare Access, or equivalents without making any one provider architectural.

Provider-specific behavior must live behind a stable authentication interface. Secure issuer, audience, algorithm, clock-skew, and key-rotation validation are mandatory.

### 10.3 Authorization must govern discovery and execution

Authorization should combine RBAC and context-aware policy where useful. It should be able to constrain:

- server;
- tenant;
- module;
- tool;
- access class;
- project, space, table, class, or other resource allowlist;
- read, write, destructive, administrative, or export behavior;
- attachment size and type;
- production versus non-production origins;
- time-limited elevation or step-up requirements.

The same server-side policy must govern both discovery and execution. Hiding a tool without enforcing it at execution is insufficient; enforcing at execution while exposing every unauthorized schema is also undesirable.

### 10.4 Upstream credentials

OttoApps should support vendor-appropriate credential types without hard-coding one global credential. The design should allow:

- a shared credential for simple deployments;
- per-module credential overrides;
- per-tenant credentials;
- delegated user identity where safely supported;
- service identities for automation;
- scoped credentials aligned with least privilege.

Credential lookup must be based on authenticated tenant and policy context, never on a raw caller-provided secret name or arbitrary origin.

### 10.5 Tenant model

The first production milestone may remain explicitly single-tenant, but the internal model should avoid assumptions that make later multi-tenancy unsafe.

A tenant context should bind at least:

- allowed vendor origin or site;
- credential reference;
- caller authorization policy;
- resource allowlists;
- state namespace;
- audit namespace;
- attachment namespace;
- feature and capability profile.

Tenant selection must be authenticated and authorized. Empty tenant or allowlist sets must fail closed rather than mean “all.”

## 11. Security and policy invariants

### 11.1 Origin and request safety

All outbound requests must:

- remain pinned to an approved HTTPS origin;
- reject arbitrary absolute URLs and unsafe redirect targets;
- resist SSRF, DNS rebinding, and credential forwarding;
- enforce timeouts and cancellation;
- enforce response-byte ceilings before buffering;
- bound pagination and total records;
- validate content types where relevant;
- redact headers and secrets from errors and telemetry;
- respect explicitly configured proxy and private-CA behavior;
- apply rate-limit and retry policy without unsafe mutation retries.

### 11.2 Read, write, destructive, and administrative classes

Every capability must be classified. Policy should distinguish at least:

- read;
- write;
- destructive;
- permission or access administration;
- credential or identity administration;
- audit access;
- bulk export;
- generic API escape hatch.

A single global “enable writes” flag must not arm every module. ServiceNow generic table writes, CMDB writes, approvals, knowledge writes, Jira writes, Confluence writes, and permission administration should have independent policy gates and confirmation audiences.

### 11.3 Portable confirmation protocol

The baseline confirmation flow should work with any compliant MCP client:

1. A preview or preparation operation validates the request and returns a bounded summary, payload digest, expiry, and confirmation challenge.
2. Execution requires the challenge plus an explicit confirmation value or signed approval.
3. The confirmation binds caller, tenant, origin, module, tool, normalized payload digest, policy version, and expiry.
4. The spend is persisted atomically and may succeed only once.
5. The accepted mutation is audited before the outbound request.
6. The outcome is recorded as succeeded, failed, or uncertain.

Interactive framework features may improve UX but cannot be the sole security mechanism.

### 11.4 Mutation uncertainty and reconciliation

Mutations must not be blindly retried after timeouts, connection loss, or undecodable responses. The system must:

- record an uncertain outcome;
- preserve request identity and idempotency evidence;
- expose reconciliation tools or operator workflows;
- retry only when the vendor and operation provide a proven idempotency contract;
- distinguish a known failure from an unknown result.

### 11.5 Audit is not ordinary logging

Security audit must be durable, queryable, tamper-evident where practical, and separate from debug logs. It should record only what is necessary to prove who attempted what, under which policy, against which tenant, and with what outcome.

Do not record secrets, full titles, bodies, comments, attachments, or changed field values by default. Any content-bearing audit mode must be explicit, justified, access-controlled, and documented.

### 11.6 Untrusted content

Content retrieved from Jira, Confluence, ServiceNow, attachments, comments, knowledge articles, and custom fields is untrusted. OttoApps must:

- never execute instructions found in source content;
- clearly separate source data from server instructions and policy;
- avoid reflecting source content into logs or errors;
- mark or structure untrusted content for downstream agents;
- apply size, type, and encoding bounds;
- sanitize active content where previews or exports require it.

## 12. State, scaling, and attachments

### 12.1 State interfaces

Stateful security behavior must use explicit interfaces for:

- confirmation spend;
- mutation ledger;
- audit records;
- OAuth state or refresh coordination where applicable;
- distributed locks;
- attachment handles;
- schema or metadata cache;
- optional rate-limit coordination.

A local backend such as SQLite may support local and single-node use. A transactional shared backend such as PostgreSQL should be the reference for horizontally scaled deployments. Backend choice must not alter security semantics.

### 12.2 Horizontal scaling

Stateless MCP request handling does not make the product stateless. Before supporting multiple replicas for writes, prove:

- atomic one-shot confirmation spending;
- consistent tenant policy;
- durable audit-before-write ordering;
- mutation reconciliation across replicas;
- safe OAuth refresh coordination;
- attachment-handle consistency;
- no reliance on process-local identity or caches for authorization.

### 12.3 Attachments and large content

Local filesystem paths from Smalt are not sufficient for remote callers. The design should support:

- bounded inline content for small files;
- short-lived opaque handles for large files;
- pluggable object storage;
- tenant-isolated namespaces;
- size, content-type, and extension allowlists;
- malware scanning hooks;
- digest verification;
- short expiry and one-time or scoped access;
- safe export names;
- no arbitrary server-side file access.

Local deployments may provide explicitly rooted filesystem adapters, but remote and local behavior must share the same policy contract.

## 13. Hosting and transport independence

OttoApps should support at least:

- stdio for local tools and desktop clients;
- Streamable HTTP for remote hosting;
- stateless request handling where compatible with required security state;
- container deployment;
- bare-process deployment;
- Kubernetes or equivalent orchestration;
- reverse proxies, corporate proxies, and private certificate authorities;
- graceful shutdown, readiness, liveness, and startup preflight.

The core must not require a particular cloud, ingress provider, secret manager, database service, object store, or observability vendor.

Deployment profiles should document supported combinations, for example:

- local single-user stdio with local state;
- single-tenant remote server with OIDC and PostgreSQL;
- enterprise private deployment behind an identity-aware proxy;
- horizontally scaled remote deployment with shared state and object storage.

Multiple replicas or role-restricted instances of `oa-atlassian` or `oa-servicenow` are allowed. Creating new product-specific server binaries for each module is not.

## 14. Observability and operations

The product should expose operational signals without leaking source data.

### 14.1 Telemetry

Plan for:

- structured logs to stderr or the host logging system;
- OpenTelemetry-compatible traces and metrics;
- request and correlation identifiers;
- server, tenant, module, tool, and access-class dimensions;
- latency, timeout, rate-limit, retry, and upstream-status metrics;
- state-backend and attachment-backend health;
- audit write failures as hard safety events;
- no high-cardinality source values or sensitive record contents.

### 14.2 Health and readiness

Readiness should prove required configuration and critical dependencies without performing unsafe mutations. Health behavior should distinguish:

- process alive;
- configuration valid;
- authentication material available;
- state backend reachable;
- vendor identity verified;
- individual modules degraded;
- writes disabled by policy rather than broken.

### 14.3 Operational documentation

OttoApps should ship runbooks for:

- credential rotation;
- issuer and signing-key rotation;
- database migration and rollback;
- uncertain mutation reconciliation;
- audit export and retention;
- attachment cleanup;
- rate-limit incidents;
- vendor API deprecation;
- emergency write disablement;
- disaster recovery;
- release rollback.

## 15. Public-repository, licensing, and provenance requirements

OttoApps is public while source material may come from private repositories. The planning agent must create a provenance ledger before copying code.

### 15.1 Smalt material

Smalt currently uses Apache-2.0. Code ported from Smalt should preserve required notices, attribution, copyright, and license obligations. Generated artifacts and copied tests must be classified as well.

### 15.2 Otto material

Otto is not assumed to grant public reuse merely because the same owner controls both repositories. Before copying proprietary Otto code into public OttoApps, obtain and record an explicit licensing decision. Where no explicit relicensing decision exists, use Otto as a behavioral reference and reimplement the functionality with synthetic tests rather than copying source text.

### 15.3 Confidentiality review

Before any public commit, scan for:

- real instance or site URLs;
- customer, employer, or tenant names;
- user identifiers and email addresses;
- API tokens, client IDs, secrets, certificates, or key material;
- real Jira, Confluence, or ServiceNow payloads;
- proprietary schema names or custom fields;
- internal network names;
- private issue links or screenshots;
- generated files that embed environment data.

Public fixtures must be synthetic and demonstrably non-sensitive.

### 15.4 Product and trademark posture

Until any formal vendor relationship exists, documentation must make clear that OttoApps is independent and is not endorsed by Atlassian or ServiceNow. Product comparison may use vendor names descriptively but must not imply ownership or official status.

### 15.5 Supply chain

The plan should include:

- dependency pinning and update policy;
- license and vulnerability scanning;
- SBOM generation;
- signed commits and releases where supported;
- provenance attestations;
- reproducible or verifiable builds;
- minimal runtime images;
- secret scanning;
- branch protection and required CI;
- release artifact verification.

## 16. Detailed migration inventory

The final plan must produce a row-by-row inventory, not just a directory-level summary.

### 16.1 Smalt mapping

At minimum, map these source areas into target owners:

| Smalt source | OttoApps destination concept |
| --- | --- |
| `packages/internal-runtime` | framework-independent core, HTTP, policy, auth, state, audit, errors, redaction |
| ServiceNow MCP packages | internal `oa-servicenow` capability modules |
| Confluence and Jira MCP packages | internal `oa-atlassian` capability modules |
| `mcp-audit` | authorized admin tools and/or `oa` CLI, not a third server |
| generated capability catalog | OttoApps parity manifest and generated docs |
| role recipes and preflight | deployment profiles, validation, and client configuration tooling |
| packed replay fixtures | differential and package-level compatibility suite |
| live sandbox tooling | OttoApps opt-in live verification |
| ServiceNow scoped control app | OttoApps-owned instance component with an explicit compatibility/migration ADR |
| local mutation and confirmation stores | storage interfaces with local and distributed implementations |

The ServiceNow scoped application's scope identity may be a real compatibility boundary. Do not assume it can or should simply be renamed. The plan must determine whether to preserve the legacy scope, create a successor application, support both temporarily, or provide a migration mechanism.

### 16.2 Otto mapping

At minimum, map these concerns:

| Otto concern | Required end state |
| --- | --- |
| Confluence hosted-MCP adapter | moved/replaced by `oa-atlassian`; removed from Otto |
| self-hosted Atlassian fallback | removed; no bypass path |
| ServiceNow REST adapter | moved/reimplemented in `oa-servicenow`; removed from Otto |
| vendor query builders | moved to relevant OttoApps modules |
| vendor response parsers | moved to OttoApps |
| source contract discovery/fingerprints | owned and published by OttoApps |
| vendor credentials/config | moved to OttoApps deployment; removed from Otto |
| direct MCP client implementation | moved to OttoApps client package |
| model-facing source tools | classify: move vendor semantics; retain only agent-specific façade behavior |
| evidence envelope | define a versioned OttoApps output contract consumed by Otto grounding |
| live source checks | move to OttoApps |
| vendor-related tests and fixtures | move or reimplement with public-safe provenance |
| Webex, conversation, model, orchestration | remain in Otto |

### 16.3 Open issue and pull-request migration

The planner must inspect every open issue and pull request in Smalt and Otto. Each item must be classified as:

- migrate to OttoApps;
- remain in source repository;
- split into linked issues;
- superseded by an OttoApps architectural decision;
- close only after replacement evidence exists.

New OttoApps issues should preserve source links, rationale, acceptance criteria, dependencies, and any security severity. Do not silently abandon source findings during repository transition.

## 17. Testing and verification strategy

### 17.1 Contract and registration tests

Prove that:

- the manifest matches real registrations;
- no undeclared capability is exposed;
- no declared required capability is missing;
- authorization affects both discovery and execution;
- input and output schemas remain stable or are explicitly versioned;
- generated client bindings match server contracts;
- server names and entry points are packaged correctly.

### 17.2 Differential tests against Smalt

For every migrated Smalt capability, compare:

- accepted inputs;
- rejected inputs;
- policy decisions;
- normalized outbound requests;
- pagination;
- response projection;
- typed errors;
- redaction;
- retry behavior;
- audit events;
- confirmation and mutation-ledger behavior;
- uncertain outcomes.

Differences must be classified and approved, not hidden by loose assertions.

### 17.3 Otto extraction tests

Add automated architecture gates proving that Otto no longer contains or performs forbidden behavior. Candidate checks include:

- forbidden imports and dependency rules;
- no `fastmcp` or direct MCP implementation in Otto;
- no Atlassian or ServiceNow SDK/client dependency;
- no vendor origin environment variables;
- no vendor token configuration;
- no direct vendor domains in source or deployment configuration;
- network tests showing vendor egress is denied;
- runtime tests showing all source operations traverse the OttoApps client;
- contract mismatch tests;
- absence of legacy fallback activation paths.

A successful functional test alone is insufficient if the old code still exists as a dormant fallback.

### 17.4 Security tests

Include negative and adversarial coverage for:

- SSRF and redirect handling;
- origin confusion;
- tenant confusion and empty allowlists;
- caller/upstream identity confusion;
- JWT issuer, audience, algorithm, key rotation, and expiry validation;
- permission visibility and direct-call bypass;
- confirmation replay, tampering, cross-tenant use, expiry, and race conditions;
- audit failure before mutation;
- uncertain mutation outcomes;
- response and attachment size limits;
- path traversal;
- malicious filenames and content types;
- prompt injection in source content;
- secret and content leakage through errors, logs, traces, and receipts;
- rate-limit storms and retry amplification;
- concurrent OAuth refresh;
- distributed state races.

### 17.5 Client and harness compatibility

Maintain a dated compatibility matrix across representative standards-compliant clients and transports. Tests should focus on protocol capabilities rather than model brands. Verify:

- stdio framing and stdout purity;
- Streamable HTTP behavior;
- sessionful and stateless modes where supported;
- cancellation;
- structured content and text fallback;
- tool discovery and direct invocation;
- resource and prompt behavior;
- large-catalog handling;
- authentication challenges;
- confirmation fallback without proprietary UI features.

### 17.6 Live verification

Live tests must be opt-in, isolated, bounded, and run only against approved sandboxes. Mutation tests must create identifiable test-owned fixtures, clean them safely, and record any cleanup failure. Live credentials must never be available to untrusted pull requests.

### 17.7 Packaging and release tests

Test the artifacts users actually install:

- wheels and source distributions;
- both server entry points;
- the first-party client package;
- container images;
- offline or hermetic install where feasible;
- dependency extras;
- generated manifests;
- upgrade and rollback paths;
- database migrations;
- provenance and signatures.

## 18. Migration phases and mandatory gates

The planning agent may refine phase boundaries, but it must preserve the following dependency logic.

### Phase 0: evidence snapshot and planning

Scope:

- pin exact SHAs;
- read repository instructions;
- inventory files, capabilities, tests, issues, releases, and deployment paths;
- record licenses and provenance;
- produce the capability and file-migration matrices;
- identify unknowns and contradictions;
- draft ADRs and the implementation issue graph.

Exit gate:

- the owner approves the transition plan;
- no implementation is required for this gate;
- every source capability has a destination or an explicit approved disposition.

### Phase 1: OttoApps repository foundation

Scope:

- license and attribution decision;
- security policy and contribution rules;
- Python/runtime/toolchain decision;
- workspace and package boundaries;
- two empty but correctly packaged server entry points;
- framework-independent core interfaces;
- manifest generator and registration-parity skeleton;
- CI, formatting, linting, typing, tests, supply-chain checks;
- ADR process and generated documentation.

Exit gate:

- clean install and package smoke tests pass;
- exactly two public MCP server identities exist;
- no vendor mutation capability exists yet;
- dependency boundaries are enforced automatically.

### Phase 2: parity harness and shared read-only runtime

Scope:

- import or recreate the Smalt capability manifest;
- build differential replay infrastructure;
- implement common HTTP, errors, redaction, identity, allowlists, pagination, and auth abstractions;
- define the OttoApps result/provenance contract;
- define the first-party client compatibility contract.

Exit gate:

- the harness can report implemented, missing, and intentionally deviating capabilities;
- common runtime security tests pass;
- no secret or real payload enters the public repository.

### Phase 3: `oa-servicenow` read-only parity

Scope:

- identity and lookup;
- ITSM reads;
- table reads and structured query support;
- CMDB reads needed by existing behavior;
- knowledge reads;
- approvals reads;
- metadata, schema, and diagnostics;
- Otto-required ServiceNow behavior currently implemented in Otto.

Exit gate:

- read-only ServiceNow parity is green against replay and approved live tests;
- acting identity and origin pinning are proven;
- result contracts satisfy Otto grounding needs;
- missing source behavior is zero or explicitly approved.

### Phase 4: `oa-atlassian` read-only parity

Scope:

- Confluence content, search, history, hierarchy, labels, attachments, and permissions reads;
- Jira issue, comment, link, transition, search, and metadata reads;
- constrained generic Atlassian API reads;
- hosted/self-hosted behavior needed by Otto, implemented directly against approved vendor APIs rather than by preserving Otto fallbacks;
- ADF and Confluence payload normalization.

Exit gate:

- read-only Atlassian parity is green;
- site, project, and space boundaries are proven;
- Otto-required source behavior is available from `oa-atlassian`;
- no runtime dependency on an official or community MCP server is required.

Phases 3 and 4 may proceed in parallel only after the shared contracts are stable enough to avoid duplicate runtimes.

### Phase 5: remote hosting, inbound auth, and tenant-aware policy

Scope:

- standard remote MCP transport;
- pluggable inbound authentication;
- authorization-driven discovery and execution;
- single-tenant production profile;
- tenant-aware internal context;
- PostgreSQL-backed shared state foundation;
- health, readiness, metrics, and deployment artifacts;
- caller identity in audit.

Exit gate:

- local and remote compatibility matrices pass;
- authorization bypass tests pass;
- multiple replicas are supported for reads;
- no caller can choose an arbitrary vendor origin or credential.

### Phase 6: shared security state and attachments

Scope:

- confirmation-spend store;
- mutation ledger;
- durable audit;
- distributed locking and refresh coordination;
- object-storage and local attachment adapters;
- reconciliation workflows;
- retention and recovery procedures.

Exit gate:

- race, replay, cross-tenant, failure-ordering, backup, and restore tests pass;
- audit failure prevents outbound mutation;
- attachment controls pass adversarial tests;
- write modules remain disabled until this gate is complete.

### Phase 7: mutation parity, module by module

Scope:

- ServiceNow ITSM and generic table writes;
- ServiceNow approvals;
- ServiceNow CMDB/IRE writes;
- ServiceNow knowledge writes;
- Confluence content, attachment, classification, and permission writes;
- Jira comments, links, transitions, and other existing writes;
- any other source mutation in the approved parity manifest.

Each module must have its own issue, gate, live fixtures, confirmation audience, audit classification, and rollback evidence.

Exit gate:

- every mutation passes preview, confirmation, audit, uncertainty, reconciliation, differential, and live tests;
- no module is armed by another module's gate;
- production writes remain opt-in.

### Phase 8: Otto client integration and hard cutover

Scope:

- publish and pin the OttoApps-owned client contract;
- integrate it into Otto through the generic agent boundary;
- migrate agent grounding to OttoApps result/provenance contracts;
- run read-only shadow comparisons where safe;
- cut source calls over to OttoApps;
- remove all Otto vendor and MCP implementation, dependencies, config, credentials, tests, documentation, and fallbacks;
- enforce network egress boundaries;
- update operations and deployment manifests.

Exit gate:

- every Otto Atlassian and ServiceNow operation traverses OttoApps;
- Otto has no vendor credential and cannot reach vendor origins;
- architecture and forbidden-dependency tests pass;
- rollback uses a previous OttoApps deployment, not legacy Otto code;
- Otto's agent behavior and citations pass regression tests.

### Phase 9: canonical release and Smalt archival

Scope:

- final Smalt-to-OttoApps compatibility report;
- migration guide;
- final supported Smalt release or tag;
- clear “superseded by OttoApps” README and release notice;
- package deprecation plan where applicable;
- preserve historical issues and source references;
- make OttoApps canonical;
- archive Smalt only after approval and completed gates.

Exit gate:

- no supported user or Otto runtime depends on Smalt;
- all security-relevant open findings have an OttoApps disposition;
- archive and package-deprecation actions are explicitly approved;
- the Smalt repository remains available as read-only history.

### Phase 10: official-server parity and product superiority

Scope:

- maintain dated feature matrices against official offerings;
- close platform-domain gaps;
- add high-level deterministic workflows;
- expand enterprise policy, delegated identity, and multi-tenant capabilities;
- publish deployment and support matrices;
- measure reliability and usability.

This phase is ongoing product development, not a reason to delay the clean source-parity transition indefinitely.

## 19. Cutover and rollback rules

### 19.1 No permanent dual implementation

Temporary compatibility adapters may exist on isolated migration branches or behind short-lived feature flags, but the final architecture must not maintain two vendor implementations.

### 19.2 Read shadowing before write shadowing

Read-only differential shadowing may compare OttoApps with legacy behavior using bounded, redacted results. Do not shadow writes against production systems.

### 19.3 Rollback by artifact

Rollback should deploy:

- the previous OttoApps server version;
- the previous compatible OttoApps client version;
- a reversible database migration where required.

Do not retain direct vendor code in Otto as the rollback strategy.

### 19.4 Contract compatibility

Server and client releases need an explicit compatibility policy. During rolling upgrades, at least one supported overlap must exist. Breaking contract changes require versioning, migration instructions, and pre-deployment checks.

## 20. Smalt archival policy

Archiving Smalt is the intended end state, but it should happen at a gate, not immediately.

Before archival:

- tag the final canonical source state;
- retain license and notices;
- publish a transition and compatibility report;
- document the OttoApps replacement;
- identify whether any published packages require deprecation notices;
- ensure no active CI, deployment, or automation still depends on Smalt;
- migrate or cross-reference unresolved security findings;
- preserve PR #486 and other useful design history;
- verify that the archive action will not block an essential migration reference.

Do not delete Smalt, rewrite its history, or merge the incomplete prototype merely to make the repository appear finished.

## 21. Acquisition-quality product requirements

The product should be designed so that Atlassian, ServiceNow, an enterprise buyer, or a managed-service operator could evaluate it seriously.

That requires more than tool count:

- a clear threat model;
- stable public contracts;
- least-privilege deployment patterns;
- comprehensive audit and policy controls;
- enterprise identity integration;
- high-quality documentation and runbooks;
- deterministic tests and live validation;
- compatibility and deprecation policy;
- provenance and signed releases;
- vendor API lifecycle tracking;
- extensibility for custom fields, tables, apps, and workflows;
- predictable performance and response bounds;
- operational SLOs and incident response;
- no dependence on a particular model vendor or agent harness;
- no runtime dependency on the vendors' official MCP servers.

The design should make it easy to demonstrate where OttoApps is safer or more capable than an official server, but claims must be backed by dated evidence and reproducible tests.

## 22. Risk register the plan must address

| Risk | Required treatment |
| --- | --- |
| Feature loss during consolidation | generated capability matrix and differential tests |
| Two large catalogs overwhelm clients | authorization visibility, role profiles, tool search, direct-call fallback |
| One process holds a highly privileged credential | per-module credentials, least privilege, restricted deployment profiles, process/container separation by vendor |
| Framework lock-in | framework-independent core and thin FastMCP adapter |
| Otto retains a hidden bypass | forbidden-import, config, network, and runtime path tests |
| Public repository leaks private data | provenance ledger, secret scan, synthetic fixtures, human review |
| Confirmation races in horizontal deployment | transactional shared state and unique one-shot spend constraint |
| Mutation retry causes duplicate changes | uncertainty ledger, reconciliation, idempotency evidence, no blind retry |
| Inbound identity confused with upstream identity | separate contracts, state, and audit fields |
| Empty allowlist widens access | fail-closed parsing and negative tests |
| Official vendor offerings change | dated comparison matrix and scheduled review |
| ServiceNow scoped app cannot be renamed safely | explicit ADR and compatibility plan |
| Smalt archived too early | objective cutover gates and final release checklist |
| OttoApps client and server drift | version negotiation, generated contract, rolling-upgrade tests |
| Package split recreates old complexity | one product family, two server identities, internal modules only |
| Large or malicious attachments | bounded handles, type policy, scanning hooks, object-store isolation |
| Audit or state backend outage | fail-safe mutation refusal, health signaling, recovery runbook |
| FastMCP version changes | reviewed pin, adapter isolation, protocol-level tests |
| License incompatibility | explicit source-by-source provenance and relicensing decision |

The final plan must add repository-specific risks found during investigation and assign an owner, severity, mitigation, validation method, and residual risk to each.

## 23. ADRs required before broad implementation

At minimum, prepare decisions for:

1. repository and product boundary;
2. Python version and dependency-management tool;
3. workspace and distribution layout;
4. FastMCP adapter boundary;
5. exactly two public server entry points;
6. capability manifest and schema-generation approach;
7. OttoApps client packaging and Otto integration contract;
8. tool-name compatibility and deprecation policy;
9. common result, provenance, and error envelope;
10. inbound authentication interface;
11. authorization and component-visibility model;
12. tenant and credential selection model;
13. local and distributed state backends;
14. confirmation and mutation-ledger protocol;
15. audit format, storage, retention, and access;
16. attachment and object-storage design;
17. ServiceNow scoped control-app migration;
18. observability and redaction policy;
19. release, versioning, and compatibility policy;
20. public license and source provenance;
21. Smalt archival and package deprecation;
22. official-server comparison and product-completeness process.

Each ADR must include context, considered alternatives, decision, consequences, migration impact, security impact, and validation evidence. Avoid placeholder ADRs that simply restate the preferred option.

## 24. Required planning-agent deliverables

The coding agent should perform read-only reconnaissance first and produce the following before implementing the transition:

1. **Source snapshot:** exact SHAs, branches, open PRs, releases, package versions, and repository instructions for Smalt, Otto, and OttoApps.
2. **Capability inventory:** every tool, resource, prompt, client operation, policy, side effect, credential type, and operational feature from Smalt and Otto.
3. **File-level migration map:** source path, current responsibility, target package/module, migration method, tests, dependencies, and deletion gate.
4. **Otto boundary report:** every artifact that violates the final “agent only” rule and the exact replacement/removal plan.
5. **Parity matrix:** source capability, target capability, compatibility requirement, current status, evidence, and approved deviation.
6. **Target architecture:** package graph, runtime processes, trust boundaries, data flows, state stores, and deployment profiles.
7. **Contract design:** capability manifest, client/server versioning, schemas, result envelope, error model, and provenance needed by Otto.
8. **Authentication and tenancy design:** inbound and upstream flows, credential stores, authorization, caller visibility, and tenant isolation.
9. **Security design:** threat model, policy invariants, confirmation, audit, mutation uncertainty, attachment handling, and redaction.
10. **Testing strategy:** unit, contract, differential, security, live, compatibility, packaging, migration, and Otto extraction gates.
11. **Implementation roadmap:** phases, atomic issues, dependencies, critical path, acceptance criteria, and evidence required to close each issue.
12. **Cutover plan:** staging, shadow reads, version compatibility, state migration, deployment sequence, and rollback.
13. **Issue migration report:** disposition for every relevant Smalt and Otto issue/PR.
14. **Provenance and licensing report:** what may be copied, what must be reimplemented, notices, and public-data review.
15. **Smalt archival checklist:** objective prerequisites and owner-approved external side effects.
16. **Open decisions:** only genuinely unresolved choices, each with evidence and a recommended default.

The roadmap should be detailed enough that individual implementation sessions can execute one issue without rediscovering architecture or silently changing scope.

## 25. Planning and implementation rules for coding agents

- Read all repository instructions before proposing changes.
- Do not implement until the owner approves the plan and ADR sequence.
- Do not treat generated catalogs as the only source of functionality; trace runtime behavior and tests.
- Do not assume a closed prototype PR is production-ready.
- Do not copy private code or payloads into the public repository without provenance approval.
- Do not weaken tests to make parity appear complete.
- Do not introduce a compatibility fallback without a named removal gate.
- Do not add another public MCP server.
- Do not put vendor logic or MCP implementation back into Otto.
- Do not make FastMCP types part of core domain interfaces.
- Do not add speculative abstraction for unrelated future vendors during source parity.
- Do not publish packages, containers, or releases without explicit approval.
- Do not archive Smalt, delete branches, close issues, or deprecate packages without explicit approval.
- Preserve unrelated changes in all repositories.
- Prefer small, reviewable vertical slices with end-to-end tests over horizontal scaffolding that proves no capability.
- Record deviations immediately; never silently approximate legacy safety behavior.

## 26. Final acceptance criteria

The transition is complete only when all of the following are true.

### OttoApps repository

- `oa-atlassian` and `oa-servicenow` are the only public MCP server identities.
- All approved Smalt capabilities are implemented or have an explicitly approved deprecation.
- All Atlassian and ServiceNow integration behavior needed by Otto is implemented.
- FastMCP is isolated to an adapter layer.
- Local and remote deployments pass their support matrices.
- Authentication, authorization, policy, audit, state, attachments, observability, and release controls are production-ready.
- Capability, client, and generated-documentation parity gates pass.
- Public provenance and licensing requirements are satisfied.

### Otto repository

- Otto contains only agent concerns and generic agent-owned interfaces.
- Otto contains no MCP server/client implementation.
- Otto contains no direct Atlassian or ServiceNow integration.
- Otto holds no Atlassian or ServiceNow credential.
- Otto has no direct vendor fallback.
- Otto can be prevented from reaching vendor origins at the network layer.
- Every source operation traverses the OttoApps-owned client and one of the two OttoApps servers.
- Agent behavior, grounding, citations, cancellation, and delivery remain correct.

### Smalt repository

- Smalt is no longer required by Otto or supported OttoApps deployments.
- Its final state, migration path, licenses, and unresolved findings are documented.
- Its useful history and prototype references are preserved.
- It is archived only after the owner explicitly approves the archive action.

### Product ambition

- OttoApps has a maintained path from source parity to official-server parity and superiority.
- Claims of replacement or improvement are backed by dated evidence.
- The product remains independent of any one model, harness, host, identity provider, cloud, or vendor MCP implementation.

## 27. Decision defaults

Unless repository evidence demonstrates a better option, the planning agent should use these defaults:

- create OttoApps as a new repository rather than renaming Smalt;
- preserve Smalt as legacy history until gated archival;
- use exactly two public servers;
- preserve existing tool names during initial parity;
- keep vendor modules internal;
- keep core logic independent of FastMCP;
- publish the concrete first-party MCP client from OttoApps, not Otto;
- keep Otto free of vendor and MCP implementation;
- use standard MCP behavior with optional negotiated enhancements;
- support stdio and Streamable HTTP;
- use local state for simple deployments and transactional shared state for distributed writes;
- keep inbound caller identity separate from upstream vendor identity;
- fail closed on missing tenant, allowlist, origin, or credential configuration;
- complete shared mutation safety before porting production writes;
- use differential tests to prove parity;
- archive Smalt only after Otto cutover and objective parity gates.

These defaults are intended to prevent repeated debate over already chosen product direction. A deviation should require evidence, an ADR, and owner approval.

## 28. Closing instruction

Plan this as a controlled product transition across three repositories, not as a framework port or mass file move.

The desired final boundary is simple and absolute:

> **OttoApps owns all Atlassian, ServiceNow, and MCP functionality. Otto owns only the agent. Smalt becomes archived legacy history after OttoApps safely replaces it.**

Every proposed package, dependency, configuration field, network path, credential, tool, test, issue, and deployment artifact should be evaluated against that sentence.
