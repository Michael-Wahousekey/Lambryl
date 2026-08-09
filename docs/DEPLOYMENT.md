# Deployment

## Overview

This document defines the deployment workflow and change-management standards for infrastructure, platform components, and applications.

The deployment process follows a progressive promotion model
`dev -> staging -> prod`

Changes are validated progressively before reaching production. Direct production modification is intentionally restricted through IAM, RBAC, CI/CD controls, and repository governance.

Each `env/region` are isolated from each other. To prevent developers from modifying protected production infrastructure.

## Infrastructure

- Infrastructure is primarily managed through Terraform.

    ## Workflow

    1. Clone the Infrastructure repository.
    2. Work from the `dev` branch.
    3. Infrastructure changes should be limited to the appropriate `dev/region/*` scope.
    4. Validate changes locally using Terraform:
        - `terraform init`
        - `terraform plan -out=tfplan`
        - `terraform apply tfplan` (Appropriately)
    5. Provided scripts may be used to simulate CI runner behavior for `dev` stage.
    6. Once changes have been tested and validated against platform standards, promote the change to the `staging` branch.
    7. The staging runner validates the change using production-like CI behavior and state.
    8. After successful staging validation, open a pull request for production promotion.
    9. Production changes require approval from repository maintainers before merging.

    ## Standards

    reusable modules should be defined in modules/*

    files should follow standard terraform practices. main.tf, variables.tf, outputs.tf, etc.


## Platform

- Platform components are primarily deployed through Helm and Kubernetes, with Argo CD providing GitOps-based deployment and reconciliation.

    ## Workflow
    1. Clone the Platform repository.
    2. Work from the `dev` branch.
    3. Changes should be limited to the appropriate `dev/region/*` scope.
    4. Validate Helm changes locally using `helm template`.
    5. Where practical, install the chart into a local development cluster to validate the actual Kubernetes behavior.
        - Push changes to `dev` branch is also allowed, as this will not disrupt other clusters.
    6. Once changes have been tested and validated against platform standards, push changes to `staging` branch.
    7. Argo CD within staging cluster reconciles the changed states.
    8. Monitor behaviors and anomalies.
    9. After successful staging validation, open a pull request for production promotion.

    Local validation is preferred before promotion so that invalid charts and Kubernetes resources are identified before entering registries.

    ## OCI Charts

    OCI charts are versioned deployment artifacts and therefore follow immutable versioning rules.

    Chart versions must not be reused. Once an OCI chart version has been published, subsequent changes require a new version.

    When modifying an OCI chart:
    1. Increment the chart version following the standard [practice](#standards-1).
    2. Validate the chart locally.
    3. Where practical, install the chart into a local development cluster.
    4. Ensure the CI workflow can package and publish the new chart version.
    5. Update the corresponding `releases/*.yaml` configuration to reference the intended chart version.
    6. Promote the release through the standard [workflow](#workflow-1) practice.

    ## Local Charts

    Local charts are used primarily as templates for resources and catalogs that are maintained directly within the platform repository.

    These charts do not require OCI publication when they are not distributed as versioned deployment artifacts.

    The corresponding `releases/*.yaml` configuration remains the source of deployment intent and must be updated whenever the resource or catalog is intended to be deployed.

    ## Standards 

    Platform configuration should follow consistent naming conventions.

    The additional qualifier should only be used when required to distinguish otherwise duplicated resources.

    Files should use the following general pattern:
    - `{application}.{optional-qualifier}.{resource}.yaml`

    Example: 
    - lambryl.yaml
    - lambryl.frontend.yaml

    File should at least contain
    ```yaml
    releaseName: _
    chart: _
    namespace: _
    targetRevision: 1.0.0
    ```

## Application

This workflow applies to user-facing and application workloads, including:
- Backend services
- Frontend services
- Workers
- Kubernetes operators

    ## Workflow

    1. Clone the Platform repository.
    2. Work from the `dev` branch.
    3. Applications should always be tested locally before being promoted.
        - The repository provides runnable `./scripts/*` to simulate CI runner behavior, including image and chart packaging.
        - Local validation should be performed before using these scripts to reduce unnecessary artifact publication and failed deployments.
    4. Update metadata
        1. Docker image
            - Application Metadata must be updated when application behavior or release content changes.
            - Exp. Node.js applications should update the version defined in package.json according to the project's release conventions.
            - When the appropriate versioning strategy is unclear, consult the application owner or supervisor before publishing a new release.
        2. Chart
            - Update the chart version.
            - Validate the chart locally.
            - Publish the new chart through CI or `./scripts/*` (local only), following [standards](#oci-charts).
    6. Once changes have been tested and validated against application standards, push changes to `staging` branch.
    7. After successful staging validation, open a pull request for production promotion.
    8. Provide a `releases/*.yaml` for deployment to the platform team, ensure to follow [standards](#standards-1).
    9. Applications would therefore be deployed through the established CI/CD and GitOps workflow rather than through manual production changes.


    ## Standards 

    Application repositories should follow the standard project structure:
    ```
    ./apps/*
    ./charts/*
    ./scripts/*
    ```

    Existing application repositories should be used as reference when creating new services.


## Rollback

Application and platform rollback is handled through Argo CD and GitOps reconciliation.

The desired state stored in Git remains the source of truth. Rollbacks should therefore normally be performed by reverting or updating the declarative configuration to a previously known-good version rather than manually modifying resources in the cluster.

This preserves:
- **Auditability**
- **Reproducibility**
- **Version history**
- **Consistent cluster state**

Emergency operational intervention may be performed when required to protect availability or security, but the resulting state must subsequently be reconciled with the declarative source of truth.

## Bootstrap

### **Application**

Application repositories may provide their own bootstrap process for establishing the initial application configuration required for startup.

Bootstrap configuration establishes the initial state of an application. However, it should not be treated as the authoritative source of current application data.

Changes to bootstrap behavior should be made through the application's source repository and follow the normal review and deployment workflow.

### **Infrastructure**

Infrasturcture have `/bootstrap/*`, please read instructions within the repository.

### **Platform**

Platform bootstrap is handled as part of cluster provisioning and infrastructure deployment rather than by individual platform components.

The infrastructure team owns cluster creation and platform bootstrap.

Platform engineers should consult the Infrastructure repository and its bootstrap documentation when modifying or troubleshooting cluster initialization.