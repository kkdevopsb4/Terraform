# Terraform Variables – **Five Delivery Methods**  
A hands-on EC2 demo you can run in class

---

## 0️⃣ Prerequisites
* Terraform ≥ 1.5  
* AWS credentials in your shell (e.g., `aws configure`)  
* Region **us-east-1** open for EC2  
* A personal key pair already created in AWS (→ `key_name`)

---

## 📂 Single-file Terraform config (`main.tf`)

```hcl
provider "aws" {
  region = "us-east-1"
}

# ── variable declarations ───────────────────────────────────────
variable "ami_id"          { description = "AMI ID"                type = string }
variable "instance_type"   { description = "EC2 type";             type = string  default = "t2.micro" }
variable "instance_count"  { description = "How many EC2 hosts?";  type = number  default = 1 }
variable "key_name"        { description = "SSH key pair name";    type = string }

# ── resource block that uses all variables ──────────────────────
resource "aws_instance" "demo" {
  count         = var.instance_count
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  tags = { Name = "Demo-${count.index + 1}" }
}
````

> **What stays constant:**
> You never edit the code again; every value is supplied by one of the five
> methods below.

---

## 1️⃣ Interactive prompt (no value supplied anywhere)

```bash
terraform init          # first-run only
terraform apply         # Terraform asks for each required var
# var.ami_id
#  AMI ID
#  Enter a value: ami-0c55b159cbfafe1f0   ← you type
# var.key_name
#  SSH key pair name
#  Enter a value: kkfunda-key            ← you type
```

**Use case** Quick demos & try-outs where you don’t mind typing values.

---

## 2️⃣ Default values (built into the variable block)

Only two variables are mandatory (`ami_id`, `key_name`).
Defaults handle the rest.

```bash
terraform apply \
  -var="ami_id=ami-0c55b159cbfafe1f0" \
  -var="key_name=kkfunda-key"
```

Result → **one** `t2.micro` instance.

**Use case** Low-friction dev environment with sensible fall-backs.

---

## 3️⃣ CLI `-var` (override on the spot)

```bash
terraform apply \
  -var="ami_id=ami-0c55b159cbfafe1f0" \
  -var="key_name=kkfunda-key" \
  -var="instance_type=t3.micro" \
  -var="instance_count=3"
```

Result → three `t3.micro` instances named `Demo-1…3`.

**Use case** CI pipelines or one-off runs where the build system knows the
exact values.

---

## 4️⃣ Environment variables (`TF_VAR_*`)

| Shell / OS                   | Commands                                                                                  |
| ---------------------------- | ----------------------------------------------------------------------------------------- |
| **Linux / macOS (bash/zsh)** | `export TF_VAR_ami_id=ami-0c55b159cbfafe1f0`<br>`export TF_VAR_key_name=kkfunda-key`      |
| **Windows PowerShell**       | `$Env:TF_VAR_ami_id  = "ami-0c55b159cbfafe1f0"`<br>`$Env:TF_VAR_key_name = "kkfunda-key"` |
| **Windows CMD**              | `set TF_VAR_ami_id=ami-0c55b159cbfafe1f0`                                                 |

Then simply run:

```bash
terraform apply
```

**Use case** Keeping secrets out of code / state while letting CI/CD inject
them as environment vars.

---

## 5️⃣ Variable-definition files (`*.tfvars` and `*.auto.tfvars`)

### a. `terraform.tfvars` (auto-loaded)

```hcl
# terraform.tfvars
ami_id        = "ami-0c55b159cbfafe1f0"
key_name      = "kkfunda-key"
instance_type = "t3.small"
instance_count = 2
```

```bash
terraform apply            # file is picked up automatically
```

### b. Named file with `-var-file`

```hcl
# prod.tfvars
ami_id         = "ami-0d99abc1234567890"
key_name       = "kkfunda-prod-key"
instance_type  = "t3.large"
instance_count = 4
```

```bash
terraform apply -var-file="prod.tfvars"
```

### c. `*.auto.tfvars`

A file like `qa.auto.tfvars` is also auto-loaded—great for per-environment
folders.

**Use case** Version-controlled inputs for **dev / qa / prod** with zero CLI
flags.

---

## 🏅 Precedence (highest → lowest)

| Rank | Source                              |
| ---- | ----------------------------------- |
| 1    | Environment vars `TF_VAR_*`         |
| 2    | CLI `-var` and `-var-file`          |
| 3    | `*.auto.tfvars`, `terraform.tfvars` |
| 4    | Defaults in `variable` blocks       |
| 5    | Interactive prompt                  |

Earlier items **override** later ones.


