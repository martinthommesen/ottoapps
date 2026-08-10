# Part VII — State, scale, observability, and operations

## 27. State abstractions

Define interfaces for:

- confirmation-spend storage;
- mutation ledger;
- security audit;
- caller/session state;
- delegated credential/token storage;
- metadata cache;
- attachment/object metadata;
- optional task/background-job state.

### 27.1 Local reference mode

A local/single-node installation should not require PostgreSQL merely to run read-only tools.

Appropriate local options may include:

- memory for non-security caches;
- SQLite with secure file permissions;
- locked local files for explicitly supported single-writer modes.

### 27.2 Distributed reference mode

For horizontal scaling and mutations, use a transactional shared store.

PostgreSQL is the recommended reference unless planning evidence selects another store.

Critical constraints include:

- unique confirmation-spend key;
- transactionally ordered audit/mutation intent;
- per-tenant partition keys;
- advisory or row locking where required;
- schema migrations;
- backup and restore;
- retention;
- encryption and access control;
- health/readiness behavior when the store is unavailable.

### 27.3 No false statelessness

An HTTP process is not stateless merely because FastMCP runs in stateless mode.

Confirmations, audit-before-write, delegated OAuth, and mutation reconciliation remain stateful concerns. Their state must be explicit.

---

## 28. Observability

OttoApps needs two distinct observability products:

1. **operational telemetry**
2. **security/mutation audit**

### 28.1 Operational telemetry

Provide:

- structured logs;
- metrics;
- optional OpenTelemetry traces;
- health and readiness;
- request/correlation IDs;
- tool latency;
- upstream latency;
- retry and rate-limit counters;
- cache metrics;
- active request/concurrency counts;
- error-code counts;
- dependency/version metadata.

Content must be excluded by default.

### 28.2 Security audit

Security audit should be:

- schema versioned;
- append-oriented;
- durable in production;
- access controlled;
- exportable;
- retention documented;
- able to distinguish caller and upstream identity;
- complete for accepted writes and abnormal terminations;
- resistant to silent sink failure;
- independently alertable.

The agent’s own audit remains in Otto. OttoApps’ audit covers MCP access, upstream access, confirmations, and mutations. Correlation identifiers should allow the two systems to be joined without sharing raw user identifiers unnecessarily.

---

## 29. Diagnostics and operator experience

Recommended executables:

```text
oa-atlassian
oa-servicenow
oa
```

Possible commands:

```text
oa config validate
oa capabilities
oa doctor atlassian
oa doctor servicenow
oa doctor state
oa doctor auth
oa audit query
oa mutation reconcile
oa version
```

Requirements:

- diagnostics are non-mutating unless a command is explicitly named and confirmed as mutating;
- offline validation performs no network call;
- live probes are bounded and content-free;
- output is structured and versioned;
- errors use stable codes;
- secrets are never echoed;
- `--print-config` is redacted;
- doctor and startup share the same validation code;
- a green doctor must not be contradicted by a different startup check;
- readiness must fail on fatal contract/identity/config problems;
- transient upstream outages should be distinguishable from bad configuration.

---

# Part VIII — Configuration ownership

## 30. Configuration matrix after the split

| Configuration | Otto | OttoApps | Notes |
|---|---:|---:|---|
| Webex bot token | Yes | No | Agent transport |
| LLM provider/gateway keys | Yes | No | Agent model runtime |
| Agent model classification ceiling | Yes | No | Agent egress policy |
| Agent caller admission/domain/org policy | Yes | No | Agent audience |
| Agent group-room delivery policy | Yes | No | Agent disclosure boundary |
| Agent audit HMAC/key | Yes | No | Agent audit only |
| OttoApps endpoint(s) | Yes | No | Consumer configuration |
| OttoApps caller token/client identity | Yes | Server validates | Credential is for Otto → OA |
| Atlassian site URL/cloud ID | No | Yes | Upstream tenancy |
| Atlassian API token/OAuth | No | Yes | Upstream credentials |
| Confluence space allowlist | No | Yes | Server/source policy |
| Jira project allowlist | No | Yes | Server/source policy |
| ServiceNow instance URL | No | Yes | Upstream tenancy |
| ServiceNow username/password/API key/OAuth | No | Yes | Upstream credentials |
| ServiceNow tables/classes/fields | No | Yes | Server/source policy |
| Knowledge visibility | No | Yes | Source policy |
| Case table/visibility | No | Yes | Source policy |
| Write gates | No | Yes | Server mutation policy |
| Confirmation policy/store | No | Yes | Server mutation policy |
| Attachment storage/root | No | Yes | Server file policy |
| Upstream concurrency/retry | No | Yes | Server operation |
| Otto agent source classification override | Possibly | Supplies base classification | Define precedence explicitly |
| Conversation memory | Yes | No | Agent state |
| Agent delivery rate limits | Yes | No | Agent boundary |
| MCP caller rate limits | No | Yes | Server boundary |

### 30.1 Classification ownership

**OPEN**

OttoApps should return trusted source classification metadata based on its policy. Otto still needs model-egress and destination-delivery ceilings.

Define whether Otto may:

- accept the OttoApps classification as authoritative;
- apply only a stricter local override;
- map OA labels into Otto labels.

Otto must never downgrade a higher classification supplied by OttoApps.

---

## 31. Configuration format

Smalt’s env-only model is portable and safe for simple deployments, but enterprise/multi-tenant operation can make large allowlists unwieldy.

**RECOMMENDED**

- Single-tenant reference deployments remain environment-driven and fail closed.
- Secrets come from environment, mounted secret files, or secret-provider adapters—not committed YAML.
- Non-secret structured policy may optionally use a typed config file.
- Unknown fields are errors.
- Empty values normalize to missing.
- Broad scope requires explicit acknowledgement.
- Every config source and precedence rule is documented.
- No mutable token cache appears by accident.
- Config schemas are generated and tested.

**OPEN**

Decide whether OttoApps 1.0 supports only env configuration or env plus a structured non-secret policy file.

---

# Part IX — Packaging, licensing, provenance, and supply chain

## 32. Packaging recommendation

**RECOMMENDED**

Use one Python distribution named `ottoapps` that installs:

```text
oa-atlassian
oa-servicenow
oa
```

Benefits:

- one version train;
- one shared core;
- no internal package publishing dance;
- separate process boundaries still preserved;
- simpler dependency and vulnerability management.

Do not publish a package per capability unless a proven external embedding use case requires it.

### 32.1 Python baseline

**OPEN**

The prototype used Python 3.11+, while Otto uses Python 3.12.

Recommended starting point:

- Python 3.12 as the minimum;
- verify 3.13 support in CI if FastMCP and dependencies support it;
- document the supported range;
- do not widen the range without tests.

### 32.2 FastMCP version policy

FastMCP 4 is the intended MCP delivery layer.

Requirements:

- verify the current stable/prerelease state at implementation time;
- pin a tested compatible range and lock exact resolved versions;
- no unbounded prerelease lower bound;
- record MCP protocol versions tested;
- isolate FastMCP imports;
- maintain contract tests through real MCP clients;
- have an upgrade runbook and compatibility matrix.

---

## 33. Public-repository safety

`ottoapps` is public while source repositories are private. Treat migration as a publication event.

Do not:

- push either private repository’s history wholesale;
- copy `.env` files;
- copy live tenant URLs;
- copy user/service-account identifiers;
- copy private issue content that contains sensitive details;
- copy real vendor response bodies;
- copy internal business data;
- copy secrets that were later deleted from source history;
- publish proprietary Otto code without a license decision.

Before every initial source import:

1. identify source file and commit;
2. confirm license/provenance;
3. scan content;
4. replace real fixtures with synthetic fixtures;
5. run secret scanners;
6. run privacy/internal-identifier checks;
7. record provenance.

Create a `PROVENANCE.md` or equivalent manifest.

### 33.1 License

Smalt is Apache-2.0. Otto is marked proprietary.

**OPEN, BLOCKING**

Before selected Otto code is copied into OttoApps:

- the owner must explicitly authorize relicensing or reimplementation;
- copyright and license notices must be correct;
- third-party derived code must be reviewed;
- the new repository must choose and add a license;
- dependency licenses must be audited.

Apache-2.0 is the natural default because Smalt already uses it, but this remains an explicit owner decision for OttoApps.

---

## 34. Supply-chain requirements

Before public stable release:

- lock Python dependencies;
- pin GitHub Actions by commit SHA;
- pin container bases by digest;
- generate an SBOM;
- scan Python and OS dependencies;
- scan secrets;
- run static security analysis;
- generate build provenance;
- sign release artifacts and images where practical;
- use trusted publishing/OIDC rather than long-lived publish tokens;
- document vulnerability severity gates and exceptions;
- publish checksums;
- maintain a security policy;
- define supported versions.

No package or image should be published from Smalt’s pending release workflow under the old identity unless explicitly required for archival purposes.

---
