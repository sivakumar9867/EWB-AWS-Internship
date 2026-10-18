# AWS VPC Peering — Cross-VPC Private Communication

**Project:** EWB AWS Internship  
**Cloud Platform:** Amazon Web Services (AWS)  
**Region:** US East (Ohio) — `us-east-2`  
**Service Category:** Networking & Security  
**Project Type:** Hands-on Cloud Infrastructure Project

## 1. Project Overview

This project demonstrates how to establish private network connectivity between two Amazon Virtual Private Clouds (VPCs) using AWS VPC Peering.

Two VPCs were configured with separate IPv4 CIDR blocks. Each VPC contained an EC2 instance, and route tables and security group rules were configured to support communication between the private IP addresses of the instances.

The project also explored SSH connectivity, private IP communication, route configuration and connectivity testing from the EC2 command line.

## 2. Problem Statement

AWS resources deployed in separate VPCs are isolated by default. Applications running in different VPCs may need to communicate privately without relying on public IP addresses for communication between those networks.

The objective of this project was to establish a VPC peering connection, configure the required routes and security rules, and validate private communication between EC2 instances in two VPCs.

## 3. Project Objectives

- Create and configure two VPCs with distinct CIDR ranges.
- Configure public and private subnet resources where required.
- Establish a VPC peering connection between the VPCs.
- Update both VPC route tables to direct traffic to the peer network through the peering connection.
- Configure security group rules for required SSH and ICMP traffic.
- Connect to EC2 instances using SSH and test private IP connectivity.
- Validate connectivity from both sides and document the results.

## 4. Architecture and Configuration

### VPC 1 — EWB_AWS_Internship_VPC1

- **VPC CIDR:** `28.0.0.0/16`
- **EC2 instance:** `EWB_VPC1_EC2`
- **Private IP shown in evidence:** `28.0.1.194`
- **Public IP shown during testing:** `18.223.239.92`
- **Security group:** `EWB_VPC1_EC2_SG`
- **Route table:** `EWB_Public_Route_Table`

### VPC 2 — EWB_AWS_Internship_VPC2

- **VPC CIDR:** `28.200.0.0/16`
- **EC2 instance:** `EWB_VPC2_EC2`
- **Private IP shown in evidence:** `28.200.1.179`
- **Public IPv4 address:** None shown for this instance
- **Security group:** `EWB_VPC2_EC2_SG`

### VPC Peering Connection

- **Connection name:** `EWB_VPC_Peering`
- **Connection ID:** `pcx-003e95287deb1beb7`
- **Region:** `us-east-2`
- **Observed status:** Active

The AWS console evidence shows the peering connection between the two VPCs and corresponding routes to the peer CIDR blocks.

> **Architecture note:** VPC peering is a private, one-to-one network connection. It does not automatically configure routes or permit traffic; the appropriate route table entries and security rules are also required.

## 5. Implementation Steps

### Step 1 — Configure the VPCs

Created two VPCs with separate IPv4 CIDR blocks:

- VPC 1: `28.0.0.0/16`
- VPC 2: `28.200.0.0/16`

The separate address ranges allow the networks to be routed independently.

### Step 2 — Configure Subnets and EC2 Instances

Configured the network resources and EC2 instances used for testing. The evidence identifies the instances as `EWB_VPC1_EC2` and `EWB_VPC2_EC2`.

The first instance had a public IPv4 address during SSH testing. The second instance was accessed through the first instance using SSH proxying.

### Step 3 — Create the VPC Peering Connection

Created a VPC peering connection named `EWB_VPC_Peering` between the two VPCs in the same AWS region.

The console screenshot records the connection as **Active**.

### Step 4 — Configure Route Tables

Added routes so that each VPC could direct traffic to the other VPC's CIDR block through the peering connection.

| Network | Destination | Route target |
|---|---|---|
| VPC 1 | `28.200.0.0/16` | VPC peering connection |
| VPC 2 | `28.0.0.0/16` | VPC peering connection |

The screenshots show these routes with an Active status. The VPC 1 route table also contains its local route and a default route through an internet gateway.

### Step 5 — Configure Security Groups

Configured the EC2 security groups to support the traffic required for testing.

The supplied console evidence shows SSH rules on TCP port `22` and ICMP rules for connectivity testing. One security group also shows an SSH source restricted to a single IPv4 address using `/32`.

Security group sources should be restricted to the appropriate trusted IP address or network CIDR. Avoid exposing SSH to the entire internet unless there is a specific, temporary requirement.

### Step 6 — Connect Using SSH

Used Windows PowerShell and SSH key files to connect to the first EC2 instance.

A proxy connection was then used to access the second instance through the first instance. This demonstrates a bastion-style access pattern without requiring a public IPv4 address on the second instance.

**Important:** Private key files must remain local and must never be committed to a public GitHub repository.

## 6. Connectivity Testing and Results

The supplied terminal screenshots demonstrate the following:

| Test | Observed result |
|---|---|
| TCP connectivity to the first instance on port 22 | `TcpTestSucceeded: True` |
| SSH connection to the first EC2 instance | Successful |
| SSH proxy connection to the second EC2 instance | Successful |
| Ping from `28.0.1.194` to `28.200.1.179` | 4 packets received, 0% packet loss |
| Initial ping from `28.0.1.194` to `28.200.1.179` in another session | 100% packet loss |
| Later ping from `28.0.1.194` to `28.200.1.179` | 4 packets received, 0% packet loss |
| Ping from `28.200.1.179` to `28.0.1.194` | 4 packets received, 0% packet loss in the later screenshot |

The initial failed ping followed by a successful test is useful evidence that connectivity troubleshooting and verification were part of the exercise.

Successful ping tests demonstrate ICMP reachability at the time of testing. They do not, by themselves, verify every application protocol or prove that all security controls are configured correctly.

## 7. Troubleshooting Considerations

If instances cannot communicate across a VPC peering connection, check:

1. The peering connection is active.
2. Both VPCs use non-overlapping CIDR blocks.
3. Each relevant route table contains a route to the other VPC's CIDR.
4. Security groups permit the required source, protocol and port.
5. Network ACLs permit the required inbound and outbound traffic.
6. The destination instance and its operating system allow the traffic being tested.
7. SSH key permissions, usernames and proxy settings are correct.

A failed ping does not always mean the peering connection is broken. ICMP may be blocked by a security group, network ACL or operating-system firewall.

## 8. Key Learning Outcomes

Through this project, I practised:

- AWS VPC architecture and IPv4 CIDR planning.
- VPC peering connection configuration.
- Route table configuration for private network communication.
- EC2 instance access using SSH.
- Bastion-style SSH access and proxy connections.
- Security group rules for SSH and ICMP.
- Connectivity verification using PowerShell and Linux terminal commands.
- Troubleshooting network reachability and documenting cloud infrastructure.

## 9. Limitations and Future Improvements

- The setup was created as a hands-on learning environment, not a production-ready enterprise network.
- The public IP and instance state shown in screenshots reflect the environment at the time of testing and may change.
- Additional validation would be required for application-level connectivity, monitoring, high availability and production security.
- Future improvements could include private access through AWS Systems Manager Session Manager, tighter security group rules, centralized logging, infrastructure as code and automated connectivity checks.
- Network architecture and address ranges should be reviewed before using the configuration in a production environment.

## 10. Evidence and Screenshots

Add the original screenshots to a folder named `screenshots/` and use descriptive filenames. Suggested names:

1. `01-vpc-peering-connection.png` — Active `EWB_VPC_Peering` connection.
2. `02-vpc1-route-table.png` — VPC 1 route table, including the route to VPC 2.
3. `03-vpc2-route-table.png` — VPC 2 route table, including the route to VPC 1.
4. `04-vpc1-security-group.png` — VPC 1 security group rules.
5. `05-vpc2-security-group.png` — VPC 2 security group rules.
6. `06-ec2-instances.png` — EC2 instances and their private IP addresses.
7. `07-ssh-connectivity.png` — SSH connection and TCP port test.
8. `08-private-ip-ping.png` — Successful cross-VPC ping test.
9. `09-reverse-ping-test.png` — Reverse-direction connectivity test.

Use the actual supplied screenshots for these files; do not create replacement screenshots that imply a result was observed when it was not.

## 11. Repository Structure

```text
EWB-AWS-Internship/
├── README.md
└── screenshots/
    ├── 01-vpc-peering-connection.png
    ├── 02-vpc1-route-table.png
    ├── 03-vpc2-route-table.png
    ├── 04-vpc1-security-group.png
    ├── 05-vpc2-security-group.png
    ├── 06-ec2-instances.png
    ├── 07-ssh-connectivity.png
    ├── 08-private-ip-ping.png
    └── 09-reverse-ping-test.png
```

## Conclusion

This project provided practical experience in connecting separate AWS VPCs through VPC peering. By configuring routes, security group rules and SSH access, I demonstrated private IP communication between EC2 instances and practised troubleshooting connectivity issues.

The project strengthened my understanding of AWS networking fundamentals, routing, access control and cloud infrastructure validation.

## 12. Evidence Screenshot Index

The following evidence images are stored in `03_Evidence_Screenshots/`. Screenshots are copied from the matching evidence gallery images in the portfolio website; image bytes are preserved without re-encoding.

1. `01_VPC_Peering_Active.png` — Active VPC peering connection.
2. `02_VPC1_Route_Table.png` — VPC 1 route table.
3. `03_VPC2_Route_Table.png` — VPC 2 route table.
4. `04_Third_Route_Table.png` — Additional/third route table.
5. `05_VPC1_Security_Group.png` — VPC 1 security-group rules.
6. `06_VPC2_Security_Group.png` — VPC 2 security-group rules.
7. `07_VPC2_Security_Group_SSH_Rule.png` — SSH rule for VPC 2.
8. `08_EC2_Resource_Details_VPC1.png` — EC2 resource details for VPC 1.
9. `09_EC2_Resource_Details_VPC2.png` — EC2 resource details for VPC 2.
10. `10_Forward_Ping_Success.png` — Successful forward-direction ICMP test.
11. `11_Reverse_Ping_Success.png` — Successful reverse-direction ICMP test.
12. `12_Private_SSH_App_B.png` — Private SSH access evidence for App B.
13. `13_Controlled_ICMP_Failure.png` — Controlled ICMP failure during troubleshooting.
14. `14_Available_Before_Cleanup.png` — EC2 instances available before cleanup.
15. `15_Cleanup_Evidence.png` — Manually redacted cleanup/termination evidence, copied without further image editing.
16. `16_ICMP_Recovery_Success.png` — Successful ICMP test after troubleshooting.
17. `17_Architecture_and_Resource_Inventory.png` — Architecture and resource inventory evidence.
