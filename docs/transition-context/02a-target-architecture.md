# Part III — Target OttoApps product architecture

## 9. Product identity and public names

### 9.1 Canonical names

| Concern | Canonical name |
|---|---|
| Product/project | OttoApps |
| Repository | `martinthommesen/ottoapps` |
| Python import root | `ottoapps` |
| Distribution | `ottoapps` |
| Atlassian executable | `oa-atlassian` |
| ServiceNow executable | `oa-servicenow` |
| Optional administrative CLI | `oa` |
| Atlassian MCP server name | `oa-atlassian` |
| ServiceNow MCP server name | `oa-servicenow` |
| Default URI scheme | `oa://` |
| Documentation shorthand | OA |

### 9.2 Names that must disappear from the new public surface

Before the first public stable release, no user-facing contract should use:

- `smalt`;
- `smalt-mcp`;
- `@smalt/*`;
- `SMALT_*`;
- `smalt://`;
- “Smalt v2”;
- old `company-ai-tools` terminology.

Legacy names may appear only in:

- migration documentation;
- provenance records;
- differential-test fixture names;
- compatibility maps;
- historical acknowledgements.

### 9.3 Tool names

Most existing tool names already use domain prefixes such as `snow_`, `confluence_`, and `jira_`. They do not need an `oa_` prefix merely to reflect product branding.

**OPEN**

Before tool names become a public stable contract, decide whether to:

1. preserve the existing names exactly;
2. normalize known inconsistencies first;
3. introduce a versioned rename map.

Smalt issue #479 identifies known inconsistent families. Because Smalt has not become an established public dependency, the preferred approach is to normalize deliberately before OttoApps 1.0 and avoid permanent aliases.

Any rename plan must provide semantic parity mapping, not just string replacement.

### 9.4 Trademark and affiliation language

**OPEN**

The server names contain Atlassian and ServiceNow trademarks. Before broad public marketing or package publication:

- review nominative-use and trademark wording;
- add a clear non-affiliation disclaimer;
- avoid logos or branding that imply vendor endorsement;
- keep product names descriptive;
- document that vendor names identify interoperability targets.

---

## 10. Architectural principles

OttoApps must follow these principles.

### 10.1 FastMCP is an adapter, not the domain architecture

Core vendor behavior must not import FastMCP.

A capability operation should be callable from:

- a FastMCP tool adapter;
- a unit test;
- an administrative CLI;
- a future REST or job interface;
- a migration/differential harness

without instantiating an MCP server.

### 10.2 Two public servers, many internal capability modules

The public composition should be:

```text
                         OttoApps
                            │
             ┌──────────────┴──────────────┐
             │                             │
       oa-atlassian                  oa-servicenow
             │                             │
   ┌─────────┼─────────┐        ┌─────────┼───────────────────┐
   │         │         │        │         │                   │
Confluence  Jira   Atlassian  Lookup     ITSM             Generic tables
 core               admin     CMDB       Knowledge        Approvals
permissions         escape    Control app Attachments     ...
```

Internal capability modules may be FastMCP providers or mounted child servers where that reduces registration duplication. They must be mounted without accidental public namespace changes.

Internal modules are not MCP protocol extensions. Protocol extensions are negotiated wire-level capabilities, not a label for product modules.

### 10.3 Separate processes and secret sets

Even if built from one repository, wheel, or image:

- `oa-atlassian` and `oa-servicenow` run as separate processes;
- they receive separate environment and secret sets;
- a compromise of one must not reveal the other’s credentials;
- they have separate readiness, audit, and lifecycle signals;
- one must not import or initialize the other’s vendor client.

### 10.4 Protocol-first, harness-neutral

OttoApps must not contain:

- Pydantic AI;
- LangChain;
- an OpenAI/Anthropic/Google model client;
- Webex;
- Slack;
- a ChatGPT-specific integration;
- a Claude-specific integration;
- Cursor-specific behavior;
- model-name branching.

Compatibility differences should be handled through MCP capability negotiation, documented transport profiles, or standards-compatible transforms—not by detecting client brands.

### 10.5 No official-server dependency

The normal `oa-atlassian` runtime must not depend on Atlassian’s official Rovo MCP server.

The normal `oa-servicenow` runtime must not depend on ServiceNow’s official MCP server.

Official servers may be used as:

- behavior references;
- conformance comparison targets;
- optional, clearly isolated compatibility adapters;
- live capability-comparison fixtures

but not as the implementation underneath the replacement product.

### 10.6 Fail closed at trust boundaries

Missing or contradictory configuration, unknown tenant selection, missing scopes, unknown write policy, output-schema mismatch, and identity drift must fail closed.

“Empty means unrestricted” must not be a silent default.

### 10.7 Prefer explicit contracts over framework magic

Every capability must have one authoritative declaration for:

- public name;
- description;
- input schema;
- output schema;
- read/write/destructive class;
- authorization scopes;
- required upstream permissions;
- configuration;
- tables/endpoints;
- confirmation requirements;
- audit class;
- retry/idempotency behavior;
- privacy/redaction rules.

The FastMCP registration should be generated from or checked against that declaration.

---

## 11. Recommended repository shape

This is a starting architecture, not permission to create every directory before it is needed.

```text
ottoapps/
├── src/
│   └── ottoapps/
│       ├── core/
│       │   ├── auth/
│       │   │   ├── caller.py
│       │   │   ├── authorization.py
│       │   │   ├── upstream.py
│       │   │   └── identity.py
│       │   ├── audit/
│       │   ├── capabilities/
│       │   ├── confirmations/
│       │   ├── errors/
│       │   ├── http/
│       │   ├── policy/
│       │   ├── state/
│       │   ├── attachments/
│       │   ├── telemetry/
│       │   └── types/
│       │
│       ├── atlassian/
│       │   ├── app.py
│       │   ├── client/
│       │   ├── auth/
│       │   ├── models/
│       │   ├── capabilities/
│       │   │   ├── confluence/
│       │   │   ├── confluence_permissions/
│       │   │   ├── jira/
│       │   │   └── constrained_api/
│       │   └── policy/
│       │
│       ├── servicenow/
│       │   ├── app.py
│       │   ├── client/
│       │   ├── auth/
│       │   ├── models/
│       │   ├── capabilities/
│       │   │   ├── lookup/
│       │   │   ├── itsm/
│       │   │   ├── table_read/
│       │   │   ├── table_write/
│       │   │   ├── approvals/
│       │   │   ├── cmdb/
│       │   │   ├── knowledge/
│       │   │   └── control_audit/
│       │   ├── control_app/
│       │   └── policy/
│       │
│       ├── mcp/
│       │   └── fastmcp/
│       │       ├── atlassian.py
│       │       ├── servicenow.py
│       │       ├── auth.py
│       │       ├── middleware.py
│       │       ├── transforms.py
│       │       └── transports.py
│       │
│       └── cli/
│           ├── atlassian.py
│           ├── servicenow.py
│           └── admin.py
│
├── tests/
│   ├── unit/
│   ├── contract/
│   ├── differential/
│   ├── integration/
│   ├── security/
│   ├── packaging/
│   ├── performance/
│   ├── live/
│   └── fixtures/
│       ├── smalt_v1/
│       ├── otto_v1/
│       ├── atlassian/
│       └── servicenow/
│
├── contracts/
│   ├── capabilities/
│   ├── compatibility/
│   └── schemas/
├── docs/
│   ├── adr/
│   ├── architecture/
│   ├── security/
│   ├── operations/
│   ├── migration/
│   └── generated/
├── deployments/
│   ├── containers/
│   ├── compose/
│   ├── systemd/
│   └── kubernetes/
├── scripts/
├── pyproject.toml
├── uv.lock
└── README.md
```

### 11.1 Dependency direction inside the repository

```text
FastMCP/CLI adapters
        │
        ▼
Capability application services
        │
        ▼
Policy + typed vendor client ports
        │
        ▼
HTTP/auth/state/telemetry adapters
```

Forbidden directions include:

```text
core ──> fastmcp
vendor domain ──> CLI
ServiceNow ──> Atlassian
Atlassian ──> ServiceNow
domain service ──> concrete PostgreSQL/Redis implementation
```

Add architecture tests or import-boundary checks so these constraints are executable.

### 11.5 Extension model

OttoApps should be extensible without turning the initial migration into a plugin-framework project.

For 1.0:

- capability modules live in-tree;
- each module implements explicit typed interfaces;
- registration is driven by the capability catalog;
- vendor clients are injected through ports;
- custom ServiceNow tables/scoped applications and Atlassian API surfaces use configuration plus constrained adapters where feasible.

Later, if external extensions are justified, consider Python entry points or a separately versioned provider API. Do not expose unstable internal classes as a public extension contract accidentally.

An extension must not be able to bypass:

- caller authorization;
- origin pinning;
- output validation;
- redaction;
- audit;
- confirmations;
- shared HTTP limits;
- attachment policy.

---

## 12. Public server composition

### 12.1 `oa-atlassian`

Initial internally mounted capability groups:

- Confluence content;
- Confluence attachments;
- Confluence comments and blogs;
- Confluence content properties;
- Confluence classifications;
- Confluence permissions and access explanation;
- Jira issues;
- Jira comments;
- Jira links;
- Jira transitions;
- Jira JQL validation/search;
- constrained Atlassian API escape hatch;
- server identity and policy resources.

Future capability candidates:

- Jira Service Management;
- Assets;
- Compass;
- Bitbucket;
- organization/admin APIs;
- Forge/custom-app endpoints.

Future candidates must not distort the initial architecture before core parity is achieved.

### 12.2 `oa-servicenow`

Initial internally mounted capability groups:

- identity and lookup;
- users, groups, assignment groups, membership, and on-call;
- curated ITSM;
- incidents, problems, changes, tasks, SLAs, journals, and history;
- Service Catalog;
- approvals;
- knowledge;
- CMDB reads, relationships, hierarchy, and IRE-backed writes;
- generic table reads;
- generic table writes;
- attachments;
- metadata/schema;
- optional control-app audit and authoritative writes;
- server policy and identity resources.

Future capability candidates:

- CSM;
- HRSD, only with a deliberate high-sensitivity design;
- ITOM;
- SecOps;
- SPM;
- custom scoped applications;
- Flow Designer actions;
- additional platform APIs.

### 12.3 Audit is not a third MCP server

The current `mcp-audit` functionality must be redesigned as one or more of:

- privileged admin tools inside each server;
- `oa` CLI commands over the shared audit store;
- read-only audit resources available only to authorized administrators;
- external SIEM/export APIs.

It must not become `oa-audit` as a third public MCP server.

### 12.4 Evidence-backed replacement benchmark

Being a “replacement” is a maintained engineering claim, not a slogan.

Create a versioned comparison document for each vendor. Each row must cite the official server documentation or observed behavior and the OttoApps evidence that supports the comparison.

At minimum, compare:

| Dimension | Required evidence |
|---|---|
| Product/API coverage | Tool-by-tool or workflow-by-workflow matrix |
| Read behavior | Inputs, projections, pagination, visibility, and limits |
| Write behavior | Confirmation, idempotency, receipts, and reconciliation |
| Authentication | Caller auth and supported upstream credential modes |
| Authorization | Discovery filtering, scopes, allowlists, row/field policy |
| Tenancy | Binding, isolation, delegated identity, and audit |
| Hosting | Local, remote, self-hosted, and data-residency choices |
| Harness compatibility | Real protocol tests, not client-name claims |
| Audit/governance | Security audit, policy explanation, and export |
| Customization | Custom tables, scoped apps, projects, spaces, and APIs |
| Reliability | Retries, rate limits, uncertain writes, and state recovery |
| Attachments | Limits, scanning, object handling, and path safety |
| Observability | Logs, metrics, traces, diagnostics, and redaction |
| Supply chain | SBOM, provenance, signatures, and vulnerability policy |
| Operator experience | Install, configure, doctor, upgrade, and rollback |
| Performance | Discovery size, latency, throughput, and resource use |

Rules for the comparison:

- record the date and vendor server/version;
- distinguish documented behavior from behavior observed in tests;
- do not copy proprietary implementation;
- do not use marketing language where evidence is incomplete;
- mark unknowns as unknown;
- update the matrix when vendors change their server;
- track OttoApps improvements separately from parity;
- never block a security improvement merely to mimic weaker official behavior.

A capability may be “better” because it is safer, more portable, more configurable, more complete, or easier for agents—not merely because OttoApps has more tools.

---
