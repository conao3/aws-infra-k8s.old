# aws-infra-k8s

AWS infrastructure for deploying Kubernetes clusters using CloudFormation and EC2 Image Builder.

## Overview

This project provides Infrastructure as Code (IaC) for setting up a production-ready Kubernetes environment on AWS. It uses CloudFormation templates generated from a custom ZAML (YAML-like) DSL and shell scripts for orchestrated deployment.

## Architecture

The infrastructure includes:

- **VPC**: Multi-AZ network with public and private subnets
- **Subnets**: Separate subnets for public access, application workloads, and databases
- **Security Groups**: Network-level access controls
- **S3**: Object storage for artifacts and state
- **EC2 Image Builder**: Custom AMIs pre-configured with Kubernetes components
- **EC2 Launch Templates**: Standardized instance configurations
- **SSH Tunnel and EICE**: Secure access to private resources via EC2 Instance Connect Endpoint

## Network Layout

| Subnet Type | AZ-A | AZ-C |
|-------------|------|------|
| Public | 10.0.0.0/24 | 10.0.30.0/24 |
| Private (App) | 10.0.10.0/24 | 10.0.40.0/24 |
| Private (DB) | 10.0.20.0/24 | 10.0.50.0/24 |

## Prerequisites

- AWS CLI configured with appropriate credentials
- The following tools installed:
  - `zaml` - YAML template preprocessor
  - `genja` - Jinja-based template generator
  - `yq` - YAML processor

## Getting Started

1. **Configure parameters**

   Copy the example parameter file and adjust for your environment:

   ```bash
   cp .param/.param.conao3.dev.k8s .param/.param
   ```

   Edit `.param/.param` with your AWS profile and environment settings.

2. **Generate deployment scripts**

   ```bash
   make gen
   ```

   This generates CloudFormation templates and shell scripts from the source `.zml` files.

3. **Deploy infrastructure**

   Deploy all stacks in order:

   ```bash
   ./deploy/sh/deploy_all
   ```

   Or deploy individual components:

   ```bash
   ./deploy/sh/000_network
   ./deploy/sh/001_routing
   ./deploy/sh/002_security_group
   ```

## Project Structure

```
.
├── .param/              # Environment parameters
├── app/
│   └── s3/scripts/      # Image builder scripts
├── deploy/
│   ├── .src/            # ZAML source templates
│   ├── cfn/             # Generated CloudFormation templates
│   └── sh/              # Generated deployment scripts
└── template/            # Jinja templates for code generation
```

## Kubernetes Setup

The EC2 Image Builder creates AMIs with the following pre-installed:

- Docker container runtime
- containerd with SystemdCgroup enabled
- kubeadm, kubelet, and kubectl (v1.29)
- Flannel CNI for pod networking

Master nodes are automatically initialized with `kubeadm init` and configured with Flannel networking.

## Deployment Order

The deployment scripts are numbered to indicate execution order:

1. `000_network` - VPC and subnets
2. `001_routing` - Internet gateway and route tables
3. `002_security_group` - Security group rules
4. `010_s3` - S3 buckets
5. `030_ec2_launch_template` - Instance launch templates
6. `031_ec2_image_builder` - AMI build pipelines
7. `990_ssh_tunnel` - SSH access configuration
8. `991_eice` - EC2 Instance Connect Endpoint

## License

See LICENSE file for details.
