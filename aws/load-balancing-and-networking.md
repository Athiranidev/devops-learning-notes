# AWS Load Balancing and Networking — Learning Notes

## 1. Load Balancer

Purpose:
- Receives incoming application requests.
- Distributes requests across healthy targets.
- Uses health checks to identify unhealthy targets.
- Can accept HTTPS traffic and forward requests to backend applications.

Important:
- A load balancer does not automatically replace failed EC2 instances.
- An Auto Scaling group can replace instances when configured appropriately.
- The public listener and backend application can use different ports.

Example:
- Public listener: HTTPS port 443
- Backend application: port 8080
- Load balancer forwards requests to the configured backend port.

## 2. Security Groups

Purpose:
- Control allowed inbound and outbound network traffic for AWS resources.
- Help restrict access to only the required sources and ports.

Example architecture:
- Load balancer Security Group: allow HTTPS on port 443 from permitted users.
- EC2 Security Group: allow port 8080 from the load balancer's Security Group.

Benefit:
- Users reach the application through the load balancer.
- The backend does not need to accept direct public connections.

## 3. VPC and Subnets

VPC:
- A logically isolated network in AWS.
- Contains subnets, route tables, and network resources.

Public subnet:
- Has a route to an Internet Gateway.
- Resources also need appropriate IP addressing and security rules to be internet-accessible.

Private subnet:
- Does not have a direct route to an Internet Gateway.
- Can host backend EC2 instances that receive requests from a load balancer.

## 4. Internet Gateway (IGW)

Purpose:
- Connects a VPC to the internet when routes and other required settings permit.

Typical public subnet route:
- Destination: 0.0.0.0/0
- Target: Internet Gateway

An Internet Gateway does not automatically make every resource publicly accessible.

## 5. NAT Gateway

Purpose:
- Allows resources in a private subnet to initiate connections to the internet.
- Common example: downloading operating-system updates.

Typical private subnet route:
- Destination: 0.0.0.0/0
- Target: NAT Gateway

For public IPv4 internet access, the NAT Gateway is normally placed in a public subnet
and associated with an Elastic IP.

A NAT Gateway is not used to provide direct, unsolicited internet access to private
EC2 instances.

## 6. Route Tables

Purpose:
- Define where network traffic should be directed.

Example public subnet:
- 0.0.0.0/0 -> Internet Gateway

Example private subnet that requires outbound IPv4 internet:
- 0.0.0.0/0 -> NAT Gateway

A NAT Gateway must exist and be correctly configured for the private route to work.

## 7. Two Important Traffic Flows

Incoming application traffic:
User -> Internet Gateway -> Public Load Balancer -> Private EC2 application

Outbound internet traffic:
Private EC2 -> NAT Gateway -> Internet Gateway -> Internet

The NAT Gateway is used for outbound connectivity, not as the entry point for
incoming user requests.

## 8. Troubleshooting an Unreachable Application

If an application is installed on EC2 but cannot be opened in a browser:

1. Check whether the application is running.
2. Inspect application logs.
3. Verify the listening address and port.
4. Check the EC2 Security Group and host firewall.
5. Check the subnet route table and internet connectivity.
6. Verify the public IP or DNS destination.
7. Test again after correcting the identified cause.

A working application does not guarantee that network traffic can reach it.

## Quick Revision

- Load balancer: routes requests to healthy targets.
- Auto Scaling group: can replace failed instances when configured.
- Security Group: controls permitted network traffic.
- Public subnet: has a route to an Internet Gateway.
- Private subnet: has no direct Internet Gateway route.
- Internet Gateway: enables VPC internet connectivity with suitable routes.
- NAT Gateway: enables private resources to initiate outbound IPv4 internet traffic.
- Route table: determines where traffic is sent.
- Port 443: commonly used for HTTPS.
- Port 8080: an example backend application port.

Remember: public listener and backend ports can differ. Keep backend access restricted
when the load balancer is the intended entry point.
