# Part V — Authentication, authorization, identity, and tenancy

## 18. Two separate authentication planes

Always distinguish:

```text
Caller → OttoApps
```

from:

```text
OttoApps → Atlassian or ServiceNow
```

A caller token proves who may use OttoApps. It must not automatically select an upstream vendor identity.

### 18.1 Caller-to-OttoApps authentication

The core must expose a `CallerAuthenticator`/`CallerAuthorizer` abstraction or equivalent.

Reference modes may include:

- stdio process/OS trust;
- static scoped bearer token for local development;
- OAuth 2.1/OIDC JWT validation;
- authorization-server integration;
- mTLS;
- trusted reverse-proxy identity only with explicit signed assertions and proxy pinning;
- workload identity for service-to-service use.

The implementation must not be tied to:

- Entra ID;
- Okta;
- Auth0;
- Keycloak;
- Cloudflare Access;
- WorkOS;
- one cloud provider.

Reference adapters are fine. The authorization model must remain portable.

### 18.2 Upstream vendor authentication

The core must expose an `UpstreamCredentialProvider`/`CredentialSession` abstraction or equivalent.

ServiceNow modes to preserve or evaluate:

- Basic auth;
- API key;
- OAuth client credentials;
- OAuth refresh token;
- delegated user OAuth;
- optional mutual TLS where required.

Atlassian modes to preserve or evaluate:

- email + API token;
- service-account/API token;
- OAuth 2.0/3LO;
- delegated user OAuth;
- Data Center PAT/basic modes if Data Center becomes supported.

Rules:

- exactly one applicable authentication mode per connection profile;
- secrets never printed or included in validation errors;
- refresh tokens and access tokens never become tool arguments;
- refresh is single-flight;
- in-memory tokens are the default for single-process service identities;
- delegated token persistence requires encryption, rotation, revocation, and explicit storage design;
- auth headers are reconstructed at the HTTP boundary;
- caller-supplied tool input cannot choose a credential object.

### 18.3 Otto’s post-transition credentials

Otto may hold:

- Webex credentials;
- model-provider credentials;
- its own audit key;
- credentials used to authenticate Otto to `oa-atlassian`;
- credentials used to authenticate Otto to `oa-servicenow`.

Otto must not hold:

- Atlassian API tokens;
- Atlassian OAuth refresh tokens;
- ServiceNow usernames/passwords/API keys;
- ServiceNow OAuth client secrets or refresh tokens;
- vendor tenant-routing secrets.

### 18.4 Identity context

Every OttoApps request should have a typed context containing, where applicable:

- authenticated caller subject;
- issuer;
- audience;
- caller tenant;
- roles/scopes;
- connection/deployment profile;
- resolved upstream identity;
- upstream tenant/site/instance;
- correlation ID;
- capability/tool;
- request start/deadline.

Audit records should distinguish caller identity from upstream acting identity.

---

## 19. Tenancy

### 19.1 Initial recommendation

Start with **one explicitly configured upstream tenant per server process**.

This means:

- one Atlassian site per `oa-atlassian` process;
- one ServiceNow instance per `oa-servicenow` process;
- no tenant/tool argument that selects arbitrary credentials;
- deployment can run multiple instances for multiple tenants.

This is operationally simple and preserves strong tenant binding.

### 19.2 Multi-tenant future

A multi-tenant server is a separate phase requiring:

- authenticated tenant routing;
- tenant configuration isolation;
- encrypted per-tenant secret storage;
- cache partitioning;
- rate-limit partitioning;
- audit partitioning;
- object-storage partitioning;
- per-tenant authorization;
- no caller-controlled tenant confusion;
- tests for cross-tenant leakage;
- operational deletion and key-rotation procedures.

Do not make the core impossible to extend, but do not prebuild a speculative multi-tenant platform before single-tenant parity.

---

## 20. Authorization and capability visibility

Authorization must operate at several levels:

1. server access;
2. component visibility;
3. tool visibility;
4. tool execution;
5. resource/prompt visibility;
6. input policy;
7. vendor table/project/space/class allowlists;
8. row/object visibility;
9. field projection;
10. write/destructive gates;
11. confirmation;
12. attachment policy.

A broad server token must not silently imply every write capability.

Recommended scope families include:

```text
oa:atlassian:confluence:read
oa:atlassian:confluence:write
oa:atlassian:permissions:write
oa:atlassian:jira:read
oa:atlassian:jira:write

oa:servicenow:lookup:read
oa:servicenow:itsm:read
oa:servicenow:itsm:write
oa:servicenow:table:read
oa:servicenow:table:write
oa:servicenow:cmdb:write
oa:servicenow:approvals:write
oa:servicenow:audit:read
```

These are examples. The final scope vocabulary needs an ADR and generated documentation.

---

# Part VI — Security invariants

## 21. Threat model

At minimum, plan for:

- a compromised or poorly behaved model;
- malicious tool arguments;
- prompt-injected vendor content;
- a compromised MCP client;
- unauthorized direct tool invocation;
- stolen or replayed confirmations;
- cross-tenant confusion;
- upstream identity drift;
- credential leakage;
- malicious vendor response payloads;
- SSRF and redirect attacks;
- DNS and proxy misconfiguration;
- decompression bombs and oversized bodies;
- rate-limit exhaustion;
- write timeout after upstream success;
- duplicate or replayed writes;
- path traversal;
- malicious attachments;
- cache poisoning;
- log/tracing leakage;
- horizontal-scale races;
- audit tampering;
- dependency/supply-chain compromise.

A threat-model document must map each threat to controls and tests.

---

## 22. Network and HTTP requirements

The shared HTTP runtime should preserve or improve Smalt’s behavior.

Requirements:

- configured origin parsing, not string-prefix checks;
- HTTPS required outside explicit local development;
- no URL userinfo;
- no caller-controlled host;
- no unexpected scheme, fragment, or query in base origins;
- redirects disabled unless a reviewed endpoint explicitly requires a safe redirect strategy;
- requests pinned to the configured origin;
- bounded connect/read/write/pool timeouts;
- streamed response-size enforcement;
- decompressed-byte limits, not only `Content-Length`;
- JSON depth/shape bounds where practical;
- proxy and `NO_PROXY` support;
- private CA support;
- retry classification;
- bounded `Retry-After`;
- idempotency-aware retry policy;
- no blind retry of ambiguous writes;
- per-upstream concurrency limits;
- per-caller and per-tenant rate limits;
- cancellation and deadline propagation;
- content-free error messages.

Network egress policy is a deployment defense and should be documented alongside application controls.

---

## 23. Query and generic API safety

### 23.1 ServiceNow

Preserve the strongest properties from both Smalt and Otto:

- typed search models;
- predefined fields and operators;
- no model-built encoded-query syntax;
- explicit value character policy;
- rejection of encoded-query control tokens;
- no `sysparm_query_no_domain`;
- server-side query plus local post-filter where upstream parsing can broaden;
- required real filters for search tools;
- bounded limits;
- exact table/class allowlists;
- exact field projections;
- direct-get identity equality;
- knowledge visibility applied to both search and direct get;
- case tools disabled until table, fields, and row-visibility semantics are explicit;
- generic table access blocked from curated/reserved tables unless deliberately allowed.

### 23.2 Atlassian

Preserve or improve:

- typed CQL/JQL construction;
- structural characters quoted or rejected;
- site/cloud binding injected by the server;
- exact project/space allowlists;
- local scope post-filtering where useful;
- bounded search limits;
- summary-first, full-body-on-demand patterns;
- result identity verification;
- correct ADF parsing into safe text;
- constrained generic API paths;
- no arbitrary URL tool;
- no unrestricted method/body forwarding.

### 23.3 Escape-hatch tools

A generic API capability can be valuable, but it must be:

- separately authorized;
- method constrained;
- path allowlisted;
- origin pinned;
- response bounded;
- output redacted;
- disabled by default;
- clearly labeled as lower-level;
- unable to bypass reserved-table or write policy.

---

## 24. Write and destructive-operation safety

No mutation module should be ported until shared security state and confirmation semantics are ready.

### 24.1 Capability-specific gates

A single global `ALLOW_WRITES=true` flag is not acceptable.

Separate gates must exist for areas such as:

- ServiceNow ITSM;
- generic table writes;
- CMDB/IRE;
- approvals;
- knowledge;
- ServiceNow control app;
- Confluence content;
- Confluence permissions;
- Jira.

### 24.2 Confirmation binding

A confirmation should bind at least:

- authenticated caller subject;
- caller tenant;
- upstream tenant/origin;
- capability;
- tool name;
- normalized payload digest;
- preview/result version where relevant;
- expiration;
- unique nonce/audience.

It must be one-shot and transactionally spent.

### 24.3 Audit-before-mutation

An accepted write must be durably recorded before the outbound mutation begins.

The record should safely identify:

- caller;
- upstream acting identity;
- tenant/origin;
- tool/capability;
- target reference;
- confirmation reference;
- normalized payload digest;
- timestamp;
- correlation ID;
- intended operation class.

Do not record sensitive field values merely to make the audit convenient.

### 24.4 Uncertain outcomes

If a write request times out, loses its response, or receives an undecodable response after transmission:

- mark the mutation uncertain;
- do not report success or ordinary failure;
- do not blind-retry;
- retain enough metadata to reconcile;
- provide a read-only reconciliation operation where possible;
- let a human or deterministic workflow verify the real upstream state.

### 24.5 Idempotency

Where the vendor supports idempotency keys, use them deliberately.

Where it does not:

- use preconditions/version checks;
- record mutation intent;
- avoid hidden retries;
- provide reconciliation guidance.

---

## 25. Redaction and sensitive-data handling

Redaction must be applied at the last enforceable boundary and as defense in depth earlier.

Rules:

- no credentials in errors;
- no input values in config-validation output;
- no raw request or response body in logs;
- no search query strings in telemetry;
- no attachment content in logs;
- no titles/bodies/field values in mutation audit unless a reviewed mode explicitly permits it;
- field-level redaction policy applied to structured results;
- confirmation prompts constructed from already-redacted structured fields;
- exception tracebacks redacted at handler/exporter boundaries;
- diagnostics report metadata and counts, not content;
- HTTP URLs logged without query strings;
- secret-bearing environment output always masked.

Security tests must inject secrets and sensitive markers into every output channel and prove absence.

---

## 26. Attachment and file handling

Local stdio and remote HTTP deployments need different transport strategies.

### 26.1 Common controls

- maximum bytes;
- maximum inline bytes;
- allowed extensions;
- allowed content types;
- filename normalization;
- no absolute caller-selected paths;
- no traversal;
- symlink-safe handling;
- descriptor-relative confinement where local paths exist;
- content-type sniffing;
- malware scanning hooks;
- quarantine state;
- checksums;
- bounded decompression;
- safe temporary-file permissions;
- explicit retention and cleanup;
- audit metadata without content.

### 26.2 Local stdio

A local deployment may support explicitly configured upload/download roots, provided confinement is robust.

### 26.3 Remote HTTP

Server-filesystem paths are not a client contract.

Use:

- bounded inline content for small files;
- short-lived signed object handles for larger files;
- server-side object storage;
- authorization bound to caller and operation;
- expiration;
- one-time or least-use handles where practical.

Object storage must be an adapter, not a required specific cloud service.

### 26.4 Data retention and privacy

Every persisted data class needs an explicit owner, purpose, location, encryption policy, and retention period.

Inventory at least:

- security audit;
- mutation ledger;
- confirmation spend;
- OAuth tokens;
- attachment objects;
- temporary files;
- caches;
- background-task state;
- diagnostic artifacts;
- live-test fixtures.

Requirements:

- minimize stored vendor content;
- default caches to metadata or bounded projections;
- never persist secrets in logs or capability catalogs;
- support deletion by tenant/profile where multi-tenancy exists;
- document backup inclusion and restore behavior;
- ensure backups preserve encryption and retention requirements;
- separate security audit retention from operational telemetry;
- expose no “debug dump” that bypasses normal redaction;
- use synthetic data in public fixtures.

A privacy review is required before enabling telemetry or fixtures that include customer content.

### 26.5 Long-running operations

Exports, large attachment transfers, ServiceNow conflict checks, and some vendor jobs may outlive a normal request.

Before introducing background tasks, define:

- task identity;
- caller and tenant binding;
- cancellation;
- status polling/subscription;
- output retention;
- retry and idempotency;
- authorization on later status/result reads;
- process-restart behavior;
- distributed ownership;
- audit;
- cleanup.

FastMCP or MCP task features may be used through the adapter layer, but the task state machine belongs to OttoApps core and must remain testable without FastMCP.

---
