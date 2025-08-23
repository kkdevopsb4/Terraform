# 🏗️ Terraform VPC Infrastructure Setup – Mumbai Region

## ✅ Scenario

Create a VPC in the **Mumbai (`ap-south-1`) region** with:

- ✅ 1 VPC
- ✅ 2 Availability Zones: `ap-south-1a`, `ap-south-1b`
- ✅ 4 Subnets:
  - 2 Public Subnets (one in each AZ)
  - 2 Private Subnets (one in each AZ)
- ✅ 1 Internet Gateway (IGW)
- ✅ 1 NAT Gateway (in a public subnet)
- ✅ Route Tables for public and private subnets

Everything is included in **one `main.tf` file**.

---

## 🧾 Terraform Code – `main.tf`

```hcl
provider "aws" {
  region = "ap-south-1"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "kkfunda-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags = {
    Name = "kkfunda-igw"
  }
}

# Public Subnets
resource "aws_subnet" "public_subnet_1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "kkfunda-public-1"
  }
}

resource "aws_subnet" "public_subnet_2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "ap-south-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "kkfunda-public-2"
  }
}

# Private Subnets
resource "aws_subnet" "private_subnet_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "ap-south-1a"
  tags = {
    Name = "kkfunda-private-1"
  }
}

resource "aws_subnet" "private_subnet_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "ap-south-1b"
  tags = {
    Name = "kkfunda-private-2"
  }
}

# Elastic IP for NAT Gateway
resource "aws_eip" "nat_eip" {
  domain = "vpc"
}

# NAT Gateway
resource "aws_nat_gateway" "nat_gw" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_subnet_1.id
  tags = {
    Name = "kkfunda-nat"
  }

  depends_on = [aws_internet_gateway.igw]
}

# Public Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "kkfunda-public-rt"
  }
}

resource "aws_route_table_association" "public_assoc_1" {
  subnet_id      = aws_subnet.public_subnet_1.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table_association" "public_assoc_2" {
  subnet_id      = aws_subnet.public_subnet_2.id
  route_table_id = aws_route_table.public_rt.id
}

# Private Route Table
resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat_gw.id
  }

  tags = {
    Name = "kkfunda-private-rt"
  }
}

resource "aws_route_table_association" "private_assoc_1" {
  subnet_id      = aws_subnet.private_subnet_1.id
  route_table_id = aws_route_table.private_rt.id
}

resource "aws_route_table_association" "private_assoc_2" {
  subnet_id      = aws_subnet.private_subnet_2.id
  route_table_id = aws_route_table.private_rt.id
}
````

---

## 🔍 Explanation (for teaching)

| Component       | Purpose                                                         |
| --------------- | --------------------------------------------------------------- |
| VPC             | Main container for all networking resources                     |
| IGW             | Allows internet access to public subnets                        |
| Public Subnets  | EC2s launched here get public IPs and access to the internet    |
| Private Subnets | No direct internet access; routed via NAT Gateway               |
| NAT Gateway     | Provides internet access to private subnets (e.g., for updates) |
| EIP             | Static IP required for NAT Gateway                              |
| Route Tables    | Public RT routes via IGW; Private RT routes via NAT GW          |

---

## ✅ How to Run

```bash
terraform init
terraform plan
terraform apply
```


