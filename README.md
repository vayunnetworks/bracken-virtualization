# QEMU/KVM integration — BRACKEN Engineering Guide

> BRACKEN integration with QEMU and KVM for virtual machine lifecycle, resource management, storage, and virtual networking.

> **Status:** Early-stage design and implementation. This guide expands the intended repository scope; verify the code and release notes before relying on any described capability.

# BRACKEN Virtualization

**Virtual machine lifecycle and host integration for BRACKEN.**

BRACKEN Virtualization is intended to integrate Linux hosts with QEMU and KVM so operators can provision and manage isolated virtual machines alongside BRACKEN network services. It will use established virtualization components rather than implement a hypervisor.

> **Project status:** Early-stage project. Supported host configurations, guest images, isolation profile, and lifecycle API are not yet defined.

## Responsibilities

- Define host prerequisites and virtualization capabilities.
- Create, inspect, start, stop, and remove virtual machines through supported interfaces.
- Manage virtual CPU, memory, storage, and network attachments.
- Integrate with the BRACKEN control plane for desired-state workflows.
- Document VM isolation boundaries, resource limits, and operational status.
- Support repeatable local development and integration testing.

## Scope boundaries

This repository owns BRACKEN-specific integration and lifecycle management. QEMU/KVM remain external system components. Network policy and forwarding belong to the [network engine](https://github.com/vayun-networks/bracken-network-engine) and [data plane](https://github.com/vayun-networks/bracken-data-plane). The [build system](https://github.com/vayun-networks/bracken-build) owns producing BRACKEN images; this repository consumes documented image artifacts.

## Design principles

- Use least privilege for host-side management processes.
- Make resource allocation and guest state explicit.
- Treat guest networking and storage as security boundaries.
- Make lifecycle actions idempotent where practical and expose failures clearly.
- Avoid promising live migration, clustering, or high availability until implemented and validated.

## Planned milestones

1. Document supported Linux host and hardware virtualization requirements.
2. Define a minimal VM lifecycle interface.
3. Provision a test guest with controlled virtual networking.
4. Add lifecycle reporting, resource constraints, and integration with the control plane.
5. Validate isolation and failure handling in an isolated test environment.

## Development

No commands or tested host matrix are available yet. Development guidance should state required kernel features, QEMU/KVM versions, permissions, and safe cleanup procedures once selected. Never run untrusted guest workloads on a host without an appropriate isolation review.

## Related components

- [bracken-control-plane](https://github.com/vayun-networks/bracken-control-plane)
- [bracken-network-engine](https://github.com/vayun-networks/bracken-network-engine)
- [bracken-build](https://github.com/vayun-networks/bracken-build)
- [bracken-deploy](https://github.com/vayun-networks/bracken-deploy)

## 1. Project purpose and current status

BRACKEN is a proposed open-source infrastructure platform built around a Linux-based host environment and a set of cooperating network services. Its long-term scope includes system images, packet processing, management, virtualization, DNS, edge delivery, network security, traffic intelligence, observability, and deployment automation. This repository owns the component described in its title and participates in the broader platform through documented interfaces.

The project is at an early design and implementation stage. This README is an engineering blueprint for contributors, reviewers, and operators. It is not evidence that every listed capability already exists. Unless the source tree contains a working implementation and validation evidence, requirements below should be read as proposed behavior, not a present guarantee. The repository must update this document as decisions are made and as code changes the actual scope.

The current project charter sets a 12-month target for an initial integrated release. That target is ambitious and should be treated as planning guidance. A first release should favor a small set of implemented, measurable, and supportable workflows over a broad list of unverified features. Production readiness, global scale, availability commitments, and performance figures require separate evidence.

## 2. Component objective and scope

**Component domain:** QEMU/KVM integration.

**Objective:** Manage virtual machine lifecycle, resources, images, storage, and virtual network attachments on supported Linux hosts..

**Boundary:** Integrate established hypervisor components; consume built images and coordinate with control and network services without implementing a hypervisor.

This component should own the behavior closest to its domain and provide a narrow contract to neighboring components. It should not make changes to unrelated layers through hidden side effects. A new feature proposal should identify the operator problem, the owning service, the inputs and outputs, the dependencies, and the recovery behavior before implementation starts.

**Primary areas to define:**

- host capability discovery
- VM definition and validation
- resource admission
- disk lifecycle and persistence
- virtual network attachment
- restart reconciliation and cleanup

These areas describe an intended scope, not a promise that every item is implemented. The maintainers should mark each item as planned, in progress, experimental, supported, or retired when the repository begins tracking implementation status.

## 3. BRACKEN system context

The platform separates management, node execution, and release engineering. The control plane is intended to accept operator intent, validate it, coordinate rollout, and expose status. The node components perform the work close to the host or traffic path. The build system assembles versioned components into images. Deployment automation installs and configures those images. Shared contracts should live in bracken-core only when multiple consumers need the same semantics.

This repository should be understandable by itself, but it must not silently redefine a platform-wide concept. A node identity, configuration revision, health status, error code, policy version, or artifact version should mean the same thing across the components that exchange it. If a common definition is missing, propose it and document the temporary boundary. Do not make an undocumented assumption about another service's storage format, log message, command line, or internal data structure.

For each dependency, record whether it is required at build time, startup time, runtime, or only for optional features. Avoid making a single failed analytics or dashboard component prevent the core service from performing its job. Conversely, if a security or policy dependency is required for safe operation, report the system as not ready rather than hiding the failure.

## 4. Proposed architecture and responsibility model

The architecture should be layered and testable. Separate input validation, domain decisions, state transitions, external side effects, and reporting. This allows developers to test decision logic without a full deployment and to test host or network integration in an isolated environment. Keep domain data structures distinct from transport-specific payloads where a stable interface requires it.

For operations that change external state, use an explicit sequence: accept a request, validate it, calculate or stage the intended change, perform the change, verify the observed result, and record completion or failure. The operation should identify the request and revision. A retry must be safe or must explain why it is not safe. If a change can be only partially applied, report the completed portion and provide an operator recovery path.

The implementation should preserve a clear source of truth. Desired state is what an operator or controller requested. Observed state is what the component found on the host or service. Do not overwrite the requested state with observations, or report success merely because a request was accepted. Where there is no controller, document which local file or database is authoritative and how it is edited safely.

Avoid premature distribution. A single-node implementation with a crisp contract is a better foundation than a multi-region or multi-controller design whose consistency and recovery rules are unspecified. Add concurrency, clustering, caching, and high availability only when the workload and failure model justify them. Each optimization must preserve understandable status and recovery.

## 5. Configuration and contract design

Configuration should be validated before it can affect the system. For every field, document its meaning, type, unit, default, allowed range, whether it is optional, who owns it, whether it contains a secret, and what happens when it changes. Unknown or unsupported fields should produce deliberate behavior; they must not be silently ignored if that could create a security or availability mismatch.

Interfaces should define versioning, authentication, authorization, idempotency, error response, timeout, and compatibility. For asynchronous work, return or expose an operation identifier and a finite set of states. Distinguish accepted, queued, applying, succeeded, failed, partially applied, and rolled back where those states make sense. A transport-level success response must not be confused with successful application of the requested change.

Settings that represent secrets should contain references to an approved secret source rather than plaintext. Logs, metrics labels, traces, examples, test fixtures, and error responses must not reveal the value. Configuration migrations need a documented order and compatibility window so older nodes can report that they cannot apply a new revision.

## 6. Lifecycle and operational behavior

Document startup prerequisites, readiness conditions, shutdown behavior, restart behavior, update behavior, and recovery. Define what the component does when a required dependency is missing, a configuration is invalid, a resource limit is reached, or a process is interrupted. A service should not report ready until it can perform the core task it advertises.

A lifecycle operation should make side effects visible. Before an operation, show its scope and prerequisites. During it, expose progress if it is long-running. After it, record result, revision, and relevant measurements. On cancellation or timeout, state whether the operation stopped, continued in the background, or left an uncertain result. Reconciliation after restart should discover actual state rather than blindly repeat a destructive action.

Runbooks should use safe inspection steps before remediation. Each procedure should state required privileges, expected output, and conditions that mean the operator should stop. Destructive actions should require a clear scope and backup or rollback explanation. The project should prefer actionable diagnostics over silently retrying an operation forever.

## 7. Failure handling and recovery

Plan for invalid input, missing permissions, exhausted disk or memory, unavailable dependencies, timeout, process restart, partial update, stale state, incompatible versions, and corrupted or missing state. For this component, additional domain risks include:

- host privilege exposure
- resource exhaustion
- orphaned disks
- guest isolation gaps
- ambiguous deletion semantics

For each failure, specify detection, externally visible status, whether a retry is safe, how the system protects existing good state, and how recovery is performed. Use bounded retries with backoff for transient failures. Do not retry permanent validation errors. Do not silently downgrade security behavior when an operation fails.

When an action has affected only part of the system, report which targets changed and which did not. Define whether the component can roll back automatically, requires operator action, or preserves last-known-good state. Keep enough history to support diagnosis while respecting data-retention and privacy requirements.

## 8. Security and privacy expectations

Write a component threat model before production use. List trust boundaries, exposed listeners, privileged actions, secret sources, untrusted data, dependencies, and the impact of compromise. Apply least privilege to processes and APIs. Validate input at every trust boundary. Require authentication and authorization for administrative operations. Record security-relevant actions with actor and revision while excluding secret values.

If this component handles network metadata, document what it collects, why collection is necessary, how long data is retained, who can read it, and whether it can be aggregated or redacted. Avoid logging packet payloads or customer content unless there is a deliberate, reviewed need. Secure examples should use reserved or synthetic values and must never include production credentials.

Security fixes should identify affected versions and supported upgrade paths. A scan result is evidence to review, not proof that the component is secure. Security claims must identify the tested configuration and known gaps. Use isolated labs and authorized traffic for validation; never test disruptive behavior against third-party systems.

## 9. Observability and supportability

Define health, readiness, and degraded states separately. Health should tell whether the process is alive; readiness should tell whether the core function is available; degraded status should identify a reduced but still useful mode. Export measurements with stable names, units, and bounded dimensions. Avoid unique IDs as metric labels because they can create unbounded time series.

Logs should include timestamp, severity, component, operation or correlation identifier, and configuration revision when relevant. They should identify the failed step and a safe next action. Traces should preserve context across service boundaries without recording sensitive request contents. Metrics, logs, and traces should complement each other instead of duplicating large amounts of data.

An operator should be able to answer what was requested, what changed, which revision is active, what failed, how broad the impact is, whether the component is still safe to use, and what recovery action is appropriate. These questions should shape the status API and dashboards from the start.

## 10. Validation and test strategy

Use several layers of validation. Unit tests cover deterministic logic, boundary conditions, and invalid input. Schema or API contract tests protect compatibility. Integration tests exercise actual host, kernel, database, or network dependencies in a controlled environment. End-to-end tests validate complete workflows. Performance tests use declared hardware and realistic workload shape.

At minimum, tests should cover: normal operation; invalid or missing configuration; permission failure; dependency unavailable; duplicate request or retry; process restart; resource exhaustion; partial operation; recovery; and version mismatch. The exact cases should be tailored to this component's risks. Tests must prove observable outcomes, not just that a function returned without error.

Test fixtures should be synthetic and safe to publish. Any traffic generation, VM workload, or DNS/HTTP load should target an isolated lab or infrastructure under the tester's control. Performance reports should include environment, software revisions, load shape, duration, measurement method, errors, and resource use. Do not turn a single benchmark into a broad product guarantee.

## 11. Compatibility, deployment, and releases

State the supported component versions and dependency versions once they are known. Keep compatibility promises realistic. A change that alters configuration, API behavior, resource ownership, or failure semantics needs a migration note. The build should identify exact source revisions. A release should include its changes, known limitations, security notes, and tested environments.

Deployment instructions should be tested from a clean environment. Separate development setup from production operation. Document privileges and ports, data directories, resource requirements, dependencies, health checks, upgrades, rollback, and uninstall behavior. If an upgrade is not yet supported, state that plainly rather than implying that replacing a binary is safe.

BRACKEN intends to add reproducible builds, manifests, SBOMs, artifact verification, and signing. These should be listed as roadmap items until a complete release pipeline validates them. Do not put private keys or release tokens in this repository. A release artifact should be traceable to its source and build inputs.

## 12. Implementation roadmap

A sensible component roadmap has four gates. First, document the domain model, trust boundaries, and smallest supported workflow. Second, implement that workflow with deterministic tests and a local development setup. Third, integrate with one neighboring component using a versioned contract and observable failure behavior. Fourth, validate operational recovery, compatibility, and performance in a controlled environment.

Do not advance a feature from experimental to supported merely because it works once. Define what “supported” means: tested environments, failure handling, security review, upgrade path, documentation, and an owner for maintenance. For capabilities that need hardware or traffic scale, state the lab limitations and what remains untested.

The roadmap is a set of engineering milestones, not a guarantee of dates. Prioritize the smallest end-to-end vertical slice that proves the architecture. Defer optional acceleration, clustering, machine learning, global distribution, or advanced automation until the basic behavior is measurable and safe.

## 13. Contributor workflow and review checklist

Before implementation, identify the intended user, specific problem, component owner, contract impact, security implications, failure behavior, and test plan. During review, ask whether the feature has one source of truth, whether it can be safely retried, whether partial failure is visible, whether defaults are safe, and whether an operator can reverse or diagnose the change.

A pull request should explain the motivation, design choice, alternatives, compatibility impact, test evidence, and documentation updates. Keep changes focused. Avoid adding a dependency without evaluating maintenance, licensing, security, build, and runtime effects. Do not include generated artifacts unless the project intentionally versions them. Update examples whenever behavior changes.

Design decisions that affect more than one repository should be recorded and linked from the relevant documentation. If a choice is still open, preserve it as an open question with tradeoffs instead of implying consensus. This protects future contributors from treating a provisional implementation detail as a platform contract.

## 14. Detailed component brief

The component-specific sections below expand the project charter into implementation concerns, review scenarios, and acceptance criteria. They remain proposals until code and tests demonstrate them.


## Component-specific engineering brief

### Domain objective
Manage virtual machine lifecycle, resources, images, storage, and virtual network attachments on supported Linux hosts.

### Ownership boundary
Integrate established hypervisor components; consume built images and coordinate with control and network services without implementing a hypervisor.

### Workstreams
- **Host capability discovery:** Define its owner, inputs, outputs, failure behavior, and validation evidence.
- **VM definition and validation:** Define its owner, inputs, outputs, failure behavior, and validation evidence.
- **Resource admission:** Define its owner, inputs, outputs, failure behavior, and validation evidence.
- **Disk lifecycle and persistence:** Define its owner, inputs, outputs, failure behavior, and validation evidence.
- **Virtual network attachment:** Define its owner, inputs, outputs, failure behavior, and validation evidence.
- **Restart reconciliation and cleanup:** Define its owner, inputs, outputs, failure behavior, and validation evidence.

### Design rationale
The workstreams above should be developed as a coherent component rather than as unrelated features. For each workstream, identify the input state, the component's decision, the side effect, the reported output, and the dependency it relies on. Keep local implementation details behind a documented boundary so other repositories can use stable behavior without depending on internal modules. Where a behavior belongs to another repository, this component should publish the required input or request and consume an explicit result instead of reimplementing the behavior.

### End-to-end operating model
A complete workflow for this component should begin with a known input revision and a validated environment. The service should discover prerequisites, reject unsupported conditions early, and distinguish a rejected request from a request that was accepted but could not be applied. The action should be traceable from request through execution to observed result. If the component has no operator API, the same principles apply to its file configuration, command line, package lifecycle, or build job.

The component should expose the current version of its configuration and the version of the software that interprets it. If the desired configuration changes while work is in progress, define whether the old operation is cancelled, completed, or superseded. Prevent duplicate side effects when an operator retries after a timeout. If retry safety cannot be guaranteed, return an uncertain state and provide an inspection procedure before retrying.

### Contract and compatibility review
For every interface, record the consumer, producer, version, authentication boundary, data sensitivity, timeout, retry behavior, error mapping, and deprecation path. Contracts include more than network APIs: package names, files, schemas, command outputs, metrics, kernel interfaces, image formats, and release artifacts can all become dependencies. Avoid making an undocumented contract stable by accident.

When a contract changes, explain which old versions remain supported and how a staged rollout behaves when producers and consumers upgrade at different times. If a node is offline during a migration, it should report why it cannot apply a revision and how it can recover. Unknown fields, missing fields, duplicate operations, and malformed values should all have deliberate outcomes. Backward compatibility should not weaken security checks.

### Resource and capacity model
Identify the resources this component consumes: CPU, memory, disk, file descriptors, kernel objects, network connections, queues, or external service capacity. Define which resources are bounded and how exhaustion is reported. If the component allocates resources for another service or user, define admission control and quota behavior. A request that cannot fit should be rejected before partial work when possible.

Capacity measurements should use representative workloads and include headroom assumptions. Document which resource can grow with the number of nodes, flows, rules, zones, objects, or virtual machines. Avoid unbounded in-memory state and unbounded metric labels. If a component depends on local disk, specify retention and cleanup behavior. If it depends on kernel or NIC resources, state which limits are discovered dynamically and which are configured.

### Failure and recovery review
For every side effect, determine how the component detects completion and how it recovers after a crash between the action and status recording. Preserve idempotency keys or operation journals when needed. If external state cannot be rolled back, make that limitation explicit and surface the exact point at which an operator may need to intervene. Do not mark a component ready while required state is unknown.

Recovery guidance should begin with read-only inspection. It should identify the expected status, relevant revision or object identity, and what evidence to collect. Remediation should be ordered from least disruptive to most disruptive. Any step that deletes persistent state, changes routing, removes a virtual machine, changes a signing key, or applies a traffic mitigation must state its impact and preconditions. If automatic recovery is safe, explain the bound on retries and how an operator can stop it.

### Security review checklist
Review process privileges, administrative endpoints, service-to-service identity, secret references, file permissions, untrusted input parsing, dependency provenance, audit coverage, data retention, and safe defaults. Document whether compromise could affect only one node, all managed nodes, traffic forwarding, stored content, DNS zones, virtual machines, or release artifacts. Put the highest-impact controls at the component boundary rather than relying on a dashboard warning.

Tests should verify rejection of unauthorized actions, handling of malformed input, redaction of secret values, and safe behavior when a trusted dependency is unavailable. An operator should be able to distinguish a security rejection from an internal error without receiving sensitive information. Vulnerability reports should have a private disclosure route once maintainers establish one; do not promise a response-time SLA unless the project adopts it.

### Component-level verification plan
Validation should include a clean setup, a normal workflow, a boundary condition, at least one injected failure, a restart or upgrade, and a recovery path. Test the contract with a consumer or a realistic fixture. Test that observed state reflects the actual external result. A unit test that mocks every dependency cannot establish integration correctness on its own.

For a performance-sensitive behavior, define the objective before optimizing and record the baseline. Measure the error rate and tail latency in addition to average throughput. For stateful behavior, test restart recovery and stale-state reconciliation. For security-sensitive behavior, test both the positive path and a deny path. For an installation or build workflow, test from a clean environment without relying on untracked local files.

### Operational readiness criteria
A component is ready for limited evaluation when its setup is reproducible, its core workflow is documented, its failure state is visible, and an operator can recover from the known failure cases. It is not production-ready until the supported environment, threat model, upgrade path, resource limits, monitoring signals, and ownership model have been reviewed. The readiness statement should identify what remains untested.

Before release, maintainers should verify that the release notes match implemented behavior, dependencies are pinned or otherwise auditable, configuration examples are safe, secrets are absent, and the package or image identifies its source revision. If the component integrates with another repository, validate the intended version combination and document incompatibilities. Never present a planned roadmap item as a release feature.

### Review scenarios
- **Host privilege exposure:** Specify how this condition is detected, how the component reports it, what state is preserved, whether retry is safe, and which recovery action is available. Add a deterministic test so the behavior remains stable as implementation changes.
- **Resource exhaustion:** Specify how this condition is detected, how the component reports it, what state is preserved, whether retry is safe, and which recovery action is available. Add a deterministic test so the behavior remains stable as implementation changes.
- **Orphaned disks:** Specify how this condition is detected, how the component reports it, what state is preserved, whether retry is safe, and which recovery action is available. Add a deterministic test so the behavior remains stable as implementation changes.
- **Guest isolation gaps:** Specify how this condition is detected, how the component reports it, what state is preserved, whether retry is safe, and which recovery action is available. Add a deterministic test so the behavior remains stable as implementation changes.
- **Ambiguous deletion semantics:** Specify how this condition is detected, how the component reports it, what state is preserved, whether retry is safe, and which recovery action is available. Add a deterministic test so the behavior remains stable as implementation changes.

### Initial acceptance criteria
A first useful milestone for this component should be narrow enough to test end to end. It should have a documented input, a deterministic decision or action, visible output, a failure case, and an operator recovery path. The milestone should avoid optional features that make the core contract ambiguous. The component should be identifiable in a BRACKEN image or node deployment without relying on undocumented manual setup.

A later milestone may add more protocols, backends, hardware, scale, automation, or distribution. Each addition should preserve backward compatibility or include a migration plan. The release should record test environment and limitations. Performance results should not be generalized beyond the test conditions. Security properties should be stated as specific controls that were tested, not as broad claims that the component is secure.

### Open questions and design records
The component-specific risks listed in the overview should become tracked design questions. For each open decision, record options, evaluation criteria, expected consumers, operational consequences, and the point at which the decision must be made. Some choices can remain open while a prototype is evaluated; choices that affect data format, security boundary, stored state, or inter-repository contracts should be resolved before a stable release.

Design records should be concise and link to the code or tests that implement the decision. When evidence changes, update the record rather than preserving a stale rationale. A decision should name what it does not solve, and should identify a review trigger such as a new hardware target, a scale threshold, a security finding, or a second implementation backend.| Workstream | Expected evidence before calling it supported |
| --- | --- |
| host capability discovery | Documented behavior, a validation case, visible status, and a recovery note appropriate to this workstream. |
| VM definition and validation | Documented behavior, a validation case, visible status, and a recovery note appropriate to this workstream. |
| resource admission | Documented behavior, a validation case, visible status, and a recovery note appropriate to this workstream. |
| disk lifecycle and persistence | Documented behavior, a validation case, visible status, and a recovery note appropriate to this workstream. |
| virtual network attachment | Documented behavior, a validation case, visible status, and a recovery note appropriate to this workstream. |
| restart reconciliation and cleanup | Documented behavior, a validation case, visible status, and a recovery note appropriate to this workstream. |

### Validation matrix

| Validation area | Questions to answer |
| --- | --- |
| Clean environment | Can a new contributor reproduce the documented setup without hidden local state? |
| Normal workflow | Does the component complete its smallest useful workflow and report the active revision? |
| Invalid input | Are malformed, missing, unsupported, and out-of-range values rejected with actionable diagnostics? |
| Permission boundary | Are unauthorized users and processes unable to perform privileged actions? |
| Dependency outage | Does the component report the missing dependency and avoid unsafe behavior? |
| Duplicate request | Does retry after a timeout avoid duplicate or destructive side effects? |
| Resource pressure | Are capacity limits visible, bounded, and recoverable? |
| Restart | Does the process reconcile external state after an unexpected restart? |
| Compatibility | Are producer/consumer and old/new configuration combinations tested? |
| Recovery | Can an operator inspect and recover without losing unrelated state? |
| Performance | Are workload, hardware, duration, errors, and measurement method recorded? |
| Release | Can an artifact be tied to source revision and validated configuration? |

These questions are a minimum framework, not a substitute for domain-specific cases. The component maintainer should add tests for the failure modes and protocols that are unique to the implementation. When a test cannot run in normal continuous integration, document the lab environment and how often the manual validation is repeated.

### Troubleshooting sequence

1. **Identify the exact version.** Record component version, build revision, configuration revision, host environment, and the time the behavior began. Without this context, reports from different builds can be confused.
2. **Inspect status before changing state.** Compare desired and observed state, dependency health, resource limits, and the last operation result. Preserve logs and counters needed to understand the failure.
3. **Check recent changes.** Review the configuration and release history. Determine whether the failure began after an image, policy, dependency, or hardware change.
4. **Reduce scope safely.** Use a development or isolated environment to reproduce with the smallest relevant input. Do not replay customer traffic or destructive load against systems that are not authorized test targets.
5. **Choose the least disruptive recovery.** Follow the documented rollback or restart path only after confirming its scope. If state is uncertain or persistent data may be affected, stop and escalate to a maintainer.
6. **Verify recovery.** Confirm actual observed state, not only process exit status. Record the revision and evidence that the core function is restored.

### Readiness review checklist

- The intended user and supported use case are documented.
- The owning repository and adjacent contracts are clear.
- Configuration has validation, defaults, units, and secret handling.
- Success, partial completion, and failure have separate observable states.
- Retries are safe or explicitly identified as unsafe.
- Resource limits and exhaustion behavior are understood.
- Security-sensitive operations require appropriate identity and authorization.
- Logs and metrics support diagnosis without exposing sensitive data.
- Unit, contract, integration, and failure-path validation are present or scheduled.
- Upgrade, rollback, and data-retention behavior are stated.
- Known limitations are visible in release notes.
- No performance or security guarantee exceeds its supporting evidence.

This checklist should be reviewed for each release and updated when the component's architecture changes.