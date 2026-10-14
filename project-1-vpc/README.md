# Project 1: Custom VPC with Bastion Host and NAT Gateway

## Goal
Build a custom AWS VPC by hand with public and private subnets, a bastion host for SSH access, and a NAT Gateway for outbound-only internet access.

## Architecture
![VPC diagram](images/vpc-diagram.png)

## CIDR plan
| Subnet | AZ | CIDR | Usable IPs |
|---|---|---|---|
| public-1 | us-east-1a | 10.0.1.0/24 | 251 |
| public-2 | us-east-1b | 10.0.2.0/24 | 251 |
| private-1 | us-east-1a | 10.0.3.0/24 | 251 |
| private-2 | us-east-1b | 10.0.4.0/24 | 251 |

VPC CIDR: `10.0.0.0/16`

## Components
- **Internet Gateway** attached to `my-vpc`
- **public-rt**: `0.0.0.0/0 → IGW`, associated with the public subnets
- **private-rt**: local route only, plus `0.0.0.0/0 → NAT` during testing
- **bastion-sg**: SSH from my IP only
- **private-sg**: SSH only from `bastion-sg`

## Results
**Public route table**
![public-rt](images/public-rt.png)

**SSH hop: laptop → bastion → private server (agent forwarding)**
![ssh hop](images/ssh-hop.png)

**Before NAT: private server has no internet**
```
[ec2-user@ip-10-0-3-166 ~]$ ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2057ms
```

**After NAT: outbound traffic works**
```
[ec2-user@ip-10-0-3-166 ~]$ ping -c 3 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=2.49 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=1.37 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=1.08 ms

3 packets transmitted, 3 received, 0% packet loss

[ec2-user@ip-10-0-3-166 ~]$ curl -I https://aws.amazon.com
HTTP/2 200
```

## What I learned
- A subnet is public only if its route table has a route to an Internet Gateway.
- A NAT Gateway must sit in a public subnet, because it needs the IGW route to reach the internet.
- NAT is outbound-only: the internet cannot start connections to the private server.
- Security groups can reference other security groups, which is safer than using IP ranges.
- SSH agent forwarding lets me hop through a bastion without copying my private key onto it.
- [Add one thing that surprised you or that you got wrong at first.]

## Cleanup
NAT Gateway deleted and Elastic IP released after testing to avoid charges.
