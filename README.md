# 🚀 Mini-Vercel on EKS — Architecture & Implementation Guide
## Overview
This platform allows your dev team to deploy POCs from a Git repo to an existing EKS cluster with minimal manual steps.
It’s similar to Vercel, but self-hosted and tailored to your infra.
----
## High-Level Architecture
```mermaid
flowchart LR
  subgraph User
    UI["Web UI / CLI / API"]
    GitWebhook["Git Webhook"]
  end

  subgraph Orchestrator
    API["Orchestrator API (FastAPI or Go)"]
    Auth["Auth & RBAC"]
    Queue["Job Queue (Redis or RabbitMQ)"]
    Secrets["Secret Manager (K8s Secrets or Vault)"]
    Logger["Build Logs -> S3 or ELK"]
  end

  subgraph KubernetesCluster["EKS Cluster"]
    BuilderNS["builder-namespace"]
    BuildJob["Build Job (Kaniko or Pod)"]
    Registry["Container Registry (ECR or Harbor)"]
    AppNS["app-namespace"]
    AppDeployment["Deployment + Service"]
    Ingress["Ingress Controller (ALB / NGINX / Traefik)"]
    CertMgr["cert-manager"]
    ExternalDNS["external-dns"]
  end

  UI --> API
  GitWebhook --> API
  API -->|store creds| Secrets
  API --> Queue
  Queue --> BuildJob
  BuildJob -->|git clone & build| GitRepo["Git Repo"]
  BuildJob -->|push image| Registry
  BuildJob --> Logger
  BuildJob --> API
  API -->|apply manifest| AppDeployment
  AppDeployment --> Ingress
  Ingress -->|route| AppDeployment
  Ingress --> ExternalDNS
  CertMgr --> Ingress
  Auth --> API

```
