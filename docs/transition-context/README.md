# OttoApps transition context and planning brief

> **Status:** Authoritative planning context; not an implementation plan and not permission to perform destructive repository actions.  
> **Snapshot date:** 2026-08-10  
> **Primary destination:** [`martinthommesen/ottoapps`](https://github.com/martinthommesen/ottoapps)  
> **Legacy MCP source:** [`martinthommesen/smalt`](https://github.com/martinthommesen/smalt)  
> **Agent source and second migration source:** [`martinthommesen/otto`](https://github.com/martinthommesen/otto)  
> **Intended audience:** Coding agents, reviewers, security reviewers, and maintainers planning the transition.

---

## 1. Purpose of this document

This document gives a coding agent the context, constraints, evidence, target boundaries, risks, and required planning outputs for a clean transition from Smalt and the MCP-related parts of Otto into a new project named **OttoApps**.

The transition is not a normal framework upgrade and it is not a repository rename. It is a product succession and repository-boundary correction:

1. **Smalt ends as a legacy implementation and reference specification.**
2. **OttoApps becomes the sole project that owns MCP servers and vendor integrations.**
3. **Otto remains the Otto agent only.**
4. **All MCP and vendor-integration functionality currently living in Otto moves out of Otto and into OttoApps.**
5. **Otto talks to OttoApps as an MCP client instead of talking directly to Atlassian or ServiceNow.**
6. **OttoApps exposes exactly two public MCP server products:**
   - `oa-atlassian`
   - `oa-servicenow`

The immediate use of this document is to produce an evidence-based, issue-level implementation plan. A coding agent must not interpret it as permission to archive repositories, close issues, delete code, publish packages, move secrets, or perform a large migration without review.

---

## 2. Normative language and decision labels

The words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used in their normal requirements sense.

The following labels distinguish owner decisions from recommendations and unresolved questions:

- **DECIDED** — an owner decision that the plan must preserve.
- **RECOMMENDED** — the preferred architecture unless evidence supports a better design.
- **OPEN** — must be decided explicitly and recorded, normally in an ADR, before implementation reaches the affected phase.

Where this document conflicts with an explicit later owner decision, the later owner decision wins. A coding agent must record the conflict rather than silently choosing.

---

## Document map

This context is maintained as one canonical document set so every section can be reviewed and changed through the repository's pull-request rules. A coding agent **must read all eight parts below, in order**, before planning or implementation. The split is editorial only; all parts are normative together.

1. [Decisions and current state](01-decisions-and-current-state.md)
2. [Target architecture and contracts](02-target-architecture-and-contracts.md)
3. [Authentication and security](03-authentication-and-security.md)
4. [Operations, configuration, and packaging](04-operations-configuration-and-packaging.md)
5. [Testing and migration phases](05-testing-and-migration-phases.md)
6. [Otto extraction, issue disposition, and risks](06-otto-extraction-issues-and-risks.md)
7. [Definition of done and planning-agent instructions](07-definition-of-done-and-planning-instructions.md)
8. [Appendices, checklists, source evidence, and glossary](08-appendices.md)

The single-file source used to generate this split is tracked externally during this initial repository bootstrap with SHA-256 `393c17c16e1526b80928dfdf8437b6fffd1bf977533c4850375c19027b7f7dbf`. Once the repository permits a generated-document workflow, it may be rendered into one file without changing the normative content.
