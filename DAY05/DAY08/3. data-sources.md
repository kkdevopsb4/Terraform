# Terraform AWS Data-Source Demo  
*(AMI lookup + dynamic Availability Zone selection)*

```

.
├── datasources.tf   # <─ NEW: aws\_ami & aws\_availability\_zones
├── main.tf          # resources that consume the data-sources
├── outputs.tf       # handy outputs for demo
├── provider.tf      # AWS provider (region comes from variables)
├── terraform.tf     # Terraform version & provider constraint
└── variables.tf     # all typed input variables

````

---

## 1️⃣ terraform.tf

```hcl
terraform {
  required_version = ">= 1.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
````

---

## 2️⃣ provider.tf

```hcl
provider "aws" {
  region = var.aws_region
}
```

---

## 3️⃣ variables.tf

```hcl
########################
# Global / Misc
########################
variable "aws_region" {
  description = "AWS region for all resources"
  type        = string
  default     = "ap-south-1"
}

variable "environment" {
  description = "Env tag applied to every resource"
  type        = string
  default     = "dev"
}

variable "common_tags" {
  description = "Base tags"
  type        = map(string)
  default = {
    ManagedBy = "Terraform"
    Owner     = "kkfunda"
  }
}

########################
# Networking
########################
variable "vpc_cidr" {
  description = "CIDR block for the new VPC"
  type        = string
  default     = "10.20.0.0/16"
}

variable "public_subnet_cidr" {
  description = "CIDR for the public subnet"
  type        = string
  default     = "10.20.1.0/24"
}

########################
# EC2
########################
variable "instance_type" {
  description = "Instance size"
  type        = string
  default     = "t2.micro"
}

variable "key_name" {
  description = "Existing key-pair for SSH"
  type        = string
  default     = "kkfunda-key"
}
```

---

## 4️⃣ datasources.tf  (★ Focus of today’s lesson)

```hcl
# ── 1. Latest Amazon Linux 2 AMI ───────────────────────────────────
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# ── 2. Get ALL available AZs in the chosen region ──────────────────
data "aws_availability_zones" "available" {
  state = "available"
}
```

**Theory recap for students**

| Data source              | Why we use it                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `aws_ami`                | Always pulls the latest Amazon Linux image – no hard-coding, automatic patching.                              |
| `aws_availability_zones` | Returns an array (e.g., `["ap-south-1a","ap-south-1b","ap-south-1c"]`), letting us pick AZs programmatically. |

---

## 5️⃣ main.tf

```hcl
# Reusable tag helper
locals {
  tags = merge(var.common_tags, { Env = var.environment })
}

# VPC
resource "aws_vpc" "demo" {
  cidr_block = var.vpc_cidr
  tags       = merge(local.tags, { Name = "kkfunda-vpc" })
}

# Public Subnet in FIRST available AZ
resource "aws_subnet" "public_1" {
  vpc_id                  = aws_vpc.demo.id
  cidr_block              = var.public_subnet_cidr
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = true
  tags                    = merge(local.tags, { Name = "kkfunda-public-1" })
}

# Security Group (SSH only)
resource "aws_security_group" "web_sg" {
  name        = "kkfunda-web-sg"
  description = "Allow SSH"
  vpc_id      = aws_vpc.demo.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(local.tags, { Name = "kkfunda-web-sg" })
}

# EC2 – uses both data sources!
resource "aws_instance" "web" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = var.instance_type
  subnet_id                   = aws_subnet.public_1.id
  key_name                    = var.key_name
  associate_public_ip_address = true
  vpc_security_group_ids      = [aws_security_group.web_sg.id]

  tags = merge(local.tags, { Name = "kkfunda-web" })
}
```

---

## 6️⃣ outputs.tf

```hcl
output "ami_id_used" {
  description = "AMI ID chosen by the aws_ami data source"
  value       = data.aws_ami.amazon_linux.id
}

output "all_azs_in_region" {
  description = "List of Availability Zones returned"
  value       = data.aws_availability_zones.available.names
}

output "ec2_public_ip" {
  description = "Public IP of the new instance"
  value       = aws_instance.web.public_ip
}
```

---

## 🧪 Demo Commands

```bash
terraform init
terraform plan
terraform apply   # review, then type 'yes'
```

Expected output snippet:

```
Outputs:
ami_id_used = "ami-0c55b159cbfafe1f0"
all_azs_in_region = [
  "ap-south-1a",
  "ap-south-1b",
  "ap-south-1c",
]
ec2_public_ip = "13.234.56.78"
```

---

## 🧑‍🏫 Key Talking Points

1. **`data` ≠ `resource`** – it only **reads**; nothing is created.
2. **Automatic updates** – whenever Amazon releases a new AMI, `aws_ami` picks it up on the next `plan/apply`.
3. **Region-independence** – Switch `aws_region` to `us-east-1`; AZ data source adapts without edits.
4. **Cost-free lookups** – Data sources perform an API read; no charges beyond normal AWS API calls.
5. **Debug in console** – `terraform console` → `data.aws_availability_zones.available.names`.


