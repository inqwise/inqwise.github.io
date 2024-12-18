# DevFinOps Project 2024

**Project Status:** Completed  
**Last Updated:** December 17, 2024
**Autor:** Alex Misyuk

## Introduction

As a technical enthusiast, I am always investigating and researching different aspects of the digital universe. Last year, I focused my research on **“Environment Costing”**, which explores how to create cloud application environments with minimal costs. This project, **DevFinOps**, embodies the principles of DevOps, FinOps, and operational excellence to achieve cost-effective, secure, and automated cloud infrastructures.

This project was processed by the **Inqwise Team**, leveraging our collective expertise to transform legacy environments into flexible, secure, and cost-aware AWS-based deployments. By employing automated processes, golden images, managed services, and well-defined CI/CD pipelines, we have achieved a more resilient and financially predictable infrastructure.

## Initial State of the Project

For our research, we selected a medium-active project with active clients and **12 instances** running on the AWS environment. This baseline state of the application is referred to as the **“origin”** environment.

One of our long-standing clients requested an update to their outdated infrastructure and the setup of an additional staging environment for their application. The project had several key objectives:

- **Automation and Modernization:**  
  Automate the processes of updating the application servers to simplify infrastructure management and increase overall stability.

- **Security and Resource Optimization:**  
  Ensure system security, including regular updates and resource reallocation, to minimize risks and improve reliability.

- **Cost-Effectiveness:**  
  Identify more affordable resource alternatives without degrading performance or reliability.

## Definition and Aims of the Project

The project involved setting up **two environments on AWS**: **Staging (STG)** and **Production (PROD)**. The STG environment utilizes native AWS services that are scheduled to be active during business hours and offline at night and on weekends to reduce costs. Repositories and CI/CD pipelines are managed on the GitHub platform.

### Project Goals

- **Deploy Two AWS Environments:**  
  Establish separate staging and production environments with optimized configurations for cost and performance.

- **Implement CI/CD Pipelines:**  
  Use GitHub for continuous integration and deployment to streamline updates and maintenance.

- **Optimize Resource Usage:**  
  Schedule non-essential services to reduce operational costs without affecting performance during peak times.

## Short Result of the Project

After **three months** of development and optimization, the **Inqwise Team** successfully completed the deployment and redirected all client traffic to the production environment. The updated infrastructure meets the objectives of automation, security, resource optimization, and cost-effectiveness.

## Services for Environment

### Native/AWS Managed Services

- **Native Services:**  
  Services installed and managed via CI/CD as spot/on-demand instances:
  - Application Front/API
  - MySQL
  - Redis
  - VPN
  - Elasticsearch
  - Kafka
  - Logstash
  - Loki/Grafana

- **AWS Managed Services:**  
  Services managed directly by AWS to ensure reliability and reduce operational overhead:
  - **S3:** Object storage for configuration, logs, and artifacts.
  - **Route53:** DNS management and routing.
  - **EC2:** Compute resources for running native services.
  - **VPC:** Network isolation and security boundaries.
  - **CloudWatch:** Monitoring and logging infrastructure.
  - **Certificate Manager:** SSL/TLS certificate provisioning and management.
  - **Key Management Service (KMS):** Encryption keys management.
  - **IAM:** Identity and access management.
  - **Lambda & SNS:** Event-driven functions and notifications.

## Installation Process of Native Services on Virtual Instances

Native services are installed using **Ansible playbooks**. To maintain scalability and reusability, playbooks are organized into logical parts:

1. **Shared Components:**  
   Common prerequisites, variables, credentials, and baseline tools (e.g., `mc`, `htop`) are stored in a shared location. Metric publishers like `telegraf` and notification clients like Discord bots are also included to ensure each instance can report status and metrics.

2. **Service-Specific "Stuff":**  
   The “stuff” section of each playbook handles unique configuration tasks required for a particular service (e.g., Redis, MySQL). This encapsulation keeps customization straightforward and localized.

### Configuration Scopes

Configurations are divided into three distinct scopes to streamline adjustments and overrides:

- **Global Scope:**  
  Hosted in an internal S3 bucket acting as an HTTP server for organization-wide settings.

- **Account Scope:**  
  Utilizes AWS Systems Manager (SSM) and an S3 bucket as an HTTP server for account-specific configurations.

- **Service Scope:**  
  Driven by autoscaling tags, allowing tailored adjustments to each service’s runtime environment.

### Metrics and Observability

To ensure transparency and facilitate data-driven decision-making, `telegraf` is installed on each instance to gather metrics such as memory, storage, CPU usage, and service health. These metrics are initially pushed to CloudWatch. However, due to rising costs, we are considering alternatives like **Grafana Mimir** to optimize performance without sacrificing insights.

## Golden Images (AMI) Advantages

**Golden images** (AMIs) provide a robust foundation for reliable and rapid instance provisioning:

1. **Pre-Installed Dependencies:**  
   3rd-party packages and dependencies are baked into the AMI, reducing setup time during instance launches.

2. **Fast Deployment:**  
   Instances can be quickly spun up from these pre-configured images, accelerating auto-scaling and recovery processes.

To support the golden-image workflow, we separated the Ansible playbook flow into two tags:

- **Installation Tag:**  
  Used for creating golden images via Packer.

- **Configuration Tag:**  
  Used to transform the pre-baked image into a fully functional service instance.

### Challenges and Solutions

- **3rd-Party Roles Execution:**  
  Overcame difficulties with inline variables and role rewrites to maintain playbook integrity and functionality.

## Image Creation with Packer

We selected **HashiCorp Packer** to build and manage AMIs due to its seamless integration with AWS services, support for multiple distributions (e.g., Amazon Linux 2 and AL2023), and ability to leverage AWS Secrets Manager for secure configurations.

**Highlights:**

- **Shared Configurations:**  
  Packer templates are centrally stored, reducing overhead and complexity.

- **Notifications:**  
  Packer builds send success/failure notifications to a Discord channel, including AMI IDs and service names.

- **Multi-Distribution Support:**  
  Supports both AL2 and AL2023 images within the same pipeline.

## AMI Distribution and Encryption Challenges

To handle multi-region, encrypted AMI distribution across independent AWS accounts (STG and PROD), we implemented the following strategy:

1. **Imaginarium Account:**  
   A dedicated account ("imagenarium") hosts the initial AMI build. Snapshots begin unencrypted.

2. **Sharing and Copying:**  
   After Packer completes, the unencrypted AMI is shared with target accounts (STG/PROD). Each environment then copies and encrypts the AMI within its own AWS account, ensuring compliance with security policies and KMS key usage constraints.

This approach avoids complex multi-region KMS management and resource-heavy re-encryption workflows.

## Cleanup and Maintenance

Automated clean-up processes were implemented to manage resource sprawl:

- **Scheduled Cleanup:**  
  A GitHub Action runs at intervals to shut down test instances, remove outdated volumes and DNS records, deregister old AMIs, and prune snapshots.

- **Version Retention:**  
  Only a predefined number of AMI versions per service are retained, ensuring a tidy and cost-effective environment.

## Final State and Cost Metrics

### Staging (STG) Environment

- **Servers:** 17
- **Total vCPU:** 38
- **Total Memory:** 43.5 GiB
- **Total Disk Storage:** 573 GiB
- **Monthly Cost:** ~$181

### Production Environment

- **Servers:** 27
- **Total vCPU:** 69
- **Total Memory:** 221 GiB
- **Total Disk Storage:** 1542 GiB
- **Monthly Cost:** ~\$1300

The final setup delivers flexible, lightweight, and adjustable environments that are cost-effective and secure.

## Tools and Libraries Used

- **HashiCorp Vagrant + vagrant-aws**
- **Ansible**
- **HashiCorp Packer**
- **VS Code**
- **AWS (S3, EC2, VPC, CloudWatch, etc.)**
- **Discord for Notifications**
- **GitHub for CI/CD**
- **Bash Scripting**

## Community and Contributions

We extend our gratitude to everyone who contributed to this project. The **Inqwise Team** worked together to bring DevFinOps to fruition, and we welcome new contributors to join our community and help improve DevFinOps.

- **Website:** [www.inqwise.com](https://www.inqwise.com)
- **Discord:** [https://discord.gg/hcqSZJFc](https://discord.gg/hcqSZJFc)
- **GitHub:** [https://github.com/orgs/inqwise](https://github.com/orgs/inqwise)

**Join us and implement these processes in your own environments to achieve a cost-aware, automated, and secure cloud infrastructure!**

---