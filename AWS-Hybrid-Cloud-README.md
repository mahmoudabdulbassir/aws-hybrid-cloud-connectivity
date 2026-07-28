# AWS Hybrid Cloud Connectivity with Transit Gateway and Site-to-Site VPN

## Overview

This project demonstrates how to design and implement a secure hybrid cloud architecture that connects an on-premises data center to Amazon Web Services (AWS). The solution uses AWS Transit Gateway as the central networking hub, AWS Site-to-Site VPN with dynamic routing (BGP), and OPNsense as the on-premises Customer Gateway.

The architecture also includes Amazon Route 53 Resolver to provide hybrid DNS resolution, allowing seamless communication between on-premises resources and AWS workloads.

---

## Architecture
                                        AWS Hybrid Cloud Connectivity Architecture

                                                    ┌──────────────────────────────┐
                                                    │           Internet           │
                                                    └──────────────┬───────────────┘
                                                                   │
                                                         Public IP: 203.0.113.10
                                                                   │
                                            ┌──────────────────────▼──────────────────────┐
                                            │            OPNsense Firewall               │
                                            │--------------------------------------------│
                                            │ WAN: Public IP                             │
                                            │ LAN: 192.168.1.4                  │
                                            │ FRR (BGP ASN: 65000)                       │
                                            │ IPSec VPN (Tunnel 1 & Tunnel 2)            │
                                            └──────────────┬─────────────────────────────┘
                                                           │
                                    On-Premises LAN        │
                                   192.168.10.0  │
                                                           │
                         ┌───────────────────────┐         │
                         │   NGINX Web Server    │         │
                         │     192.168.10.5     │
                         └───────────────────────┘         │
                                                           │
                 ======================================================================
                               AWS Site-to-Site VPN (IPSec + BGP)
                 ======================================================================
                                                           │
                           ┌───────────────────────────────┴──────────────────────────────┐
                           │                                                              │
             Tunnel 1                                                     Tunnel 2
     Outside: 203.0.113.20                                      Outside: 203.0.113.30
     Inside AWS: 169.254.60.121                                 Inside AWS: 169.254.126.125
     Inside OPNsense: 169.254.60.122                            Inside OPNsense: 169.254.126.126
                           │                                                              │
                           └───────────────────────────────┬──────────────────────────────┘
                                                           │
                                     BGP Peering (ASN 65000 ↔ ASN 64512)
                                                           │
                                      ┌────────────────────▼─────────────────────┐
                                      │        AWS Transit Gateway               │
                                      │               ASN 64512                  │
                                      └───────┬────────────┬────────────┬────────┘
                                              │            │            │
                              TGW Attachment   │            │            │ TGW Attachment
                                              │            │            │
                     ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
                     │    Dev VPC     │ │  Staging VPC   │ │   Prod VPC     │
                     │172.16.0.0/16   │ │172.17.0.0/16   │ │172.18.0.0/16   │
                     └──────┬─────────┘ └──────┬─────────┘ └──────┬─────────┘
                            │                  │                  │
                 ┌──────────▼─────────┐ ┌──────▼─────────┐ ┌──────▼─────────┐
                 │ Public Subnet      │ │ Public Subnet  │ │ Public Subnet  │
                 │ Private Subnet     │ │ Private Subnet │ │ Private Subnet │
                 │ EC2 Instances      │ │ EC2 Instances  │ │ EC2 Instances  │
                 └────────────────────┘ └────────────────┘ └────────────────┘
                                              │
                                              │
                    ┌─────────────────────────▼─────────────────────────┐
                    │            Amazon Route 53 Resolver               │
                    │---------------------------------------------------│
                    │ • Inbound Resolver Endpoint                       │
                    │ • Outbound Resolver Endpoint                      │
                    │ • Hybrid DNS Resolution                           │
                    └─────────────────────────┬─────────────────────────┘
                                              │
                           ┌──────────────────▼──────────────────┐
                           │      AWS Network Firewall           │
                           │ Centralized Traffic Inspection      │
                           └──────────────┬──────────────────────┘
                                              │
                           ┌──────────────────▼──────────────────┐
                           │  AWS CloudTrail & AWS Config        │
                           │  Auditing & Compliance Monitoring   │
                           └─────────────────────────────────────┘
---

## Project Objectives

- Design a scalable hybrid cloud network.
- Connect an on-premises network to AWS using Site-to-Site VPN.
- Configure AWS Transit Gateway as the central routing hub.
- Implement dynamic routing using Border Gateway Protocol (BGP).
- Connect multiple AWS VPCs through a single Transit Gateway.
- Configure hybrid DNS using Amazon Route 53 Resolver.
- Follow AWS networking best practices for security and scalability.

---

## Technologies Used

### AWS Services

- Amazon VPC
- AWS Transit Gateway
- AWS Site-to-Site VPN
- AWS Customer Gateway
- Amazon Route 53 Resolver
- AWS Resource Access Manager (RAM)
- AWS Config
- Amazon EC2

### On-Premises

- OPNsense Firewall
- FRRouting (FRR)
- IPsec
- BGP

---

## Network Design

### On-Premises Network

| Component | Value |
|-----------|-------|
| WAN Network | 192.168.20.0/24 |
| OPNsense WAN | 192.168.20.4 |
| LAN Network | 192.168.10.0/24 |
| OPNsense LAN | 192.168.10.4 |
| Web Server | 192.168.10.5 |
| Customer Gateway ASN | 65000 |

### AWS Networks

| Environment | CIDR |
|------------|------|
| Development | 172.16.10.0/16 |
| Staging | 172.16.20.0/16 |
| Production | 172.16.30.0/16 |

Transit Gateway ASN: **64512**

---

## Solution Components

### AWS Transit Gateway

Acts as the central routing hub connecting:

- Development VPC
- Staging VPC
- Production VPC
- Site-to-Site VPN

---

### Site-to-Site VPN

Two redundant IPSec tunnels provide secure communication between AWS and the on-premises environment.

Features:

- Dual VPN tunnels
- High Availability
- Dynamic routing (BGP)
- Automatic failover

---

### Border Gateway Protocol (BGP)

Dynamic routing is implemented using FRRouting (FRR).

| Parameter | Value |
|-----------|-------|
| Customer Gateway ASN | 65000 |
| AWS ASN | 64512 |

Benefits:

- Automatic route propagation
- Simplified routing
- High availability
- Automatic failover

---

### Hybrid DNS

Amazon Route 53 Resolver enables:

- On-premises DNS servers to resolve AWS private DNS records.
- AWS resources to resolve on-premises DNS records.
- Centralized DNS management.

---

## Deployment Summary

The deployment consists of the following high-level steps:

1. Create AWS VPCs.
2. Configure public and private subnets.
3. Create the AWS Transit Gateway.
4. Attach all VPCs to the Transit Gateway.
5. Configure the Customer Gateway.
6. Create the AWS Site-to-Site VPN.
7. Configure OPNsense IPsec tunnels.
8. Configure FRRouting (FRR) and BGP.
9. Configure Route 53 Resolver.
10. Validate connectivity and routing.

A detailed deployment guide is available in:

**docs/implementation-guide.md**

---

## Validation

The following validation tests were performed:

- Transit Gateway attachments
- VPN tunnel establishment
- BGP neighbor establishment
- Route propagation
- EC2 connectivity between VPCs
- On-premises to AWS communication
- Hybrid DNS resolution

---

## Security Best Practices

- Non-overlapping CIDR ranges
- Dynamic routing with BGP
- Redundant VPN tunnels
- Private subnets for backend resources
- Security Groups and Network ACLs
- CloudTrail auditing
- AWS Config compliance monitoring
- AWS Network Firewall support
- Hybrid DNS with Route 53 Resolver

---

## Future Enhancements

- Replace VPN with AWS Direct Connect
- Infrastructure as Code using Terraform
- High Availability Customer Gateway
- Centralized monitoring with Amazon CloudWatch
- Transit Gateway sharing using AWS RAM
- Enhanced traffic inspection using AWS Network Firewall

---

## References

- AWS Transit Gateway
- AWS Site-to-Site VPN
- Amazon Route 53 Resolver
- AWS Network Firewall
- OPNsense
- FRRouting (FRR)

---

## Author

**Mahmoud Abdulbasir**

System Engineer | AWS Cloud Practitioner

---

## License

This project is provided for educational and portfolio purposes.
