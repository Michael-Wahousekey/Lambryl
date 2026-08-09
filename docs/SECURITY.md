# Security & Governance

## Overview

This document describes security and governance of the project. It is meant to be followed by the developers of the project. 

The platform follows least privilege, defense in depth, zero-trust principles, and policy-as-code, with security controls managed declaratively through GitOps wherever practical.

## Security Controls

| Domain | Control | Purposes |
|---|---|---|
| Identity | IAM, OIDC |	Federated identity and cloud authorization
| Kubernetes Access | RBAC |	Namespace and resource-level authorization
| Network | Cilium |	Network segmentation and workload connectivity policies
| Service Communication | Istio	| Service identity, mTLS, and service authorization
| Admission | Kyverno |	Kubernetes security and governance policies
| Secrets | External Secrets Operator |	External secret synchronization without storing credentials in Git
| Certificates | cert-manager |	Automated certificate issuance and lifecycle management
| Deployment | Argo CD |	Declarative, auditable GitOps delivery


## Identity & Access

Access follows a least-privilege model.

1. Human authentication is managed through centralized identity providers using OIDC, while authorization is enforced through IAM, RBAC, or service-specific role mappings. Workloads use dedicated service identities and should not rely on long-lived credentials.

2. Cluster-wide administrative permissions are restricted to platform-level operations. Application workloads are isolated to the minimum namespace and resource scope required for their operation.

3. Cloud credentials are provided through workload identity mechanisms rather than embedded credentials in workloads, images, or repositories.

4. OIDC provides authentication and identity federation, while authorization remains controlled by the target platform through IAM, RBAC, or service-specific roles.

## Network Security

The platform follows a default-deny where practical approach, with connectivity explicitly permitted between required workloads and services.

Network policies are used to control:
1. Workload-to-workload communication
2. Namespace boundaries
3. Ingress and egress
4. Access to infrastructure services
5. External connectivity

Network access is treated as an explicit dependency rather than an implicit capability of workloads within the cluster.

## Service-to-Service Security

Mutual TLS is used where applicable to provide encrypted and authenticated service-to-service communication.

Service authorization policies provide an additional layer above network connectivity, allowing access decisions to be based on workload identity rather than network location alone.

## Workload Security & Policy

Security and governance policies can enforce requirements such as:
- Approved container registries
- Secure workload configuration
- Non-root execution
- Restricted capabilities
- Resource requirements
- Required metadata and ownership
- Prohibited privileged workloads
- Restricted host access
- Approved deployment patterns

Policies can operate in audit or enforcement mode depending on the maturity and compatibility of the control.

The goal is to prevent insecure configurations from reaching production rather than relying exclusively on runtime detection.

## Secrets Management

Secrets are not stored directly in application manifests or Git repositories.

External Secrets Operator integrates Kubernetes workloads with an external secrets provider, allowing credentials to remain outside the deployment repository while still being consumed by workloads through standard Kubernetes Secret interfaces.

Secret access follows the same least-privilege model as other platform resource.

## Certificate Management

Certificate issuance and renewal are automated to reduce operational dependency on manually managed credentials.

Private keys remain managed by the certificate infrastructure and are not committed to source control.

## Governance

Security configuration is managed as code and delivered through the platform's GitOps workflow.

Changes to security-sensitive configuration are therefore:
- Version controlled
- Peer reviewed
- Auditable
- Reproducible
- Reversible

Security policies are treated as platform contracts rather than optional application configuration.

Applications are expected to conform to the platform's security baseline without requiring each team to independently implement foundational security controls.

## Exceptions

Security controls may require exceptions for legitimate technical or operational requirements.

Exceptions should be:
1. Explicitly documented
2. Scoped to the minimum required resources
3. Time-bound where possible
4. Reviewed by the appropriate platform/security owner

Exceptions should not silently weaken the platform's baseline.