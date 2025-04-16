# Task 2a: Infrastructure Deployment using Terraform

This document illustrates the proposed method for deploying the TT&C system's AWS infrastructure using **Terraform** as the Infrastructure as Code (IaC) tool. The focus is on demonstrating the deployment strategy through a well-structured and maintainable project layout, rather than providing the complete HCL code.

## Approach: Modular Terraform Structure with Remote State

To manage the infrastructure effectively, a modular approach is proposed. The Terraform code is organized into logical components (e.g., networking, security, EKS cluster, data stores), each residing in its own directory. Key characteristics of this approach include:

* **Separate State Files:** Each component manages its own Terraform state file remotely in an Amazon S3 bucket (configured via `backend.tf`). This reduces the blast radius of changes and prevents state file conflicts.
* **Modularity & Reusability:** Common infrastructure patterns (like creating a VPC or an EKS Node Group) are encapsulated in reusable modules stored in the central `modules/` directory. Root modules (in numbered directories) call these shared modules.
* **Explicit Dependencies:** Components declare dependencies on outputs from earlier stages using `terraform_remote_state` data sources (e.g., the EKS cluster component retrieves VPC details from the network component's state). This ensures resources are created in the correct order.
* **Clear Deployment Order:** The numbered directories suggest a logical sequence for provisioning the infrastructure layers.

## Proposed Terraform Project Structure

The following structure illustrates the layout of the Terraform project designed to deploy the architecture described in Task 1:

```plaintext
terraform-ttc-infra/
├── README.md                      # Project overview, deployment order, prerequisites
├── common.tfvars                  # (Optional) Common variables used across components (region, project tags, etc.)
├── modules/                       # Reusable, generic Terraform modules
│   ├── vpc/                       # Module for creating VPC, subnets, route tables, GWs
│   ├── security_group/            # Module for creating security groups
│   ├── iam_role/                  # Module for creating IAM roles/policies (e.g., for IRSA)
│   ├── kms_key/                   # Module for creating KMS keys
│   ├── ecr_repository/            # Module for creating ECR repositories
│   ├── eks_cluster/               # Module for provisioning EKS control plane
│   ├── eks_node_group/            # Module for provisioning EKS managed node groups
│   ├── eks_fargate_profile/       # Module for provisioning EKS Fargate profiles (if used)
│   ├── eks_addons/                # Module for deploying common K8s addons (e.g., AWS LB Controller via Helm/manifest)
│   ├── rds_aurora_postgres/       # Module for provisioning Aurora PostgreSQL cluster
│   ├── s3_bucket/                 # Module for creating S3 buckets with standard configs
│   ├── sqs_queue/                 # Module for creating SQS queues
│   ├── timestream_database_table/ # Module for creating Timestream DB and Table
│   └── secrets_manager_secret/    # Module for creating Secrets Manager secrets
│
├── 01-network/                    # Deploys foundational networking
│   ├── main.tf                    # Defines resources using the vpc module, maybe SG module
│   ├── variables.tf               # Input variables (e.g., CIDR blocks, AZs)
│   ├── outputs.tf                 # Outputs (VPC ID, subnet IDs, security group IDs) needed by other components
│   └── backend.tf                 # Configures S3 backend (e.g., key = "network/terraform.tfstate")
│
├── 02-security-foundational/      # Deploys core security resources (KMS, core IAM roles)
│   ├── main.tf                    # Uses kms_key module, iam_role module (e.g., base EKS cluster role)
│   ├── variables.tf               # Input variables
│   ├── outputs.tf                 # Outputs (KMS key ARN, Role ARNs)
│   └── backend.tf                 # Configures S3 backend (e.g., key = "security-foundational/terraform.tfstate")
│
├── 03-ecr/                        # Deploys ECR repositories for services
│   ├── main.tf                    # Uses ecr_repository module, possibly loops over service names
│   ├── variables.tf               # Input variables (service names)
│   ├── outputs.tf                 # Outputs (repository URLs)
│   └── backend.tf                 # Configures S3 backend (e.g., key = "ecr/terraform.tfstate")
│
├── 04-eks-cluster/                # Deploys the EKS control plane
│   ├── main.tf                    # Uses eks_cluster module
│   ├── data.tf                    # Reads remote state from 01-network (VPC, subnets), 02-security (roles)
│   ├── variables.tf               # Input variables (cluster version, name)
│   ├── outputs.tf                 # Outputs (cluster endpoint, OIDC provider URL, cluster SG ID)
│   └── backend.tf                 # Configures S3 backend (e.g., key = "eks-cluster/terraform.tfstate")
│
├── 05-eks-nodes/                  # Deploys EKS worker nodes (Managed Node Groups or Fargate)
│   ├── main.tf                    # Uses eks_node_group / eks_fargate_profile modules
│   ├── data.tf                    # Reads remote state from 01-network (subnets), 04-eks-cluster (cluster info), 02-security (node role)
│   ├── variables.tf               # Input variables (instance types, desired size)
│   ├── outputs.tf                 # Outputs (node group IDs, Fargate profile ARN)
│   └── backend.tf                 # Configures S3 backend (e.g., key = "eks-nodes/terraform.tfstate")
│
├── 06-eks-addons/                 # Deploys essential K8s addons (e.g., AWS Load Balancer Controller, metrics-server, maybe observability agents)
│   ├── main.tf                    # Uses Helm provider or Kubernetes provider, potentially calling eks_addons module
│   ├── data.tf                    # Reads remote state from 04-eks-cluster (cluster info for K8s/Helm provider)
│   ├── variables.tf               # Input variables (addon versions, settings)
│   ├── outputs.tf                 # Outputs (e.g., LB Controller IAM role ARN if created here)
│   └── backend.tf                 # Configures S3 backend (e.g., key = "eks-addons/terraform.tfstate")
│
├── 07-data-stores/                # Deploys databases, queues, storage buckets
│   ├── main.tf                    # Uses rds_aurora_postgres, s3_bucket, sqs_queue, timestream_database_table modules
│   ├── data.tf                    # Reads remote state from 01-network (subnets, SGs), 02-security (KMS key)
│   ├── variables.tf               # Input variables (DB names, bucket names, etc.)
│   ├── outputs.tf                 # Outputs (DB endpoint, bucket names, queue URL, Timestream DB/Table names)
│   └── backend.tf                 # Configures S3 backend (e.g., key = "data-stores/terraform.tfstate")
│
└── 08-app-support/                # Deploys resources specifically supporting the applications
    ├── main.tf                    # Uses iam_role (for IRSA), secrets_manager_secret, security_group modules
    ├── data.tf                    # Reads remote state from 01-network (VPC), 04-eks-cluster (OIDC), 07-data-stores (DB endpoint for secret)
    ├── variables.tf               # Input variables (application names, secrets to store)
    ├── outputs.tf                 # Outputs (App IAM role ARNs, Secret ARNs, App-specific SG IDs)
    └── backend.tf                 # Configures S3 backend (e.g., key = "app-support/terraform.tfstate")
```

*(Note: `...` indicates standard files like `main.tf`, `variables.tf`, `outputs.tf` within each numbered directory.)*

## Deployment Flow

Infrastructure deployment follows the numerical order of the directories:

1.  **Prerequisites:** Ensure AWS credentials, Terraform CLI, and the S3 backend bucket (+/- DynamoDB lock table) are configured.
2.  **Sequential Apply:**
    * `cd 01-network/ && terraform init && terraform apply -auto-approve`
    * `cd ../02-security-foundational/ && terraform init && terraform apply -auto-approve`
    * ... continue for `03` through `08`.
3.  **Dependency Management:** The use of `terraform_remote_state` ensures that Terraform fetches necessary outputs (like VPC ID) from previously applied components before attempting to create dependent resources.

This step-by-step process guarantees that foundational layers are built before the components that rely on them.

## CI/CD Integration

This modular structure is ideal for automation via CI/CD pipelines (e.g., GitHub Actions). The pipeline would execute the `terraform apply` steps for each component sequentially, potentially incorporating stages for planning (`terraform plan`), approvals, and testing.

## Conclusion

This illustrated Terraform project structure provides a robust, maintainable, and scalable method for deploying the TT&C system's AWS infrastructure. It adheres to IaC best practices by promoting modularity, managing state effectively, and clearly defining dependencies between infrastructure components.
