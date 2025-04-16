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

This repository is organized to directly map to the assignment tasks:

* **`1-Architecture-Design/`**: The AWS architecture design and rationale
  * `a-Infrastructure-Diagram/`: Visual representation of the architecture
  * `b-Infrastructure-Components/`: Detailed breakdown of each infrastructure component

* **`2-Infrastructure-Deployment/`**: Infrastructure-as-Code implementation approach
  * `a-Terraform-Structure/`: Modular Terraform project structure with separate components for network, security, EKS, data stores, and more
  * Contains reusable modules and deployment order to build the complete infrastructure

* **`3-Service-Deployment/`**: Kubernetes application deployment strategy
  * `a-Deployment-Methodology/`: Helm-based approach for deploying microservices
    * Contains Helm charts for the command-service, telemetry-service, and frontend-service
  * `b-Example-Configuration-Helm/`: Detailed example implementation for the frontend service
    * Demonstrates Kubernetes best practices (HPA, PDB, ingress configuration, etc.)

Each directory contains README files with explanations of design decisions, implementation details, and usage instructions. The structure follows a logical progression from architecture design to infrastructure deployment to application deployment, providing a complete solution for the Satellite Telemetry, Tracking, and Control (TT&C) system.
