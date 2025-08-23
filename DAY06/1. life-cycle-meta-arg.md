# ⚙️ Terraform `lifecycle` Meta-Argument – EC2 Real-Time Use Cases

This document explains the `lifecycle` meta-argument in Terraform with **only EC2 (`aws_instance`) examples**, ideal for teaching DevOps infrastructure behavior control.

---

## 📘 What is `lifecycle`?

The `lifecycle` block in Terraform provides **fine-grained control** over how resources behave during:

- **Creation**
- **Updates**
- **Destruction**

It is written **inside a `resource` block** and supports these keys:

- `create_before_destroy`
- `prevent_destroy`
- `ignore_changes`

---

## 🧾 Basic Syntax (EC2 Example)

```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  key_name      = "kkfunda-key"

  lifecycle {
    create_before_destroy = true
    prevent_destroy        = false
    ignore_changes         = [tags, user_data]
  }
}
````

---

## 🔹 1. `create_before_destroy`

### ✅ Use Case:

Ensure **zero downtime** when making changes (e.g., AMI update).

```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  key_name      = "kkfunda-key"

  lifecycle {
    create_before_destroy = true
  }
}
```

### 💬 Explanation:

> “Terraform usually destroys the old EC2 before creating the new one when a change is detected. With `create_before_destroy`, it launches the new EC2 first, then removes the old one, ensuring **no downtime** during updates.”

---

## 🔹 2. `prevent_destroy`

### ✅ Use Case:

**Block accidental deletion** of critical EC2 instances.

```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  key_name      = "kkfunda-key"

  lifecycle {
    prevent_destroy = true
  }
}
```

### 💬 Explanation:

> “This is useful when a production EC2 instance must be protected. If someone tries to run `terraform destroy`, Terraform will throw an error unless `prevent_destroy` is removed.”

---

## 🔹 3. `ignore_changes`

### ✅ Use Case:

**Ignore manual changes** made in the AWS console for selected fields.

```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  key_name      = "kkfunda-key"

  user_data = file("setup.sh")

  tags = {
    Name = "KKFunda-EC2"
  }

  lifecycle {
    ignore_changes = [tags, user_data]
  }
}
```

### 💬 Explanation:

> “Sometimes the support or operations team updates tags or user\_data scripts manually in the console. We don’t want Terraform to reset them during the next apply. So, we use `ignore_changes` to skip those attributes.”

---

## 📊 Summary Table

| Lifecycle Option        | Use Case                               | Behavior in EC2                                 |
| ----------------------- | -------------------------------------- | ----------------------------------------------- |
| `create_before_destroy` | Zero-downtime deployment               | Creates new EC2 before deleting old             |
| `prevent_destroy`       | Prevent accidental deletion            | Throws error if someone tries to delete the EC2 |
| `ignore_changes`        | Allow manual changes outside Terraform | Skips applying changes to specified attributes  |

---


