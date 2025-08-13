
# 📘 Terraform – Local File Resource Example with Full Workflow

This example demonstrates how to use **Terraform's local provider** to manage a simple file (`test.txt`) on your local system.

---

## 🧾 Terraform Code Explanation

```hcl
terraform {
  required_version = ">1.9.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "2.5.3"
    }
  }
}

provider "local" {
  # No configuration required
}

resource "local_file" "main" {
  filename = "test.txt"
  content  = "KK FUNDA DevOps with AWS"
}
````

---

## 🔄 Terraform Workflow (Step-by-Step)

### 1️⃣ `terraform init` – Initialization

```bash
terraform init
```

**What Happens:**

* Downloads the `hashicorp/local` provider (v2.5.3).
* Sets up `.terraform/` directory.
* Prepares working directory for Terraform operations.

**Output:**

```
Initializing provider plugins...
Downloading hashicorp/local...
Terraform has been successfully initialized!
```

---

### 2️⃣ `terraform validate` – Validate Configuration

```bash
terraform validate
```

**What Happens:**

* Checks syntax of the `.tf` files.
* Validates internal consistency.

**Output:**

```
Success! The configuration is valid.
```

---

### 3️⃣ `terraform plan` – Preview Execution Plan

```bash
terraform plan
```

**What Happens:**

* Shows what Terraform will do (create `test.txt` file).
* No actual changes yet — just a dry run.

**Sample Output:**

```
+ local_file.main will be created
  filename: "test.txt"
  content:  "KK FUNDA DevOps with AWS"
```

---

### 4️⃣ `terraform apply` – Apply Configuration

```bash
terraform apply
```

**What Happens:**

* Applies the plan and creates the file `test.txt`.
* Adds the file to the Terraform state.

**Result:**

* `test.txt` is created with content:

  ```
  KK FUNDA DevOps with AWS
  ```

---

### 5️⃣ `terraform destroy` – Tear Down Resource

```bash
terraform destroy
```

**What Happens:**

* Deletes `test.txt`.
* Updates the state to reflect the deletion.

**Output:**

```
Destroy complete! Resources: 1 destroyed.
```

---

## 📦 Summary Table

| Step       | Command              | Purpose                          | Outcome                  |
| ---------- | -------------------- | -------------------------------- | ------------------------ |
| `init`     | `terraform init`     | Download provider, setup project | Initializes working dir  |
| `validate` | `terraform validate` | Syntax and structure check       | Confirms config is valid |
| `plan`     | `terraform plan`     | Show what Terraform will do      | Shows create action      |
| `apply`    | `terraform apply`    | Actually creates file            | Creates `test.txt`       |
| `destroy`  | `terraform destroy`  | Deletes the file                 | Deletes `test.txt`       |

---

## 🧠 Why Use the Local Provider?

The `local` provider is useful for:

* Learning and practicing Terraform without cloud setup.
* Generating configuration files dynamically.
* Creating `.env`, scripts, metadata locally.

---

> 📚 Prepared for: **KK FUNDA DevOps with AWS – Terraform Training**

