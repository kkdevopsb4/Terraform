## Terraform Remote State with S3 and DynamoDB Locking

### What is Terraform Remote State?

By default, Terraform stores state locally in a file called `terraform.tfstate`. This file keeps track of all resources Terraform manages. In a team or production environment, storing state locally can cause issues like conflicts, overwrites, or outdated changes.

**Remote state** allows storing the state file in a shared backend like AWS S3. It provides centralized, consistent, and reliable state management.

### Why Remote State is Important

* Shared access to the latest state file
* Enables team collaboration
* Centralized backup with versioning
* Prevents state file corruption
* Supports Terraform modules and automation pipelines

### Why Use DynamoDB?

When multiple people use Terraform together, locking is required. DynamoDB provides a lock mechanism so that only one person can modify infrastructure at a time. It avoids race conditions or duplicate resource creation.

### Components Required

1. **S3 Bucket** - Stores the state file
2. **DynamoDB Table** - Used for state locking
3. **Terraform backend block** - Configures remote backend

### Step-by-Step Setup

#### Step 1: Create an S3 Bucket

```bash
aws s3api create-bucket \
  --bucket kkfunda-tfstate \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

aws s3api put-bucket-versioning \
  --bucket kkfunda-tfstate \
  --versioning-configuration Status=Enabled
```

#### Step 2: Create DynamoDB Table for Locking

```bash
aws dynamodb create-table \
  --table-name kkfunda-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

#### Step 3: Terraform Backend Configuration

File: `terraform.tf`

```hcl
terraform {
  required_version = ">= 1.3"

  backend "s3" {
    bucket         = "kkfunda-tfstate"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "kkfunda-locks"
    encrypt        = true
  }
}
```

Note: This file should not use variables and must be initialized before running any plan or apply.

#### Step 4: Provider Configuration

File: `provider.tf`

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

#### Step 5: Main Configuration

File: `main.tf`

```hcl
resource "aws_instance" "demo" {
  ami                         = "ami-0c55b159cbfafe1f0"
  instance_type               = "t2.micro"
  key_name                    = var.key_name
  subnet_id                   = var.subnet_id
  associate_public_ip_address = true

  tags = {
    Name = "kkfunda-remote-state"
  }
}
```

#### Step 6: Variables

File: `variables.tf`

```hcl
variable "key_name" {
  type    = string
  default = "kkfunda-key"
}

variable "subnet_id" {
  type = string
}
```

#### Step 7: Initialize and Apply

```bash
terraform init
terraform apply
```

### How to Verify

* S3 bucket will contain the file `dev/terraform.tfstate`
* DynamoDB table will show lock entry when `apply` is running

### Best Practices

* Use a different key per environment (e.g., dev/terraform.tfstate, prod/terraform.tfstate)
* Enable versioning in the S3 bucket
* Apply IAM policies to restrict delete actions
* Keep backend block separate and clean
* Never manually edit `.tfstate` file

### Summary Table

| Resource      | Purpose                                |
| ------------- | -------------------------------------- |
| S3 Bucket     | Stores Terraform state file            |
| DynamoDB      | Locks the state file during changes    |
| Backend Block | Configures remote backend in Terraform |

### Remember

* Local state is risky in teams
* S3 helps share the state centrally
* DynamoDB prevents two people from applying at the same time
* This is essential for collaboration, CI/CD, and real projects

### Sample Interview Question

Q: What is the purpose of using S3 and DynamoDB with Terraform?
A: S3 stores the state file centrally so everyone uses the latest version. DynamoDB provides locking so only one user can apply changes at a time, avoiding conflicts.
