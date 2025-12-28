# Lab 03: Multi-Tier Application

## Objective

Create a Helm chart for a complete multi-tier application (Frontend, Backend, Database).

## Prerequisites

- Completed Labs 01 and 02
- Understanding of Kubernetes Deployments and Services
- Basic understanding of StatefulSets

## Coming Soon

This lab will guide you through creating a Helm chart for a complete multi-tier application including:

- Frontend service (React)
- Backend API (Node.js)
- Database (PostgreSQL with StatefulSet)
- Redis cache
- Ingress configuration
- Inter-service communication

## Expected Components

```
mychart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── frontend/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── configmap.yaml
    ├── backend/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── configmap.yaml
    ├── database/
    │   ├── statefulset.yaml
    │   ├── service.yaml
    │   └── pvc.yaml
    └── ingress.yaml
```

---

**Status:** Content coming soon  
**Related:** [Lab 02: Nginx Deployment](../02-nginx-deployment/)

