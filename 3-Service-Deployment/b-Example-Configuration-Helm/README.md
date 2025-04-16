# Task 3b: Example Helm Chart - Frontend Service

This directory contains an example Helm chart specifically for deploying the **Frontend Service**. It serves as a concrete illustration of the deployment methodology outlined in Task 3a, showcasing how Helm is used to configure Kubernetes resources for scalability and high availability.

The files provided (`Chart.yaml`, `values.yaml`, and templates within `templates/`) represent a standard Helm chart structure.

## Key Chart Files Overview

* **`Chart.yaml`**: Standard Helm chart metadata (name, version, etc.).
* **`values.yaml`**: **(Primary Configuration Interface)** This crucial file defines the *default configurable parameters* for the Frontend service deployment. It allows customization of:
    * Deployment specifics (image repository/tag).
    * Scalability settings (initial replica count, HPA min/max replicas, CPU/memory targets, container resource requests/limits).
    * High Availability settings (PDB configuration, probe endpoints and timings, pod anti-affinity preferences).
    * Networking (Service port, Ingress rules, hostnames, ALB annotations including placeholder for ACM cert ARN).
    * Security (ServiceAccount annotations placeholder for IRSA role ARN).
* **`templates/`**: This directory holds the Kubernetes manifest templates rendered by Helm using values from `values.yaml`. Key templates include:
    * `deployment.yaml`: Templates the `Deployment`, incorporating HA features like probes, resource definitions, anti-affinity, and rolling updates.
    * `hpa.yaml`: Templates the `HorizontalPodAutoscaler` based on `values.yaml` targets.
    * `pdb.yaml`: Templates the `PodDisruptionBudget` using availability settings from `values.yaml`.
    * `service.yaml`: Templates the internal `ClusterIP` Service.
    * `ingress.yaml`: Templates the `Ingress` resource, configuring external access via ALB based on `values.yaml`.
    * `serviceaccount.yaml`: Templates the `ServiceAccount` configured for IRSA via annotations set in `values.yaml`.

## How this Example Addresses HA & Scalability

This chart configuration directly implements the strategy from Task 3a:

* **High Availability:** Achieved via templated `Deployment` (replicas, probes, anti-affinity, rolling updates) and `PDB` resources, configured through `values.yaml`.
* **Scalability:** Addressed via the templated `HPA` resource automatically adjusting replicas based on load, with targets and container resources defined in `values.yaml`.

## Customization for Environments

The power of this approach lies in using the `values.yaml` file (or overriding its values via `--set` flags or separate `values-*.yaml` files during `helm install/upgrade`) to tailor the deployment for specific environments (dev, staging, prod) without altering the underlying Kubernetes resource templates.

## Conclusion

This example provides a tangible configuration demonstrating how the Frontend service can be deployed using Helm. It packages Kubernetes best practices for HA and scalability into a reusable and configurable format. Similar charts, tailored with appropriate `values.yaml` settings and templates (e.g., without `Ingress` for the Command service), would be created for the other TT&C services.