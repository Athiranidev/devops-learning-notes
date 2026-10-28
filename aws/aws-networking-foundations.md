# AWS Networking Foundations — Quick Revision Notes

## 1. Load Balancer

- Receives incoming application requests.
- Distributes requests across configured targets.
- Health checks help identify unhealthy targets.
- Stops sending new requests to unhealthy targets.
- Does not automatically replace failed EC2 instances.
- An Auto Scaling group can replace instances when configured.

### Example: Frontend and backend ports

- Public listener: HTTPS port 443.
- Backend application: port 8080.
- The load balancer forwards requests from 443 to 8080.
- The frontend and backend ports do not need to match.

Recommended security design:
- Allow permitted user traffic to the load balancer on port 443.
- Allow backend port 8080 on EC2 only from the load balancer's Security Group.

## 2. VPC and Subnets

### VPC
A logically isolated network in AWS.

### Public subnet
- Has a route to an Internet Gateway.
- Internet reachability also depends on IP addressing and security rules.

### Private subnet
- Has no direct route to an Internet Gateway.
- Can host backend EC2 instances.
- Can use a NAT Gateway for outbound IPv4 internet access when needed.

## 3. Internet Gateway (IGW)

- Attaches to the VPC.
- Provides a path between the VPC and the internet.
- Does not automatically make every EC2 instance publicly accessible.

Typical public subnet route:

Destination: 0.0.0.0/0
Target: Internet Gateway

## 4. NAT Gateway

- A public NAT Gateway is created in a public subnet.
- It commonly uses an Elastic IP for public IPv4 internet access.
- Allows private instances to initiate outbound internet connections.
- Example: downloading operating-system updates.
- Does not allow arbitrary new connections from the internet directly to private instances.

Typical private subnet route:

Destination: 0.0.0.0/0
Target: NAT Gateway

The NAT Gateway's public subnet also needs a route to the Internet Gateway.

Not every private subnet needs a NAT Gateway. It depends on the architecture.

## 5. Route Tables

A route table tells AWS where to send traffic based on its destination.

Example VPC CIDR: 10.0.0.0/16

| Route table | Destination | Target |
|---|---|---|
| Private subnet | 10.0.0.0/16 | local |
| Private subnet | 0.0.0.0/0 | NAT Gateway |
| Public subnet | 10.0.0.0/16 | local |
| Public subnet | 0.0.0.0/0 | Internet Gateway |

These are example routes. The VPC CIDR and resource IDs depend on your actual network.

Important:
- Keep the local route.
- Add the default route; do not replace the local route.

### Private EC2 outbound traffic

Private EC2
-> Private route table
-> NAT Gateway
-> Public subnet route table
-> Internet Gateway
-> Internet

If it fails, check both route tables, NAT Gateway state, Security Groups, NACLs, DNS and network connectivity.

## 6. Security Groups

- Apply to network interfaces associated with resources.
- Are stateful.
- Allow inbound and outbound traffic according to their rules.
- Do not support explicit deny rules.

If EC2 initiates an allowed outbound HTTPS connection:
- The request leaves EC2.
- The response returns automatically through the stateful Security Group.
- A separate inbound rule is not needed just for that response.

However, a new incoming SSH or web connection needs an appropriate inbound rule.

## 7. Network ACLs (NACLs)

- Apply at the subnet level.
- Are stateless.
- Support both ALLOW and DENY rules.
- Evaluate rules in ascending rule-number order; the first matching rule applies.

If a restrictive NACL blocks return traffic, a connection may fail even when the Security Group permits it.

For stateless NACLs, ensure both request traffic and the required return traffic are allowed.

## 8. Security Group vs NACL

| Feature | Security Group | NACL |
|---|---|---|
| Applies to | Resource/network interface | Subnet |
| Stateful? | Yes | No |
| Explicit DENY rule | No | Yes |
| Return traffic | Automatically allowed for an allowed connection | Must be allowed by the appropriate rules |

## 9. Common Questions

### Does a load balancer replace unhealthy EC2 instances?
No. It stops routing new requests to unhealthy targets. An Auto Scaling group can replace instances when configured.

### Does an Internet Gateway automatically make an EC2 instance public?
No. Routes, IP addressing and security controls must also permit access.

### Why does a private instance need a NAT Gateway?
To initiate outbound IPv4 internet connections without direct public exposure.

### Does a NAT Gateway receive incoming user requests for a private application?
No. Incoming application traffic normally reaches the load balancer, which forwards requests to the backend.

### Why can frontend and backend ports differ?
The load balancer listener and its target port are configured separately. For example, HTTPS 443 can forward to application port 8080.

## 10. Troubleshooting Checklist

If a website on EC2 cannot be reached:

1. Check whether the application is running.
2. Inspect application logs.
3. Verify the listening address and port.
4. Test the application locally.
5. Check the load balancer listener, target port and target health.
6. Check Security Groups and host firewall rules.
7. Check subnet route tables and Internet Gateway connectivity.
8. For private outbound internet access, check the private route to NAT and the NAT subnet route to IGW.
9. Check NACLs, DNS and other network controls.
10. Fix the identified cause and verify the result.

## One-Minute Revision

- Load balancer: receives and distributes application requests.
- Auto Scaling group: can replace EC2 instances when configured.
- VPC: logically isolated AWS network.
- Public subnet: has a route to an Internet Gateway.
- Private subnet: has no direct Internet Gateway route.
- IGW: provides the VPC's internet path when correctly configured.
- NAT Gateway: enables private resources to initiate outbound IPv4 internet connections.
- Route table: determines where traffic goes.
- Security Group: stateful resource-level traffic control.
- NACL: stateless subnet-level traffic filtering.
- Port 443: commonly HTTPS.
- Port 8080: an example application port.

Remember:
- Private subnet default route -> NAT Gateway.
- NAT Gateway's public subnet default route -> Internet Gateway.
- Security Groups are stateful; NACLs are stateless.
- Correct routes alone are not enough: addressing, listeners and security controls must also align.
