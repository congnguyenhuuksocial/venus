### AWS Network ACL (NACL) Overview

**Network ACLs (Access Control Lists)** in AWS are **stateless firewalls** that provide an additional layer of security at the **subnet level** within a Virtual Private Cloud (VPC). They act as a filter for both **inbound and outbound traffic**, controlling which packets are allowed to enter and leave a subnet based on rules you define.

### 1. **What is a Network ACL?**
- A **Network ACL** is a set of rules, similar to a traditional firewall, that you can use to allow or deny traffic to and from subnets in your VPC.
- Unlike Security Groups (which are stateful), Network ACLs are **stateless**. This means that both inbound and outbound rules must be explicitly defined.
- NACLs operate at the **subnet level**, while Security Groups operate at the **instance level**.

#### Key Characteristics:
- **Stateless**: Each request and response is evaluated independently. You need to define both **inbound** and **outbound** rules for bidirectional traffic.
- **Subnet-level Control**: Applies to all instances within the associated subnet.
- **Ordered Rules**: Rules are evaluated in numerical order (starting from the lowest number).
- **Default Deny**: If a packet does not match any rule, it is denied by default.

### 2. **Default Network ACL**
When you create a VPC, AWS automatically creates a **default NACL** that:
- Allows all inbound and outbound traffic.
- Is associated with all subnets in the VPC by default.

You can modify the default NACL or create custom NACLs for more granular control.

### 3. **Network ACL Rules**
Each rule in a NACL specifies:
- **Rule Number**: A unique identifier (from 1 to 32,766). Rules are evaluated in ascending order.
- **Protocol**: The protocol type (e.g., TCP, UDP, ICMP, or all).
- **Port Range**: The range of ports (e.g., 22 for SSH, 80 for HTTP).
- **Source/Destination CIDR**: The IP address range for inbound (source) or outbound (destination) traffic.
- **Action**: `Allow` or `Deny`.

#### Example Rules:
| Rule # | Type   | Protocol | Port Range | Source/Destination | Action |
|--------|--------|----------|------------|--------------------|--------|
| 100    | Inbound| TCP      | 80         | `0.0.0.0/0`        | Allow  |
| 200    | Inbound| TCP      | 22         | `192.168.1.0/24`   | Allow  |
| 300    | Outbound| TCP     | 443        | `0.0.0.0/0`        | Allow  |
| *      | Inbound| All      | All        | `0.0.0.0/0`        | Deny   |
| *      | Outbound| All     | All        | `0.0.0.0/0`        | Deny   |

- The `*` rule at the end is the **default deny rule**, which blocks all traffic not explicitly allowed by previous rules.

### 4. **Stateful vs. Stateless**
- **Security Groups** are **stateful**, meaning if an inbound request is allowed, the response is automatically allowed.
- **Network ACLs** are **stateless**, so you need to explicitly define **both inbound and outbound rules** for bidirectional communication.

#### Example:
- If you allow inbound traffic on port 80 (HTTP) in a NACL, you must also explicitly allow outbound traffic for the response on port 80.

### 5. **Associating a NACL with a Subnet**
- Each subnet in a VPC can be associated with **one NACL** at a time.
- If you do not explicitly associate a subnet with a custom NACL, it uses the **default NACL**.
- You can associate multiple subnets with the same NACL.

#### Example CLI Command:
```bash
aws ec2 associate-network-acl \
    --network-acl-id acl-12345678 \
    --subnet-id subnet-12345678
```

### 6. **Default vs. Custom NACLs**
- **Default NACL**:
    - Allows all inbound and outbound traffic by default.
    - Cannot be deleted, but can be modified.
- **Custom NACL**:
    - By default, denies all inbound and outbound traffic.
    - Provides granular control over allowed and denied traffic.

### 7. **Common NACL Use Cases**
- **Public Subnet NACL**: Allows HTTP, HTTPS, and SSH traffic from the internet.
- **Private Subnet NACL**: Blocks all inbound traffic from the internet, only allowing traffic from the VPC.
- **Isolation of Subnets**: Create custom NACLs to enforce strict rules between different subnets (e.g., separating web servers from databases).

### 8. **NACLs vs. Security Groups**
| Feature                  | Network ACL (NACL)      | Security Group           |
|--------------------------|-------------------------|--------------------------|
| Level of Control         | Subnet level            | Instance level           |
| Stateful/Stateless       | Stateless               | Stateful                 |
| Default Behavior         | Denies all traffic by default (custom NACL) | Allows all outbound traffic by default |
| Rule Evaluation Order    | Evaluates rules in order (lowest first) | Evaluates all rules     |
| Number of Rules          | Up to 20 inbound and 20 outbound rules per NACL | More flexible, typically fewer rules needed |
| Use Case                 | Broad subnet-level access control | Fine-grained instance-level access control |

### 9. **Best Practices for Network ACLs**
- **Use Security Groups for Instance-level Protection**: Use NACLs for broad subnet-level control and Security Groups for fine-grained instance-level security.
- **Limit the Use of `0.0.0.0/0`**: Avoid allowing all traffic (`0.0.0.0/0`) unless absolutely necessary, especially for inbound rules.
- **Test Changes in Staging**: Before applying changes in production, test them in a staging environment to prevent accidental disruptions.
- **Order Rules Carefully**: Place the most specific and restrictive rules first to minimize unnecessary traffic evaluation.
- **Monitor and Audit**: Use AWS CloudTrail and VPC Flow Logs to track changes and monitor traffic for compliance and security.

### 10. **Troubleshooting NACL Issues**
- **Unexpected Traffic Blocked**: Check that both inbound and outbound rules allow the traffic. Since NACLs are stateless, both directions need explicit rules.
- **Connectivity Problems**: Verify that the subnet is associated with the correct NACL. Check for conflicting rules that might be inadvertently blocking traffic.
- **Testing with VPC Flow Logs**: Enable VPC Flow Logs to capture traffic information and identify the cause of blocked or dropped traffic.

### **Example NACL Configuration**
#### Public Subnet NACL:
```bash
aws ec2 create-network-acl \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=PublicSubnetACL}]'
```

#### Add Inbound Rule (Allow HTTP Traffic):
```bash
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --egress false \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow
```

#### Add Outbound Rule (Allow HTTP Response):
```bash
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --egress true \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow
```

### **Summary**
Network ACLs are powerful tools for controlling traffic at the subnet level in AWS. By using NACLs alongside Security Groups, you can implement a **layered security model** that provides robust protection for your VPC resources.
