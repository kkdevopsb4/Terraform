# Terraform **Workspaces** – Hands-On Guide  
*(Using the same VPC + EC2 project you built earlier)*

---

## 0️⃣ Why Workspaces?

A **workspace** is an *isolated snapshot of state* inside the **same folder of
code**.  
Think of it as *“multiple state files managed for you by Terraform”*.

With workspaces you can:

| Benefit                 | What it means in class                           |
|-------------------------|--------------------------------------------------|
| **Single source code**  | One repo, one set of `.tf` files                 |
| **Multiple environments** | Dev, QA, Prod each get their own state          |
| **Fast switching**      | Change environment with a single command         |

> ⚠️ **Scope of this doc:** local workspaces only – no remote–state topics.

---

## 1️⃣ Recap – Code & Variables

You already have:

* `main.tf`, `variables.tf`, `provider.tf`, `terraform.tf`  
* Two var-files for environments:

```

dev.tfvars   # environment = "dev",  smaller CIDR, t3.micro, etc.
prod.tfvars  # environment = "prod", larger CIDR, t3.large, etc.

````

The resource names (`aws_vpc.main`, `aws_instance.public_ec2`, …) **do not
change**; workspaces keep everything separate.

---

## 2️⃣ Workspace Command Reference

| Command                                | What it does                              |
|----------------------------------------|-------------------------------------------|
| `terraform workspace list`             | Show all workspaces (bold = current)      |
| `terraform workspace new dev`          | Create **dev** and switch to it           |
| `terraform workspace select prod`      | Switch to **prod**                        |
| `terraform workspace show`             | Print current workspace name              |
| `terraform workspace delete qa`        | Delete **qa** (must be on a different one)|
| `terraform workspace rename dev devold`| Rename existing workspace                 |

> The first workspace is always called **`default`**.

---

## 3️⃣ End-to-End Walk-Through

### 🔹 Step 1 – Initialize once

```bash
terraform init
````

### 🔹 Step 2 – Create and apply **dev**

```bash
terraform workspace new dev           # create & switch
terraform apply -var-file=dev.tfvars
```

*Creates VPC `10.10.0.0/16`, small EC2, etc.*

### 🔹 Step 3 – Create and apply **prod**

```bash
terraform workspace new prod          # create & switch
terraform apply -var-file=prod.tfvars
```

*Creates separate VPC `10.0.0.0/16`, larger EC2, etc.*

### 🔹 Step 4 – Verify isolation

```bash
terraform workspace select dev
terraform state list | grep aws_vpc      # shows only dev VPC

terraform workspace select prod
terraform state list | grep aws_vpc      # shows only prod VPC
```

### 🔹 Step 5 – Destroy **dev** only

```bash
terraform workspace select dev
terraform destroy -var-file=dev.tfvars
```

Prod resources remain intact.

---

## 4️⃣ How Terraform Stores Workspace State (Local)

* `.terraform/`

  * `terraform.tfstate`  ← for **default**
  * `env:/dev/terraform.tfstate`
  * `env:/prod/terraform.tfstate`

You never touch these directly—Terraform handles the paths.

---

## 5️⃣ Typical Workflow Cheatsheet

```bash
# create workspaces once
terraform workspace new dev
terraform workspace new prod

# Dev pipeline
terraform workspace select dev
terraform apply -var-file=dev.tfvars

# Prod pipeline
terraform workspace select prod
terraform apply -var-file=prod.tfvars
```

> *Tip:* Make the workspace name an environment variable in CI
> (`$TF_WORKSPACE`) and scripts stay identical for every stage.

---

## 6️⃣ FAQ for Students

| Question                                | Short Answer                                                                      |
| --------------------------------------- | --------------------------------------------------------------------------------- |
| **Do I need different resource names?** | No. Same code, same names. Workspaces keep state separate.                        |
| **Where are the state files?**          | Inside `.terraform/env:/<workspace>/terraform.tfstate`.                           |
| **Can I use `terraform output`?**       | Yes—returns values from the *current* workspace only.                             |
| **What happens if I forget `select`?**  | Terraform operates on whatever workspace is active (check with `workspace show`). |
| **Can I destroy all at once?**          | Not with one command—you must loop through workspaces.                            |

---

## 7️⃣ Key Takeaways

1. **One folder, many environments** – workspaces give isolation without code duplication.
2. **Always set workspace explicitly** in scripts to avoid accidents.
3. **Var-files complement workspaces** – flags like `-var-file=dev.tfvars` still determine *what* you build; workspace decides *where state lives*.

