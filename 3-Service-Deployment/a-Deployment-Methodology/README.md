# Task 3a: Service Deployment Methodology

This document outlines the strategy for deploying the Command, Telemetry, and Front-end services onto the EKS cluster, focusing on ensuring scalability and high availability.

## Recommendation: Helm Charts

The recommended approach is to package and manage each service using **Helm charts**. A dedicated chart will encapsulate all Kubernetes resources for each service (Command, Telemetry, Front-end), organized within the `charts/` directory structure visible alongside this document. This leverages Helm for standardized packaging, templating, and release management.
```
3a-service-deployment-helm/
├── README.md                      # Overall deployment strategy using Helm charts
│
└── charts/                        # Directory containing individual service charts
    ├── command-service/           # Helm chart for the Command Service (Structure below)
    │   ├── Chart.yaml
    │   ├── values.yaml
    │   ├── templates/
    │   │   ├── deployment.yaml
    │   │   ├── service.yaml
    │   │   ├── hpa.yaml
    │   │   ├── pdb.yaml
    │   │   ├── serviceaccount.yaml  # For IRSA configuration
    │   │   └── _helpers.tpl         # Optional template helpers
    │   └── README.md                # Chart-specific details (optional)
    │
    ├── telemetry-service/         # Helm chart for the Telemetry Service (Similar structure)
    │   ├── Chart.yaml
    │   ├── values.yaml
    │   ├── templates/
    │   │   ├── deployment.yaml
    │   │   ├── service.yaml
    │   │   ├── ingress.yaml         # If using push/webhook model via ALB
    │   │   ├── hpa.yaml
    │   │   ├── pdb.yaml
    │   │   ├── serviceaccount.yaml
    │   │   └── _helpers.tpl
    │   └── README.md
    │
    └── frontend-service/          # Helm chart for the Front-end Service (Example structure detailed below)
        ├── Chart.yaml             # Metadata: name, version, appVersion etc.
        ├── values.yaml            # Default configuration values (overridable)
        ├── templates/             # Directory with templated K8s manifests
        │   ├── NOTES.txt          # Optional: Notes displayed after install/upgrade
        │   ├── _helpers.tpl       # Optional: Template helper functions/partials
        │   ├── deployment.yaml    # Template for the Deployment object
        │   ├── service.yaml       # Template for the ClusterIP Service object
        │   ├── ingress.yaml       # Template for the Ingress object
        │   ├── hpa.yaml           # Template for the HorizontalPodAutoscaler
        │   ├── pdb.yaml           # Template for the PodDisruptionBudget
        │   └── serviceaccount.yaml# Template for the K8s ServiceAccount (needed for IRSA)
        └── README.md              # Chart-specific details (optional)
```
*(Refer to the example chart structure and templates for the Front-end service in `../b-Example-Configuration-Helm/` for a concrete implementation example.)*

## Core Strategy for High Availability & Scalability

Each Helm chart achieves HA and scalability by templating the following core Kubernetes resources, configured via `values.yaml`:

* **`Deployment`:** Manages application lifecycle with multiple replicas, zero-downtime rolling updates, readiness/liveness probes for health checking, resource requests/limits for scheduling, and pod anti-affinity rules for AZ/node spread (HA).
* **`HorizontalPodAutoscaler` (HPA):** Enables automatic scaling of pod replicas based on configurable CPU/memory utilization targets (Scalability).
* **`PodDisruptionBudget` (PDB):** Protects against voluntary disruptions during maintenance by ensuring minimum pod availability (HA).
* **`Service` / `Ingress`:** Provides stable endpoints for service discovery (`ClusterIP`) and external access via ALB (`Ingress`) where required (HA & Access).
* **`ServiceAccount`:** Configured with annotations for IRSA, providing secure, role-based access to AWS services (Security & Functionality).

## Configuration & Customization

The use of Helm's `values.yaml` file within each chart is central to this strategy. It allows critical parameters (image tags, replica counts, resource limits, HPA settings, probe configurations, IRSA roles, Ingress details) to be easily configured and overridden per environment (dev, staging, prod), ensuring flexibility and consistency.

## Deployment Integration

Deployments are managed using standard Helm commands (`helm upgrade --install ...`). This process is designed for integration into CI/CD pipelines (e.g., GitHub Actions) for automated, reliable application releases onto the EKS cluster.

## Conclusion

This Helm-based methodology provides a robust and standardized way to deploy the TT&C services. It directly addresses high availability and scalability requirements through configuration-driven Kubernetes patterns, managed effectively via Helm charts.