# Learning Helm

A comprehensive guide to mastering Helm, the Kubernetes package manager.

## Overview

Helm is the package manager for Kubernetes, allowing you to define, install, and upgrade complex Kubernetes applications. This repository provides hands-on learning materials to master Helm for application packaging and deployment.

## What is Helm?

Helm helps you:
- **Package** Kubernetes applications into reusable charts
- **Manage** application deployments with releases
- **Version** your Kubernetes configurations
- **Share** application configurations across teams
- **Simplify** complex deployments with templates and values

## Learning Path

### Prerequisites
- Working Kubernetes cluster (AKS, EKS, GKE, or local)
- kubectl configured and working
- Basic Kubernetes knowledge (Deployments, Services, ConfigMaps)
- Command line familiarity

### Learning Structure

```
learning-helm/
├── README.md (this file)
├── concepts/              # Theoretical knowledge
│   ├── 01-helm-fundamentals.md
│   ├── 02-charts-and-templates.md
│   ├── 03-values-and-configuration.md
│   └── 04-advanced-helm.md
├── labs/                  # Hands-on exercises
│   ├── 01-first-chart/
│   ├── 02-nginx-deployment/
│   ├── 03-multi-tier-app/
│   └── 04-chart-dependencies/
├── examples/              # Reference implementations
│   ├── simple-webapp/
│   ├── microservice/
│   └── stateful-app/
└── REFERENCES.md          # External resources
```

## Quick Start

1. **Install Helm:**
   ```bash
   # macOS
   brew install helm
   
   # Linux
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   
   # Windows
   choco install kubernetes-helm
   ```

2. **Verify Installation:**
   ```bash
   helm version
   ```

3. **Create Your First Chart:**
   ```bash
   helm create myapp
   cd myapp
   helm install myapp-release .
   ```

## Learning Objectives

By the end of this learning path, you will be able to:

- ✅ Understand Helm architecture (Charts, Releases, Repositories)
- ✅ Create and package Helm charts
- ✅ Use values files for environment-specific configurations
- ✅ Work with Helm templates and functions
- ✅ Manage chart dependencies
- ✅ Deploy and upgrade applications with Helm
- ✅ Implement Helm best practices

## Recommended Learning Order

1. **Start Here:** [Concepts: Helm Fundamentals](./concepts/01-helm-fundamentals.md)
2. **First Lab:** [Lab 01: Create Your First Chart](./labs/01-first-chart/)
3. **Continue Learning:** Work through concepts and labs sequentially
4. **Practice:** Use examples as reference for your own projects

## Related Learning Repositories

- [learning-kubernetes](https://github.com/vanHeemstraSystems/learning-kubernetes) - Kubernetes fundamentals
- [learning-crossplane](https://github.com/vanHeemstraSystems/learning-crossplane) - Infrastructure as Code
- [learning-backstage](https://github.com/vanHeemstraSystems/learning-backstage) - Developer portal (uses Helm)

## Contributing

Found an issue or want to add content? Please open an issue or submit a pull request.

---

**Last Updated:** December 26, 2025  
**Maintainer:** Willem van Heemstra  
**License:** MIT
