# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GitOps deployment repository for the **BioSimulations** platform. ArgoCD watches this repo and automatically syncs changes to a Kubernetes cluster. There is no application code here — only Kubernetes manifests, Kustomize overlays, and infrastructure configuration.

## Key Commands

```bash
# Bootstrap the entire cluster (run once)
kubectl apply -k cluster/

# Preview rendered manifests for an environment
kustomize build overlays/dev
kustomize build overlays/prod

# Update image tags for a deployment (what the CI workflow does)
cd overlays/dev && kustomize edit set image ghcr.io/biosimulations/api:v9.65.3

# Apply an overlay directly (prefer ArgoCD auto-sync instead)
kubectl apply -k overlays/dev

# Update the secrets submodule
git submodule update --remote secrets
```

## Architecture

**GitOps flow:** commit to this repo → ArgoCD detects change → syncs to cluster (auto-prune, auto-self-heal enabled).

**Kustomize layering:**
- `base/` — shared Deployment/Service definitions for all BioSimulations microservices
- `overlays/{dev,test,prod}/` — environment-specific patches (namespace, image tags, replica counts, ingress rules, NATS config)
- `config/{dev,test,prod}/` — ConfigMaps generated from `.env` files (shared config + per-service config)
- `secrets/` — private git submodule (`git@github.com:biosimulations/secrets`) with per-environment secrets

**Cluster infrastructure** (`cluster/`):
- `cluster/applications/` — ArgoCD Application CRDs that define what gets deployed and where
- `cluster/argocd/` — ArgoCD server config (GitHub SSO via Dex, RBAC, kustomized-helm plugin)
- `cluster/nginx-ingress/` — Ingress controller + Let's Encrypt issuers
- `cluster/prometheus/` — Prometheus + Grafana (with Loki/Tempo datasources)
- `cluster/argo-workflows/` — Workflow engine for simulation jobs
- `cluster/cert-manager/` — TLS certificate automation

**Microservices** (all images from `ghcr.io/biosimulations/*`):
`api`, `account-api`, `dispatch-service`, `simulators-api`, `mail-service`, `combine-api`, `simdata-api`

**Data stores:** MongoDB (ReplicaSet, 3 members), Redis HA, NATS

## Deployment Workflow

The GitHub Actions workflow (`.github/workflows/deploy.yml`) is triggered manually with `overlay` (dev/test/prod) and `tag` inputs. It runs `kustomize edit set image` for all microservice images and commits the change, which ArgoCD then picks up.

## Environment Differences

- **Dev:** namespace `dev`, latest tags (currently v9.65.x), lower replica counts, `account-api` scaled to 0
- **Prod:** namespace `prod`, stable tags (currently v9.62.x), higher replica counts for `mail-service`, `simulators-api`, `combine-api`
- Both use automated ArgoCD sync with prune and self-heal

## Important Conventions

- Image tags in overlay `kustomization.yaml` files are the primary mechanism for promoting versions between environments
- All services share a `shared.env` ConfigMap plus individual per-service ConfigMaps
- The `hack/` directory contains experimental configs (HPC integration, operator alternatives) — not actively deployed
- Secrets are never stored in this repo directly; they live in the private `secrets` submodule