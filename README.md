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