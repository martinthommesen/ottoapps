# Part IV — MCP, tool, and compatibility contracts

## 13. Supported transports

Each public server should support:

1. **stdio**
   - local desktop agents and coding harnesses;
   - credentials supplied through environment/process isolation;
   - no HTTP authorization layer required by default;
   - protocol traffic only on stdout;
   - diagnostics and logs only on stderr.

2. **Streamable HTTP**
   - remote hosting;
   - standards-based caller authentication;
   - stateful or stateless modes only where semantics are correct;
   - health/readiness endpoints outside the MCP message stream;
   - reverse-proxy-safe request handling.

3. **Stateless HTTP**
   - only after all request/session assumptions are audited;
   - no in-memory confirmation state relied upon across calls;
   - no in-memory caller state required for correctness;
   - shared state stores used for replay protection and mutations.

Legacy transports should be added only when there is a concrete compatibility requirement.

---

## 14. Capability discovery and large catalogs

Combining thirteen servers into two can expose more than one hundred tools per server. That is an operational and model-context problem, not a reason to restore thirteen public servers.

Use multiple controls:

- caller authorization scopes;
- deployment capability profiles;
- component-level visibility;
- role recipes;
- server-side tool tags;
- optional tool-search transforms;
- resources/prompts represented as tools only for clients that need that compatibility mode;
- explicit `OA_ENABLED_COMPONENTS` or equivalent;
- split deployments of the same server binary for high-assurance roles.

Rules:

- Unauthorized components must be absent from discovery, not merely rejected at execution.
- Direct invocation of a hidden component must still be rejected.
- No client may gain capabilities because it omits a scope claim.
- Client brand must not select the catalog.
- A tool-search abstraction must not make direct tools unavailable to standards-compliant clients.
- Experimental transforms must be optional until proven stable.

---

## 15. Tool contract quality

Every public tool should have:

- a stable, descriptive name;
- a concise model-facing description;
- a strict typed input model;
- `extra="forbid"` or equivalent at trust boundaries;
- bounded strings, arrays, and limits;
- a strict structured output model;
- explicit access classification;
- explicit retryability;
- deterministic pagination;
- safe, typed error results;
- no raw secret or unrestricted vendor payload;
- examples in generated documentation;
- an owner and capability module;
- contract tests over real MCP sessions.

### 15.1 Domain failures versus protocol failures

Expected domain failures should be returned as MCP tool error results with stable machine-readable codes.

Examples:

- `policy.refused`
- `auth.authentication_failed`
- `auth.authorization_failed`
- `resource.not_found`
- `input.invalid`
- `upstream.rate_limited`
- `upstream.timeout`
- `upstream.contract_drift`
- `mutation.uncertain`
- `confirmation.required`
- `confirmation.declined`
- `confirmation.expired`
- `confirmation.already_spent`

Protocol errors are reserved for malformed MCP/session behavior, not a missing incident or denied write.

### 15.2 Output validation

A tool handler must not return content until:

1. the upstream response has been bounded;
2. the vendor payload has been validated;
3. the result has been projected into the public DTO;
4. sensitive fields have been redacted;
5. the public output schema has been validated;
6. the result metadata has been attached.

Redaction must cover:

- structured tool results;
- text content;
- confirmation prompts;
- receipts;
- logs;
- tracing attributes;
- errors.

### 15.3 Versioning and deprecation

Public MCP components are APIs.

Define:

- server semantic-version policy;
- capability-catalog schema version;
- tool-schema compatibility rules;
- resource/prompt compatibility rules;
- supported MCP protocol versions;
- minimum client expectations;
- deprecation period;
- removal process;
- migration aliases, if ever justified.

A patch release must not silently:

- remove a tool;
- make an optional field required;
- narrow an enum incompatibly;
- change result identity;
- broaden a query;
- arm writes;
- weaken redaction;
- change confirmation semantics.

The server should expose version and catalog metadata in a machine-readable, authorization-safe form. Otto’s consumer contract should declare a compatible range rather than assume “latest.”

Smalt issue #484 is a specific warning: masking only final structured results is insufficient if confirmation prose was built from raw values first.

---

## 16. Resources, prompts, and URI identity

Replace `smalt://` resources with `oa://` resources before stable release.

Recommended base resources:

- `oa://policy`
- `oa://capabilities`
- `oa://identity`
- `oa://health` only if it does not expose operationally sensitive data;
- backend-specific templates such as `oa://atlassian/confluence/page/{id}`;
- backend-specific templates such as `oa://servicenow/record/{table}/{sys_id}`.

Resource contents must obey caller authorization and redaction.

Prompts inherited from Smalt should be inventoried. They must either:

- be ported and tested;
- be replaced by better prompts with a migration note;
- or be intentionally retired with rationale.

Prompts must not introduce a model dependency into the server. They are templates exposed over MCP.

---

## 17. Otto-to-OttoApps consumer contract

Otto should depend on a deliberately small subset of the full OttoApps surface.

### 17.1 Recommended contract approach

Otto keeps its current canonical model-facing tools and calls general OttoApps tools underneath.

Illustrative mapping:

| Otto canonical tool | Likely OttoApps operation |
|---|---|
| `confluence_search` | `confluence_search` |
| `confluence_get_page` | `confluence_page_get` |
| `servicenow_search_incidents` | `snow_incident_list` or a deliberate `snow_incident_search` contract |
| `servicenow_get_incident` | `snow_incident_get` |
| `servicenow_search_knowledge` | `snow_knowledge_search` |
| `servicenow_get_knowledge_article` | `snow_knowledge_get` |
| `servicenow_search_cases` | a scoped generic-table or curated CSM operation |
| `servicenow_get_case` | the matching scoped get operation |

The exact mapping is **OPEN** and must be based on schema/semantic comparison.

### 17.2 Structured source-record DTO

Otto should not parse vendor payloads. OttoApps should return enough typed metadata for Otto to create evidence safely, for example:

```text
SourceRecord
- source
- tenant_ref
- record_type
- record_id
- display_id
- title
- summary or body
- classification
- canonical_url
- source_version
- source_updated_at
- truncation/body-omitted metadata
- policy metadata safe for the caller
```

This is illustrative, not a mandated schema.

### 17.3 Consumer contract verification

At startup and in `otto doctor`, Otto should verify:

- server identity;
- protocol compatibility;
- required tool presence;
- required input fields;
- output-schema compatibility or fingerprints;
- server capability/catalog version;
- the expected tenant/profile when relevant;
- caller authorization sufficient for the narrow read profile;
- one bounded read-only probe.

An additive unrelated tool must not fail Otto startup. A breaking change in a required tool must.

### 17.4 No Python package coupling

Do not solve contract sharing by adding:

```text
otto depends on ottoapps Python package
```

That would collapse the repository boundary and couple release trains.

Share contracts through:

- MCP schemas;
- versioned checked-in JSON Schema/OpenAPI-like artifacts;
- generated consumer fixtures;
- a tiny separately governed protocol-contract package only if later evidence justifies it.

The default should be protocol artifacts, not runtime imports.

---
