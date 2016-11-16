---
layout: iac-manager
title:  Creating a Managment Environment in AWS with terraform.
date: 2016-07-19 11:20:30
tags: manager
banner: /images/iac-manager.jpg
categories: Projects
Projects: Iac-manager
git: https://github.com/microservices-today/IaC-manager
---
### IaC: Manager Node (Jump Server)
This terraform script will setup an infra for management in AWS:
 - CentOS (Manager Node)
 - VPC
 - Public subnet
 - Internet gateway

#### Pre-requisites
- An IAM account with administrator privileges.
- Install [terraform](https://www.terraform.io/intro/getting-started/install.html) on your machine.
- Public Key Access with Agent support/ Agent Forwarding:

  ```bash
  ssh-add <key_pair_name>.pem
  ssh -A centos@<manager_public_ip>
  ```

#### Steps for installation
- Clone this repo .
- `cp terraform.dummy terraform.tfvars`
- Modify params in `terraform.tfvars`
- Modify params in `variable.tf` to change subnet or add AMI accordingly to your aws region
- Export AWS credentials as bash variables (e.g. `ap-northeast-1` for Tokyo and `ap-southeast-1` for Singapore region):
```bash
export AWS_ACCESS_KEY_ID="anaccesskey"
export AWS_SECRET_ACCESS_KEY="asecretkey"
export AWS_DEFAULT_REGION="ap-northeast-1"
```
- Run `terraform plan` to see the plan to execute.
- Run `terraform apply` to run the scripts.
- You may have `prod/dev/stage` configurations in
`terraform.tfvars.{prod/dev/stage}` files (already ignored by `.gitignore`).

#### Generate Docs
Generate terraform documentation by running `bash generate-docs.sh`

#### Notes
- SSH into the manager node and check whether `terraform.out` in `home/centos` contains:
a record of the VPC, Subnet, Security Group and Nat gateway ID.
- More details on [terraform-docs](https://github.com/segmentio/terraform-docs).
