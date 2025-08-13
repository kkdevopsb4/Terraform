# 📘 Terraform Workflow (Basic Understanding)

---

## 🔹 SCOPE (Requirement Phase)

➡️ Manage a file Resource  
➡️ Manage (Create / Update / Delete) a file using Terraform

---

## ✍️ WRITE (Develop / Define Phase)

➡️ Using HCL (HashiCorp Configuration Language),  
➡️ Define / Develop the desired state of your resource(s)

---

## 🚀 APPLY (Manage Phase)

➡️ Manage → Create, Update

---

## 📦 HCL Top Level Blocks

1️⃣ **terraform** – Configure backend, provider versions  
2️⃣ **provider** – Define cloud provider (e.g., AWS)  
3️⃣ **resource** – Define resources like EC2, S3, etc.  
4️⃣ **variable** – Input values to make code dynamic  
5️⃣ **local** – Internal calculated variables  
6️⃣ **output** – Output values after resource creation  
7️⃣ **data** – Read existing infra (does not create)  
8️⃣ **module** – Reuse grouped Terraform code

---


