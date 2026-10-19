# AWS-Secure-Three-Tier-Architecture
Designed and deployed a secure AWS three-tier architecture using Amazon VPC, EC2, PostgreSQL, public and private subnets, route tables, security groups, and network ACLs. Focused on network segmentation, least-privilege access, secure application connectivity, and cloud infrastructure security.


## Step 1 – Verify Existing AWS VPC Infrastructure

![Verify Existing VPC Infrastructure](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/5e4812171242df0caa0e709e661a024698f1a1a3/Screenshot%202026-10-18%20130659.png)

**Figure 1 – Existing AWS VPC Infrastructure:** I verified the `CloudSecurity-VPC` environment in the AWS Management Console before beginning the three-tier architecture deployment.

The VPC uses the `10.0.0.0/16` CIDR block and contains separate public and private subnets, dedicated route tables, and an attached internet gateway.

The AWS VPC Resource Map confirmed the network relationships between these components.

This existing infrastructure provides the foundation for implementing a secure three-tier architecture with separate web, application, and database layers.

## Step 2 – Create Private Application Subnet

![Create Private Application Subnet](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/118aa4adf9c9f43fde3a8cae680d6acee79dd02c/Screenshot%202026-10-18%20131017.png)

**Figure 2 – Private Application Subnet Creation:** I created a new subnet named `CloudSecurity-App-Subnet` within the existing `CloudSecurity-VPC` using the IPv4 CIDR block `10.0.3.0/24`.

The AWS Management Console confirmed that the subnet was successfully created and entered the `Available` state.

This subnet will host the application tier of the three-tier architecture, separating application workloads from the public-facing web tier and the private database tier.

The next configuration step establishes private routing for the application subnet to prevent direct internet gateway access.

## Step 3 – Associate Application Subnet with Private Route Table

![Private Route Table Associations](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/bea17ae4de9f1522bc68e3545b320730d695d68a/Screenshot%202026-10-18%20131424.png)

**Figure 3 – Private Subnet Route Table Associations:** I configured the `CloudSecurity-Private-RT` route table to serve both the application and database subnets within `CloudSecurity-VPC`.

The AWS Management Console confirmed two explicit subnet associations:

- `CloudSecurity-Private-Subnet` – `10.0.2.0/24`
- `CloudSecurity-App-Subnet` – `10.0.3.0/24`

This configuration allows both private subnets to use a shared routing policy while keeping their network segments logically separated.

The application and database tiers will use dedicated security groups to enforce access restrictions between workloads.

## Step 4 – Create Application Tier Security Group

![Application Tier Security Group](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/eb31905b4ab1816dfd59f85405a7852ffbbea997/Screenshot%202026-10-18%20132604.png)

**Figure 4 – Application Security Group Configuration:** I created `CloudSecurity-App-SG` within `CloudSecurity-VPC` to establish network access controls for the application tier.

The security group was created with one inbound rule and zero outbound rules, removing the default unrestricted outbound access.

This configuration establishes an initial least-privilege security boundary. The inbound rule will be verified to ensure that only the authorized web tier can initiate connections to the application service on TCP port 8080.

Additional outbound permissions will be configured when establishing secure application-to-database communication.

## Step 5 – Verify Application Tier Security Group

![Application Security Group Verification](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/35e5a97fcbb4ed1f57aac83ad74e73676b52866e/Screenshot%202026-10-18%20134332.png)

**Figure 5 – Application Tier Security Verification:** I verified that `CloudSecurity-App-SG` permits inbound TCP traffic on port `8080` from the authorized `CloudSecurity-Web-SG`.

This configuration restricts application access to resources associated with the web-tier security group instead of allowing unrestricted public access.

AWS also confirmed that the application's outbound security group rules were successfully updated. The outbound destination and database permissions require separate verification.


## Step 6 – Verify Database Security Group Access

![Database Security Group Verification](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/3311eb78734b6091a3c7bb206f87e234fc7c2d18/Screenshot%202026-10-18%20142416.png)

**Figure 6 – Database Security Group Access Verification:** Verified that `CloudSecurity-DB-SG` allows inbound PostgreSQL traffic on TCP port `5432` exclusively from `CloudSecurity-App-SG`.

This configuration establishes security-group-based access control between the application and database tiers, restricting direct database access from the web tier and external networks.

The security group configuration was verified successfully. End-to-end connectivity testing remains a separate validation step.

## Step 7 – Verify Default Network ACL Inbound and Outbound Rules

![Default Network ACL Inbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/eda5b945a18604add842e71357f3eaf761540e94/Screenshot%202026-10-18%20143514.png)

![Default Network ACL Outbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/c9a4db12d8ba1fc63a9016e43adabd101042be13/Screenshot%202026-10-18%20144925.png)

**Figure 7 – Default Network ACL Security Verification:** Reviewed the inbound and outbound rules of the default Network ACL associated with `CloudSecurity-Public-Subnet` and `CloudSecurity-App-Subnet`.

Verified that rule `100` permits all inbound and outbound IPv4 traffic (`0.0.0.0/0`), followed by the default deny rule.

The assessment confirmed that both subnets share a permissive Network ACL without subnet-specific traffic restrictions. Although security groups provide resource-level access controls, the default ACL does not independently enforce network segmentation.

To strengthen the three-tier architecture, I proceeded with implementing a dedicated application Network ACL to restrict unauthorized traffic, enforce controlled communication between network tiers, and support defense-in-depth security.

## Step 8 – Create Application Network ACL

![Create Application Network ACL](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/21502fe5265ca9488d84c16ebc77d6f698697bba/Screenshot%202026-10-18%20144430.png)

**Figure 8 – Application Network ACL Creation:** Created a dedicated network access control list named `CloudSecurity-App-NACL` to provide subnet-level traffic filtering for the application tier.

The custom Network ACL will be configured to permit required application traffic and return traffic while denying unauthorized network connections.

The ACL will be associated with the application subnet after its inbound and outbound rules are configured.

## Step 9 – Configure Application Network ACL Inbound Rules

![Application Network ACL Inbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/65afb7f39197a958f4b3fbb30041b003f48ccbde/Screenshot%202026-10-18%20145602.png)

**Figure 9 – Application Network ACL Inbound Configuration:** Configured the `CloudSecurity-App-NACL` with an inbound rule permitting TCP traffic on port `8080` from the public web subnet (`10.0.1.0/24`).

Rule `100` allows the web tier to send application requests to the application subnet, while the default deny rule blocks unmatched inbound traffic.

This configuration supports network segmentation and defense-in-depth security within the three-tier AWS architecture.

## Step 10A – Configure Application Network ACL Return Traffic

![Application Network ACL Return Traffic](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/57be298a761cc7959dfe3c3f278e1e70817cac8e/Screenshot%202026-10-18%20185553.png)

**Figure 10A – Application Network ACL Return Traffic Configuration:** Added inbound rule `110` to `CloudSecurity-App-NACL` to permit TCP traffic on ephemeral ports `1024–65535` from the database subnet (`10.0.2.0/24`).

This rule supports return traffic from the database tier to the application tier because AWS Network ACLs are stateless and require explicit rules for both directions of communication.

The existing rule `100` continues to permit web-to-application traffic on TCP port `8080`, while the default deny rule blocks unmatched inbound traffic.

## Step 10B – Configure Application Network ACL Return Traffic

![Application Network ACL Return Traffic](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/8c6d85843a89dab7297f698cd322f5837ad86e56/Screenshot%202026-10-18%20185615.png)

**Figure 10B – Application Network ACL Inbound and Return Traffic:** Configured `CloudSecurity-App-NACL` with rule `100` allowing TCP port `8080` from the web subnet (`10.0.1.0/24`) and rule `110` allowing ephemeral TCP ports `1024–65535` from the database subnet (`10.0.2.0/24`).

These rules support communication between the web, application, and database tiers while accounting for the stateless behavior of AWS Network ACLs. The default deny rule blocks unmatched inbound traffic.

## Step 11 – Configure Application Network ACL Outbound Rules

![Application Network ACL Outbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/623f305cf8830cf0deef0bca8a284bf926dcebd6/Screenshot%202026-10-18%20191540.png)

**Figure 11 – Application Network ACL Outbound Configuration:** Configured `CloudSecurity-App-NACL` with outbound rule `100` allowing TCP ephemeral ports `1024–65535` to the web subnet (`10.0.1.0/24`) and rule `110` allowing PostgreSQL traffic on TCP port `5432` to the database subnet (`10.0.2.0/24`).

These rules support application-to-database communication and return traffic to the web tier while enforcing subnet-level network segmentation.

Verified that unmatched outbound traffic is denied by the default Network ACL rule.

## Step 12 – Associate Application Network ACL with Application Subnet

![Application Network ACL Subnet Association](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/1d5249863e5c63159d612db5d170137ca141851a/Screenshot%202026-10-18%20192842.png)

**Figure 12 – Application Network ACL Subnet Association:** Successfully associated the custom `CloudSecurity-App-NACL` with `CloudSecurity-App-Subnet` (`10.0.3.0/24`) within `CloudSecurity-VPC`.

This association replaces the subnet's previous default Network ACL and applies the custom inbound and outbound filtering rules configured in earlier steps.

The configuration establishes dedicated subnet-level access controls for the application tier, supporting network segmentation and defense-in-depth security across the three-tier AWS architecture.

Verified that the AWS console displays the correct subnet association.

## Step 13 – Configure Database Network ACL Inbound Rules

![Database Network ACL Inbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/30616f1974946e94bc6d4b35231593201841e750/Screenshot%202026-10-18%20193801.png)

**Figure 13 – Database Network ACL Inbound Configuration:** Updated `CloudSecurity-Private-NACL` to permit PostgreSQL traffic on TCP port `5432` from the application subnet (`10.0.3.0/24`).

Rule `100` restricts database-bound traffic to the application subnet, while the default deny rule blocks unmatched inbound connections.

This configuration strengthens network segmentation by defining a controlled application-to-database communication path within the three-tier AWS architecture.

Verified that the updated inbound rule was saved successfully in the AWS VPC console.

## Step 14 – Configure Database Network ACL Outbound Return Traffic

![Database Network ACL Outbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/c0583e44a73fe81b7f76430cc0eb0100bd741bae/Screenshot%202026-10-18%20194142.png)

**Figure 14 – Database Network ACL Outbound Configuration:** Updated `CloudSecurity-Private-NACL` to allow outbound TCP traffic on ephemeral ports `1024–65535` to the application subnet (`10.0.3.0/24`).

Rule `100` supports PostgreSQL response traffic returning from the database tier to the application tier. The default deny rule blocks unmatched outbound connections.

Together with the inbound PostgreSQL rule configured in Step 13, this establishes the intended subnet-level filtering configuration for application-to-database communication.

Verified that AWS successfully saved the updated outbound rule.

## Step 15 – Configure Web Security Group Outbound Rules

![Web Security Group Outbound Rules](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/c1cf7b5075a41192488cc7abce33ca975dfe7f07/Screenshot%202026-10-18%20195456.png)

**Figure 15 – Web Security Group Outbound Configuration:** Updated `CloudSecurity-Web-SG` to allow outbound TCP traffic on port `8080` exclusively to `CloudSecurity-App-SG`.

Removed the default unrestricted outbound rule permitting all traffic to `0.0.0.0/0`.

This configuration applies the principle of least privilege by restricting outbound network access to the application tier, supporting controlled communication and defense-in-depth security within the three-tier AWS architecture.

Verified that the AWS console displays only the intended outbound security group rule.


