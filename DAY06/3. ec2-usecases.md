# 🌐 Terraform EC2 Internet Access Scenarios – Public vs Private Subnet

This guide explains how to **launch EC2 instances in both public and private subnets** using Terraform, and whether they can access the internet based on your VPC setup.

---

## ✅ Situation 1: EC2 in Public Subnet — Internet Access ✅

### 📘 Scenario

You want to launch an EC2 instance in a **public subnet** (`aws_subnet.public_subnet_1`) with:
- Public IP
- Security group allowing outbound traffic
- Subnet associated with **Internet Gateway (IGW)**

### ✅ Terraform Code

```hcl
# Security Group to Allow Internet Access
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow internet access"
  vpc_id      = aws_vpc.main.id

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

  tags = {
    Name = "kkfunda-web-sg"
  }
}

# EC2 in Public Subnet
resource "aws_instance" "public_ec2" {
  ami                         = "ami-03a54d2235b76d9d0"
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public_subnet_1.id
  key_name                    = "your-key-name"  # Replace with real key
  associate_public_ip_address = true
  security_groups             = [aws_security_group.web_sg.id]

  tags = {
    Name = "kkfunda-public-ec2"
  }
}
````

### 🧠 Outcome

✅ EC2 instance can access the internet because:

* It’s in a public subnet
* It has a public IP
* Subnet is associated with IGW
* Security group allows egress traffic

---

## ❌ Situation 2: EC2 in Private Subnet — No Direct Internet ❌

### 📘 Scenario

You want to launch an EC2 instance in a **private subnet** (`aws_subnet.private_subnet_1`) without a public IP.

Even if a security group allows outbound traffic, it cannot connect to the internet **directly**.

### ✅ Terraform Code

```hcl
# EC2 in Private Subnet
resource "aws_instance" "private_ec2" {
  ami                         = "ami-03a54d2235b76d9d0"
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.private_subnet_1.id
  key_name                    = "your-key-name"
  associate_public_ip_address = false
  security_groups             = [aws_security_group.web_sg.id]

  tags = {
    Name = "kkfunda-private-ec2"
  }
}
```

### 🧠 Outcome

❌ EC2 instance **cannot access internet directly**, because:

* It’s in a private subnet
* It has no public IP
* Not associated with IGW

✅ However, if NAT Gateway and private route table are correctly set up (from previous VPC configuration), it **can make outbound internet requests** (like `yum update`), but you **cannot SSH** into it directly.

---

## 🔁 Summary Table

| EC2 Type      | Subnet         | Public IP | Internet Gateway | NAT Gateway | Internet Access |
| ------------- | -------------- | --------- | ---------------- | ----------- | --------------- |
| `public_ec2`  | Public Subnet  | ✅ Yes     | ✅ Yes            | ❌ No        | ✅ Yes           |
| `private_ec2` | Private Subnet | ❌ No      | ❌ No             | ✅ Yes       | ✅ Outbound only |

---

## ✅ How to Test

### SSH into public EC2:

```bash
ssh -i your-key.pem ec2-user@<public-ip>
```

### Ping from public EC2:

```bash
ping google.com
```

### SSH into private EC2?

❌ Not directly possible — you'll need a **bastion host** in public subnet + private key.

---

## 📌 Notes for Teaching

* Emphasize the importance of **public IP + IGW** for internet access.
* Explain how **private subnets use NAT** for outbound only.
* Real-time cloud environments must secure private subnets, even with NAT.

---

