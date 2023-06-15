# Deployment Repository

This repository contains code to deploy to Kubernetes.

Type of code and their respective paths are:

- `flux/*`:  deployment/dync manifests for Flux CD.
- `helm/*`: Helm Charts source code.
- `kustomize/*`: either plain YAMLs or additinally environment specific Kustomize overlays.

## Code Organisation Strategy

You would maintain 3 different repositories:

- Flux Fleet
- Project Deployment
- Project/App Source Code

### Repositories

#### Flux Fleet Repository

Manage organisation wide deployments to Kubernetes with FluxCD. This repository would contain:

- Cluster configurations for all your envvironments (e.g. staging, production, etc.). You would bootstrap the clusters by pointing to this repository.
- Cluster-wide infrastructure components (e.g. Ingress Controllers, repository Secrets, etc.).
- Project onboarding configurations AKA tenants spec. This is the spec that would point to a project repository for each tenant, fetch the sync manifests and deploy.

You could call it **flux-fleet** repository, which is managed typically by the SREs.

Who maintains Fleet Repository ==> Site Reliability Engineers / DevOps / Platform Engineers.

#### Project Deployment Repository (Tenant)

This repository would contain:

- Flux sync/deployment code. For example:
  - GitRepository
  - HelmRepository
  - Kustomization (FluxCD Deployments)
  - HelmRelease
- Helm Charts source code
- YAML manifests as either:
  - Plain YAML
  - With Kustomize overlays

This project repository serves as one tenant on the cluster.

Who maintains Project Deployment Repository ==> Developers + SREs.

#### Project/App Source Repository

- Application source code
  - Self-explanatory
  - Contains source code + Dockerfile + etc.

Who maintains Project Source Repository ==> Developers.

## Onboarding a Project

1. SREs setup Fleet Management of Kubernetes clusters by:
   - Creating *Flux Fleet Repository* with:
     - Cluster definitions, e.g.: qa, staging, production, etc.
     - Cluster-wide infrastructure deployment code
     - Projects (tenants) onboarding scaffold
   - Bootstrapping clusters with:
     - Git Source integration with *flux-fleet* repository
     - Flux Controllers
     - CRDs
     - Policies: RBAC, Network Policy, etc.
     - Git Source repository integrations
2. Developers/SREs create *Project Deployment Repository* with:
   - Flux sync/deployment manifests
   - YAMLs + Kustomize overlays
   - Helm Charts
3. Project owners raise an onboarding request:
   - Enable *Project Deployment Repository*
4. SREs onboard the project by:
   - Generating tenant RBAC
   - Generating project onboarding manifests
5. FluxCD reconciles the cluster state by:
   - Syncing project deployment code
   - Generating Flux resources for the project, e.g: GitRepository, Kustomization, HelmRelease, etc.
   - Running reconciliation for each of the Flux resources in the Kubernetes cluster.
