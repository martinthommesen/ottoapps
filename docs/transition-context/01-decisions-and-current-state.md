# Part I — Non-negotiable product and repository decisions

## 3. Owner decisions

The following decisions are authoritative.

### 3.1 Project identity

**DECIDED**

- The new project is named **OttoApps**.
- The canonical repository is `martinthommesen/ottoapps`.
- The public server names and executable entry points are:
  - `oa-atlassian`
  - `oa-servicenow`
- New public branding, package names, documentation, resource URIs, configuration names, logs, images, and server metadata must use OttoApps/OA terminology rather than Smalt terminology.
- Smalt is not renamed in place. It remains the historical Smalt project.

### 3.2 Exactly two public MCP servers

**DECIDED**

OttoApps has exactly two public MCP server products:

| Server | Product responsibility |
|---|---|
| `oa-atlassian` | Atlassian capabilities, initially Jira and Confluence, with room for JSM, Assets, Compass, Bitbucket, and additional Atlassian products |
| `oa-servicenow` | ServiceNow capabilities, including the complete useful platform surface represented by Smalt and later additions |

Internally, each server may contain many capability modules, providers, services, clients, policies, and adapters. Those internal modules must not become additional public MCP server products.

Multiple deployments of the same server binary are allowed. For example, a high-assurance installation may run two differently scoped instances of `oa-servicenow`. They are still deployments of one server product, not new server types.

### 3.3 OttoApps owns all MCP server and vendor-integration functionality

**DECIDED**

`martinthommesen/ottoapps` becomes the sole owner of:

- MCP server construction and registration;
- FastMCP server adapters;
- public tools, resources, prompts, and server capability metadata;
- Atlassian and ServiceNow API clients;
- vendor-specific authentication and token refresh;
- vendor-specific request building, query compilation, response parsing, and schema handling;
- upstream tenant selection and tenant binding;
- vendor allowlists and capability policy;
- write gates and confirmation handling;
- mutation receipts, uncertain-outcome handling, and reconciliation support;
- server-side security audit;
- server-side rate limits, upstream concurrency limits, retries, caches, and response bounds;
- attachment upload, download, export, storage, and validation behavior;
- ServiceNow control-app artifacts and operator tooling, if retained;
- official/community MCP compatibility adapters, if any are deliberately retained;
- all tests, fixtures, documentation, deployment files, and diagnostics belonging to those responsibilities.

### 3.4 Only the Otto agent remains in `martinthommesen/otto`

**DECIDED**

The Otto repository remains the home of the **agent**, not the integrations it uses.

Otto may retain only the functionality required to be the agent:

- Webex or other conversation transports;
- Pydantic AI or another model orchestration runtime;
- model-provider configuration and policy;
- the agent prompt and model-facing canonical tools;
- caller admission and destination-delivery policy;
- model-egress policy;
- evidence registration and structural grounding;
- citation rendering and output sanitization;
- conversation memory;
- turn coordination, delivery state, and rate limiting at the agent boundary;
- agent operational logs and agent security audit;
- generic MCP client functionality needed to connect to OttoApps;
- narrow, vendor-neutral or consumer-side adapters that translate OttoApps structured results into Otto evidence objects;
- startup and doctor checks that validate Otto’s connection to OttoApps.

Otto **MUST NOT** retain:

- an MCP server;
- a FastMCP `FastMCP` server instance or server registration code;
- direct Atlassian or ServiceNow HTTP clients;
- direct Atlassian or ServiceNow credentials;
- bindings to Atlassian’s official hosted MCP server;
- bindings to a community `mcp-atlassian` sidecar;
- ServiceNow Table API implementation;
- vendor query languages or serializers such as CQL or ServiceNow encoded-query construction;
- vendor response-shape parsers;
- vendor-specific pagination or cursor implementations;
- vendor-specific attachment behavior;
- upstream identity probes;
- vendor-specific write, confirmation, or audit logic;
- fallback paths that bypass OttoApps and call a vendor directly;
- duplicated copies of OttoApps domain or security code.

A minimal generic MCP client in Otto is not a violation of this rule. It is protocol consumption by the agent. The line is that Otto may know **how to call a trusted MCP endpoint and validate its consumer contract**, but must not know how Atlassian or ServiceNow implement the requested operation.

### 3.5 Direction of dependency

**DECIDED**

The dependency direction is:

```text
Otto agent ──MCP protocol──> OttoApps ──vendor APIs──> Atlassian / ServiceNow
```

Not:

```text
OttoApps ──imports──> Otto
Otto ──imports──> OttoApps internal Python modules
Otto ──HTTP/API──> Atlassian / ServiceNow
```

There must be no cross-repository Python import dependency between Otto and OttoApps. Their compatibility boundary is MCP plus explicitly versioned structured contracts.

### 3.6 Product ambition

**DECIDED**

OttoApps is intended to become a full replacement and material improvement over official vendor MCP servers, not merely a wrapper around them.

That means the project should aim for:

- broader useful capability coverage;
- safer write and destructive-operation handling;
- stronger least-privilege controls;
- better structured output and agent ergonomics;
- auditable, explainable policy decisions;
- deployment independence;
- authentication independence;
- model and harness independence;
- local and remote operation;
- excellent diagnostics and documentation;
- enterprise-grade observability and lifecycle management;
- extensibility for custom ServiceNow applications and Atlassian deployments.

This ambition does **not** permit unverified “complete coverage” claims. Coverage must be represented by an evidence-backed capability matrix.

### 3.7 Smalt repository lifecycle

**DECIDED**

Smalt is to be archived as the predecessor project after a deliberate archival handoff.

Before the GitHub archive flag is applied, the transition plan must include:

1. a final source and capability snapshot;
2. a final commit or archival note pointing to OttoApps;
3. a preserved reference to the closed, unmerged FastMCP prototype;
4. issue disposition or migration;
5. a license, provenance, and secret/privacy review;
6. any final tag needed to identify the authoritative Smalt baseline.

The repository should enter feature freeze immediately. Archiving must not happen so early that essential preservation work becomes awkward, and it must not be delayed merely to keep implementing the successor inside Smalt.

---

## 4. Definition of “all MCP functionality moves out of Otto”

This phrase is intentionally broader than “move the MCP client file.”

The migration includes every layer that exists because Otto currently knows how to integrate directly with Atlassian or ServiceNow.

### 4.1 Functionality that moves to OttoApps

Examples from the current Otto tree include:

- `src/otto/integrations/confluence/adapter.py`
- `src/otto/integrations/confluence/contracts.py`
- `src/otto/integrations/confluence/hosted_atlassian.py`
- `src/otto/integrations/confluence/mcp_client.py`
- `src/otto/integrations/confluence/payload.py`
- `src/otto/integrations/confluence/query.py`
- `src/otto/integrations/confluence/self_hosted.py`
- `src/otto/integrations/servicenow/adapter.py`
- `src/otto/integrations/servicenow/contracts.py`
- `src/otto/integrations/servicenow/query.py`
- `src/otto/integrations/servicenow/rest.py`
- vendor credential construction in `src/otto/credentials/` and `src/otto/app.py`;
- hosted Atlassian and ServiceNow configuration in `src/otto/config.py`;
- ServiceNow provisioning and permission-verification utilities under `tools/`;
- vendor-specific doctor checks and test fixtures;
- tests that lock direct vendor behavior rather than Otto’s consumer contract.

Some code should be **ported**, some should be **reimplemented using Smalt’s stronger behavior**, and some should be **deleted as obsolete**. “Move” does not require copying code byte-for-byte.

### 4.2 Functionality that stays in Otto

Examples include:

- `src/otto/runtime/agent.py` as the model-facing agent layer;
- `src/otto/runtime/pipeline.py`;
- `src/otto/runtime/turn_coordinator.py`;
- `src/otto/conversation/`;
- `src/otto/grounding/`;
- `src/otto/transports/`;
- agent caller admission and delivery policy;
- agent source/model/destination classification gates;
- evidence objects and citation rendering;
- model resolver and provider policy;
- agent audit and delivery telemetry;
- canonical model-facing tool names, unless a separate Otto redesign intentionally changes them.

The implementation of those canonical tools must change. They should invoke OttoApps, not direct vendor adapters.

### 4.3 Consumer-side code that may remain in Otto

A small `otto.sources.ottoapps` or equivalent package may remain, containing:

- generic MCP transport/session setup;
- authentication to OttoApps;
- expected OttoApps tool-schema fingerprints or compatible-range declarations;
- narrow protocols such as `AtlassianKnowledgeSource` and `ServiceNowKnowledgeSource`;
- conversion from OttoApps structured records to Otto `EvidenceRecord`;
- client-side timeouts and classification of transport failures;
- readiness and diagnostics calls.

It must not contain vendor query construction, vendor credentials, vendor response parsing, or server policy.

---

# Part II — Evidence snapshot and current state

## 5. Snapshot references

The planning agent must verify these references before using them. If a branch has advanced, it must record and analyze the delta.

| Repository or reference | Snapshot |
|---|---|
| `martinthommesen/ottoapps` | Empty public repository at the start of this document |
| `martinthommesen/smalt` `main` | `b0ad4563be55aeb6c772b97ea336ae2f753b8093` |
| `martinthommesen/otto` `main` | `1e609992fa677e2f1aa8840b96c331a0640be7ec` |
| Smalt FastMCP prototype PR | Closed, unmerged PR [#486](https://github.com/martinthommesen/smalt/pull/486) |
| PR #486 head | `e3c14239370d91ce37beef91afbf16f162db08e5` |
| PR #486 base | `b0ad4563be55aeb6c772b97ea336ae2f753b8093` |

The new public repository must not blindly import either private repository’s Git history.

---

## 6. Current Smalt architecture

Smalt is the primary capability and safety source for OttoApps.

At the snapshot, Smalt is a TypeScript/pnpm workspace with a shared private runtime and thirteen MCP server packages. Its architecture has several valuable properties that must not be lost merely because the packaging changes.

### 6.1 Smalt public server packages

| Current Smalt server | OttoApps destination |
|---|---|
| `@smalt/mcp-confluence` | `oa-atlassian` → Confluence core capability module |
| `@smalt/mcp-confluence-permissions` | `oa-atlassian` → Confluence permissions capability module |
| `@smalt/mcp-confluence-api` | `oa-atlassian` → constrained Atlassian/Confluence API capability module |
| `@smalt/mcp-jira` | `oa-atlassian` → Jira capability module |
| `@smalt/mcp-servicenow-lookup` | `oa-servicenow` → lookup capability module |
| `@smalt/mcp-servicenow-itsm` | `oa-servicenow` → ITSM capability module |
| `@smalt/mcp-servicenow-table-read` | `oa-servicenow` → generic table-read capability module |
| `@smalt/mcp-servicenow-table-write` | `oa-servicenow` → generic table-write capability module |
| `@smalt/mcp-servicenow-approvals` | `oa-servicenow` → approvals capability module |
| `@smalt/mcp-servicenow-cmdb-write` | `oa-servicenow` → CMDB/IRE capability module |
| `@smalt/mcp-servicenow-knowledge-write` | `oa-servicenow` → knowledge capability module |
| `@smalt/mcp-servicenow-control-audit` | `oa-servicenow` → control-app audit/admin capability module |
| `@smalt/mcp-audit` | No third public server; becomes admin tooling and/or privileged capabilities within the two servers |

Smalt also has:

- `@smalt/internal-runtime`;
- `@smalt/skills`;
- the `smalt` wrapper/CLI;
- generated capability documentation and machine-readable catalogs;
- an optional ServiceNow scoped control application;
- packed-replay and live verification infrastructure.

### 6.2 Smalt behavior worth preserving

The successor must account for, test, and preserve or deliberately improve at least:

- env-oriented, fail-closed configuration;
- one configured origin per connection context;
- exact authentication-mode validation;
- explicit table, class, space, and project allowlists;
- production-origin acknowledgements;
- per-capability write gates;
- preview-before-execute flows;
- fresh, one-shot confirmations;
- confirmation receipts;
- audit-before-mutation;
- mutation ledgers;
- explicit uncertain outcomes;
- no blind retry of ambiguous writes;
- acting-identity pinning and identity drift detection;
- bounded HTTP;
- response redaction and output validation;
- rate limiting and retry classification;
- proxy and private-CA support;
- metadata caching;
- tracing;
- attachment size, type, extension, and path controls;
- capability declarations as a single source of truth;
- registration parity between declared and actual tools;
- generated operator docs;
- real packed MCP-session tests;
- opt-in live tests against isolated vendor sandboxes.

### 6.3 Smalt is a specification, not just a code donor

For every Smalt capability, the transition plan must answer:

1. What is the current public name?
2. What are the input and output schemas?
3. Is it read, write, or destructive?
4. What environment and policy contract applies?
5. What tables, endpoints, or vendor permissions does it touch?
6. What identity does it act as?
7. What audit and confirmation behavior applies?
8. What are the pagination, retry, and uncertain-outcome semantics?
9. What tests and fixtures prove the behavior?
10. What will OttoApps call it?
11. Is the behavior equivalent, improved, or intentionally retired?
12. What evidence proves the disposition?

A green tool-count comparison alone is not parity.

---

## 7. Closed FastMCP prototype PR #486

PR #486 is a useful prototype and design reference, but it is closed and unmerged. It must not be treated as if it exists on Smalt `main`.

### 7.1 What the prototype proved

The prototype demonstrated:

- a Python project using a FastMCP 4 prerelease;
- two public server builders;
- stdio and Streamable HTTP startup;
- a stateless HTTP mode;
- an Atlassian server shell;
- the first ServiceNow lookup vertical slice;
- a catalog-parity harness;
- fail-closed ServiceNow environment parsing;
- Basic, API-key, and OAuth credential construction;
- in-memory OAuth refresh with single-flight behavior;
- an origin-pinned bounded HTTP client;
- acting-identity pinning and a drift guard;
- typed error codes;
- 31 passing tests in the prototype branch;
- a working MCP initialize smoke test over HTTP.

The implemented ServiceNow tools were:

- `snow_ping`
- `snow_whoami`
- `snow_acting_user_get`
- `snow_user_get`
- `snow_group_get`
- `snow_assignment_group_get`

The prototype reported six of the twenty-two Smalt lookup tools ported at that point.

### 7.2 What the prototype did not prove

It did not provide:

- full Smalt capability parity;
- any Atlassian capability;
- mutation safety;
- shared confirmation or audit state;
- production client authentication;
- horizontal-scale correctness;
- full differential output parity;
- stable FastMCP 4 production readiness;
- an Otto consumer migration;
- the OttoApps naming and repository boundaries.

### 7.3 How to use it

**RECOMMENDED**

- Preserve the PR head SHA as a source reference.
- Extract its design and tests into a provenance manifest.
- Port selected code into clean OttoApps commits after review.
- Rename packages and symbols deliberately; do not perform a global blind substitution.
- Fix prototype-level dependency policy. A production project should not use an unbounded `fastmcp>=...` requirement for a prerelease framework.
- Re-evaluate every prototype behavior against current FastMCP 4 and MCP SDK documentation at implementation time.

Do not merge PR #486 into Smalt merely to move it later.

---

## 8. Current Otto architecture and why it must be split

Otto currently combines a well-separated agent pipeline with vendor integration implementations.

### 8.1 Agent responsibilities already well placed

Otto’s agent architecture includes:

- a Webex websocket transport;
- caller and destination admission;
- a per-conversation coordinator;
- a Pydantic AI model runtime;
- canonical model-facing source tools;
- structural evidence grounding;
- citation validation and rendering;
- output sanitization;
- conversation memory;
- delivery state;
- agent audit and telemetry.

These are Otto concerns and should remain.

### 8.2 Vendor/MCP responsibilities currently misplaced in Otto

Otto also currently:

- talks to the hosted Atlassian Rovo MCP server;
- supports a self-hosted community `mcp-atlassian` sidecar;
- knows those servers’ tool names, argument shapes, and result wrappers;
- constructs a FastMCP client;
- pins an Atlassian Cloud ID;
- serializes CQL;
- parses Atlassian page payloads and ADF/storage representations;
- directly authenticates to ServiceNow;
- constructs ServiceNow encoded queries;
- performs ServiceNow Table API GETs;
- implements retries and byte bounds;
- maps vendor rows into agent evidence;
- owns upstream space, table, field, and workflow-state scope;
- owns upstream credentials.

Those responsibilities belong in OttoApps.

### 8.3 Otto’s canonical tools should survive the cutover

The current canonical tool approach is valuable: the model sees Otto-owned, task-oriented tools rather than a full upstream catalog.

Examples include:

- `confluence_search`
- `confluence_get_page`
- `servicenow_search_incidents`
- `servicenow_get_incident`
- `servicenow_search_knowledge`
- `servicenow_get_knowledge_article`
- optional case variants

**RECOMMENDED**

Keep those model-facing names and agent contracts stable during the infrastructure migration. Change their implementation to call OttoApps tools.

This avoids combining:

- a repository migration;
- an integration rewrite;
- a server rewrite;
- and an agent behavior/tool-selection migration

in one step.

OttoApps must not become Otto-specific, however. It should expose generally useful tools. Otto’s canonical wrappers may compose or narrow those tools.

---
