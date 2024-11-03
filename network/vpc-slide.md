# Understanding VPC (Virtual Private Cloud)

---

## What is a VPC?

- **Definition**: A Virtual Private Cloud (VPC) is an isolated network within a public cloud that enables you to securely launch AWS resources.
- **Key Features**:
    - **Isolation**: Full control over your virtual networking environment.
    - **Customizable**: You can define your IP address range, subnets, route tables, and security groups.
    - **Connectivity**: Options for private, hybrid, or public connections.

---

## VPC Setup

### Scenario: Application Deployment

1. **Public Subnet**:

    - Hosts public-facing resources like web servers.
    - Accessible via the internet through an Internet Gateway.

2. **Private Subnet**:

    - Hosts backend services like databases.
    - Not directly accessible from the internet – communication only through private connections.

3. **Security**:

    - **Security Groups**: Control inbound and outbound traffic.
    - **Network ACLs**: Additional layer of traffic filtering.

---
## Terraform Code: Building a VPC for a Social Media Network

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_vpc" "social_media_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "social-media-vpc"
  }
}

resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.social_media_vpc.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-west-2a"
  tags = {
    Name = "public-subnet"
  }
}

resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.social_media_vpc.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-west-2a"
  tags = {
    Name = "private-subnet"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.social_media_vpc.id
  tags = {
    Name = "social-media-igw"
  }
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.social_media_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table_association" "public_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_nat_gateway" "nat_gw" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_subnet.id
  tags = {
    Name = "social-media-nat-gw"
  }
}

resource "aws_eip" "nat_eip" {
  vpc = true
}

resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.social_media_vpc.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat_gw.id
  }
  tags = {
    Name = "private-route-table"
  }
}

resource "aws_route_table_association" "private_association" {
  subnet_id      = aws_subnet.private_subnet.id
  route_table_id = aws_route_table.private_rt.id
}
```
### Explanation of the Code

This Terraform code defines the infrastructure for a simple network setup in AWS using resources such as VPCs, subnets, an internet gateway, a NAT gateway, and route tables.

1. **Provider Configuration**
   ```hcl
   provider "aws" {
     region = "us-west-2"
   }
   ```
    - Configures AWS as the provider and sets the region to `us-west-2`.

2. **VPC (Virtual Private Cloud)**
   ```hcl
   resource "aws_vpc" "social_media_vpc" {
     cidr_block           = "10.0.0.0/16"
     enable_dns_support   = true
     enable_dns_hostnames = true
     tags = {
       Name = "social-media-vpc"
     }
   }
   ```
    - Creates a VPC with a CIDR block of `10.0.0.0/16`, enabling DNS support and hostnames.

3. **Subnets**
    - **Public Subnet**
      ```hcl
      resource "aws_subnet" "public_subnet" {
        vpc_id                  = aws_vpc.social_media_vpc.id
        cidr_block              = "10.0.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = "us-west-2a"
        tags = {
          Name = "public-subnet"
        }
      }
      ```
        - A public subnet within the VPC (`10.0.1.0/24`) with automatic public IP assignment on launch.

    - **Private Subnet**
      ```hcl
      resource "aws_subnet" "private_subnet" {
        vpc_id            = aws_vpc.social_media_vpc.id
        cidr_block        = "10.0.2.0/24"
        availability_zone = "us-west-2a"
        tags = {
          Name = "private-subnet"
        }
      }
      ```
        - A private subnet within the VPC (`10.0.2.0/24`), not exposed directly to the internet.

4. **Internet Gateway**
   ```hcl
   resource "aws_internet_gateway" "igw" {
     vpc_id = aws_vpc.social_media_vpc.id
     tags = {
       Name = "social-media-igw"
     }
   }
   ```
    - Attaches an internet gateway to the VPC for internet connectivity for resources in the public subnet.

5. **Route Tables**
    - **Public Route Table**
      ```hcl
      resource "aws_route_table" "public_rt" {
        vpc_id = aws_vpc.social_media_vpc.id
        route {
          cidr_block = "0.0.0.0/0"
          gateway_id = aws_internet_gateway.igw.id
        }
        tags = {
          Name = "public-route-table"
        }
      }
      ```
        - A route table that routes internet traffic (`0.0.0.0/0`) through the internet gateway for the public subnet.

      ```hcl
      resource "aws_route_table_association" "public_association" {
        subnet_id      = aws_subnet.public_subnet.id
        route_table_id = aws_route_table.public_rt.id
      }
      ```
        - Associates the public subnet with the public route table.

    - **Private Route Table**
      ```hcl
      resource "aws_route_table" "private_rt" {
        vpc_id = aws_vpc.social_media_vpc.id
        route {
          cidr_block     = "0.0.0.0/0"
          nat_gateway_id = aws_nat_gateway.nat_gw.id
        }
        tags = {
          Name = "private-route-table"
        }
      }
      ```
        - A route table that routes traffic from the private subnet to the internet through the NAT gateway.

      ```hcl
      resource "aws_route_table_association" "private_association" {
        subnet_id      = aws_subnet.private_subnet.id
        route_table_id = aws_route_table.private_rt.id
      }
      ```
        - Associates the private subnet with the private route table.

6. **NAT Gateway and Elastic IP**
   ```hcl
   resource "aws_nat_gateway" "nat_gw" {
     allocation_id = aws_eip.nat_eip.id
     subnet_id     = aws_subnet.public_subnet.id
     tags = {
       Name = "social-media-nat-gw"
     }
   }
   
   resource "aws_eip" "nat_eip" {
     vpc = true
   }
   ```
    - A NAT gateway and an Elastic IP to allow instances in the private subnet to access the internet while keeping them inaccessible from outside.

### Block Diagram
Below is the block diagram illustrating this setup:

```
                +---------------------------------------+
                |              AWS VPC                 |
                |         CIDR: 10.0.0.0/16            |
                +---------------------------------------+
                          |                 |
          +---------------+                 +------------------+
          |                                                     |
+----------------------+                         +----------------------+
|    Public Subnet     |                         |    Private Subnet    |
|   CIDR: 10.0.1.0/24  |                         |   CIDR: 10.0.2.0/24  |
|    AZ: us-west-2a    |                         |    AZ: us-west-2a    |
|  Internet Access via |                         | Internet Access via  |
|  Internet Gateway    |                         |    NAT Gateway       |
+----------------------+                         +----------------------+
          |                                                     |
+----------------------+                         +----------------------+
| Internet Gateway     |                         |    NAT Gateway       |
| (Attached to VPC)    |                         | Allocated EIP        |
+----------------------+                         +----------------------+
```
---

## Benefits of Using a VPC

- **Enhanced Security**: Isolated environment ensures better protection.
- **Scalability**: Customizable architecture for varying needs.
- **Flexibility**: Multiple connectivity options (VPN, Direct Connect).

---

## Summary

- A VPC provides a secure and customizable cloud networking solution.
- Think of it as a gated community in a public cloud city for easier visualization.

---

## Q&A
