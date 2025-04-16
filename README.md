# TT&C Infrastructure Design - Take-Home Assignment Response

## Introduction

This repository contains the proposed solution and supporting artifacts for the Satellite Telemetry, Tracking, and Control (TT&C) Infrastructure Design take-home assignment.


## How to Review

Please navigate the numbered directories corresponding to each assignment task to find the detailed responses, explanations, diagrams, and configuration examples. Each section's `README.md` provides context for the artifacts within it.


## Technology Stack

This solution proposes a cloud-native architecture leveraging **AWS** and incorporating DevOps best practices:

* **Core Infrastructure:** AWS VPC, EKS (Kubernetes), ALB, Aurora (PostgreSQL), Timestream, S3, SQS.
* **Management & Automation:** Terraform (IaC), Helm (K8s Packaging), Docker (Containerization), GitHub Actions (CI/CD - implied).

## Repository Structure

This repository is organized directly mapping to the assignment tasks:

* **`1-Architecture-Design/`**: Contains the detailed AWS architecture.
    * `README.md`: Architectural rationale and design decisions.
    * `a-Infrastructure-Diagram/`: Visual diagram (Mermaid code and PNG image).
    * `b-Infrastructure-Components/`: Breakdown of infrastructure components and their roles.

* **`2-Infrastructure-Deployment/`**: Illustrates the infrastructure deployment approach.
    * `README.md`: Overview of the Terraform IaC strategy.
    * `a-Terraform-Structure/`: Proposed Terraform project layout for managing AWS resources, emphasizing modularity and state management.

* **`3-Service-Deployment/`**: Outlines the application deployment strategy onto Kubernetes (EKS).
    * `README.md`: Overview of the Helm-based deployment methodology.
    * `a-Deployment-Methodology/`: Explanation of using Helm and Kubernetes features (HPA, PDB, Probes etc.) for scalable and HA deployments.
    * `b-Example-Configuration-Helm/`: A concrete example Helm chart for the Front-end service, demonstrating the implementation.
