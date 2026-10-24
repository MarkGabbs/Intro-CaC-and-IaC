# Terraform Module: Private Linux VMs (No Internet)

Deploys a VPC with a single private subnet and 3 (configurable) Amazon Linux 2023 instances of the smallest size (`t3.nano` by default).

**Key characteristics:**
- No Internet Gateway
- No NAT Gateway
- No public IP addresses
- Route table contains only the local VPC route
- Security group explicitly blocks all inbound and outbound traffic to the internet

## Usage

```hcl
module "private_vms" {
  source = "./terraform-private-linux-vms"

  region               = "us-east-1"
  instance_type        = "t3.nano"   # or "t4g.nano" for Graviton
  instance_count       = 3
  vpc_cidr             = "10.0.0.0/16"
  private_subnet_cidr  = "10.0.1.0/24"
  # availability_zone  = "us-east-1a"  # optional
  # key_name           = "my-key"      # optional (SSH will not work without network access)
  # ami_id             = "ami-xxxxxxxx" # optional, overrides Amazon Linux 2023

  tags = {
    Environment = "dev"
    Owner       = "mark"
  }
}
```

## Outputs

| Name                | Description                      |
|---------------------|----------------------------------|
| `vpc_id`            | VPC ID                           |
| `private_subnet_id` | Private subnet ID                |
| `instance_ids`      | List of EC2 instance IDs         |
| `private_ips`       | List of private IP addresses     |
| `security_group_id` | Security group ID                |

## Notes

- Instances are launched in a completely isolated private subnet.
- There is no path to the internet in either direction.
- To access the instances later you would need to add a bastion host, VPN, or SSM (with VPC endpoints) — none of which are included by design.
- The default instance type `t3.nano` is the smallest current general-purpose x86 instance. Use `t4g.nano` if you prefer Graviton (requires an ARM AMI).

## Requirements

- Terraform >= 1.5
- AWS provider >= 5.0
- AWS credentials with permissions to create VPC, Subnet, Route Table, Security Group, and EC2 instances
