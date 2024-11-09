### AWS WAF (Web Application Firewall) Overview

**AWS WAF (Web Application Firewall)** is a security service that helps protect your web applications from common web exploits and attacks that could affect application availability, compromise security, or consume excessive resources. It allows you to create rules to **filter and block malicious traffic** based on specific conditions.

### 1. **What is AWS WAF?**
AWS WAF is a **managed firewall** service that works at the **application layer (Layer 7)**. It inspects incoming HTTP/S traffic to your web applications and provides a way to define rules to allow, block, or count requests based on custom criteria.

#### Key Benefits:
- **Protects against common attacks** (e.g., SQL injection, cross-site scripting).
- **Highly customizable** with rule-based filtering.
- **Seamless integration** with AWS services like CloudFront, API Gateway, and Application Load Balancer (ALB).
- **Real-time monitoring** and detailed logging for threat analysis.

### 2. **AWS WAF Components**
- **Web ACL (Web Access Control List)**: A set of rules that define how to inspect and handle web requests. Each Web ACL can be associated with one or more AWS resources (e.g., CloudFront, API Gateway).
- **Rules**: Individual filters that specify conditions to look for in web requests (e.g., IP addresses, HTTP headers, request size).
- **Rule Groups**: A collection of rules that can be managed together. AWS Managed Rules are pre-configured rule groups provided by AWS.
- **Conditions**: Specific criteria for a rule (e.g., IP addresses, geo-location, SQL injection patterns).

### 3. **How AWS WAF Works**
1. **Traffic Inspection**: AWS WAF sits in front of your web application (e.g., behind CloudFront or ALB) and inspects incoming HTTP/S requests.
2. **Rule Evaluation**: Each request is evaluated against the rules in the associated Web ACL.
3. **Action**: Based on the evaluation, AWS WAF applies the specified action:
    - **Allow**: The request is forwarded to the application.
    - **Block**: The request is denied, and the client receives a 403 Forbidden response.
    - **Count**: The request is counted for monitoring purposes but not blocked.

### 4. **AWS WAF Rule Types**
- **IP Set Rule**: Filters requests based on IP addresses or CIDR blocks. Useful for blocking known malicious IPs.
- **Geo Match Rule**: Blocks or allows requests based on the geographic location of the IP address.
- **Size Constraint Rule**: Blocks requests that exceed a specified size (e.g., body, headers).
- **Rate-Based Rule**: Protects against DDoS attacks by blocking IPs that exceed a defined request rate (e.g., 1,000 requests per 5 minutes).
- **Managed Rule Groups**: Pre-configured rules provided by AWS to protect against common threats (e.g., OWASP Top 10 vulnerabilities like SQL injection and XSS).

#### Example Rule:
A rule to block requests from a specific country:
```json
{
  "GeoMatchStatement": {
    "CountryCodes": ["CN", "RU"]
  },
  "Action": {
    "Block": {}
  }
}
```

### 5. **AWS Managed Rules**
AWS provides **Managed Rule Groups** that offer a pre-configured set of rules for common use cases. Examples include:
- **AWS Managed Rules - Core Rule Set**: Protects against common attacks such as SQL injection and XSS.
- **AWS Managed Rules - Bot Control**: Identifies and blocks known bots and automated scripts.
- **AWS Managed Rules - Anonymous IP List**: Blocks requests from IP addresses associated with VPNs, Tor nodes, and proxies.

### 6. **Setting Up AWS WAF**
To set up AWS WAF, follow these steps:

1. **Create a Web ACL**:
    - Go to the AWS WAF Console.
    - Click **Create Web ACL**.
    - Choose the AWS resource to associate (e.g., CloudFront, ALB, API Gateway).
    - Define the default action (Allow or Block).

2. **Add Rules**:
    - Add custom rules or use AWS Managed Rule Groups.
    - Define the conditions for each rule (e.g., IP Set, Geo Match, Rate-Based).

3. **Associate the Web ACL**:
    - Attach the Web ACL to your AWS resource (CloudFront, ALB, or API Gateway).
    - The Web ACL will now inspect and filter incoming traffic based on the defined rules.

### 7. **Monitoring and Logging**
- **CloudWatch Metrics**: AWS WAF provides metrics such as `AllowedRequests`, `BlockedRequests`, and `CountedRequests`.
- **AWS WAF Logs**: You can enable detailed logging to an S3 bucket, CloudWatch Logs, or AWS Kinesis Data Firehose for deeper analysis.
- **AWS WAF Security Automations**: Use AWS WAF Security Automations to deploy common WAF configurations automatically.

#### Example CloudWatch Alarm:
Create an alarm to notify you if the number of blocked requests exceeds a threshold:
```json
{
  "AlarmName": "HighBlockedRequests",
  "MetricName": "BlockedRequests",
  "Namespace": "AWS/WAF",
  "Statistic": "Sum",
  "Period": 300,
  "Threshold": 1000,
  "ComparisonOperator": "GreaterThanThreshold"
}
```

### 8. **Best Practices for AWS WAF**
- **Use Managed Rules**: Start with AWS Managed Rule Groups to get out-of-the-box protection against common threats.
- **Enable Logging**: Monitor logs to analyze blocked requests and fine-tune your rules.
- **Apply Rate-Based Rules**: Use rate-based rules to prevent abuse and DDoS attacks.
- **Test Rules First**: Use the **Count** action to test new rules before applying them to avoid unintended blocking of legitimate traffic.
- **Use IP Sets for Known Threats**: Maintain a list of known malicious IPs and block them using IP Set rules.
- **Leverage AWS Firewall Manager**: For multi-account environments, use AWS Firewall Manager to centrally manage AWS WAF rules across multiple AWS accounts.

### 9. **AWS WAF Pricing**
AWS WAF pricing is based on three main factors:
- **Web ACL Charges**: A fixed monthly fee for each Web ACL.
- **Rules Charges**: A fee for each rule added to the Web ACL.
- **Request Charges**: A fee for the number of requests processed by AWS WAF.

AWS also offers a **free tier**, which includes:
- **Up to 1 Web ACL**.
- **Up to 10 rules** per Web ACL.
- **Up to 1 million web requests per month**.

### 10. **AWS WAF Use Cases**
- **API Protection**: Use AWS WAF with API Gateway to protect APIs from malicious traffic.
- **Web Application Security**: Use AWS WAF with CloudFront or ALB to protect websites from common attacks like XSS and SQL injection.
- **Bot Mitigation**: Use AWS Managed Rules to block unwanted bot traffic, preserving bandwidth and resources for legitimate users.
- **DDoS Protection**: Combine AWS WAF with AWS Shield Advanced for enhanced DDoS protection.

### **Summary**
AWS WAF provides a robust and flexible way to protect your web applications from a wide range of threats. By leveraging AWS Managed Rules, custom rules, and integration with other AWS services, AWS WAF can help you enhance the security of your applications without significant performance overhead.
