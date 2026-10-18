# AWS-Secure-Three-Tier-Architecture
Designed and deployed a secure AWS three-tier architecture using Amazon VPC, EC2, PostgreSQL, public and private subnets, route tables, security groups, and network ACLs. Focused on network segmentation, least-privilege access, secure application connectivity, and cloud infrastructure security.


## Step 1 – Verify Existing AWS VPC Infrastructure

![Verify Existing VPC Infrastructure](https://github.com/Kevinolee1/AWS-Secure-Three-Tier-Architecture/blob/5e4812171242df0caa0e709e661a024698f1a1a3/Screenshot%202026-10-18%20130659.png)

**Figure 1 – Existing AWS VPC Infrastructure:** I verified the `CloudSecurity-VPC` environment in the AWS Management Console before beginning the three-tier architecture deployment.

The VPC uses the `10.0.0.0/16` CIDR block and contains separate public and private subnets, dedicated route tables, and an attached internet gateway.

The AWS VPC Resource Map confirmed the network relationships between these components.

This existing infrastructure provides the foundation for implementing a secure three-tier architecture with separate web, application, and database layers.

## Step 2 – Create Private Application Subnet

![Create Private Application Subnet](images/02-create-private-application-subnet.png)

**Figure 2 – Private Application Subnet Creation:** I created a new subnet named `CloudSecurity-App-Subnet` within the existing `CloudSecurity-VPC` using the IPv4 CIDR block `10.0.3.0/24`.

The AWS Management Console confirmed that the subnet was successfully created and entered the `Available` state.

This subnet will host the application tier of the three-tier architecture, separating application workloads from the public-facing web tier and the private database tier.

The next configuration step establishes private routing for the application subnet to prevent direct internet gateway access.

## Step 3 – Associate Application Subnet with Private Route Table

![Private Route Table Associations](images/03-associate-application-private-route-table.png)

**Figure 3 – Private Subnet Route Table Associations:** I configured the `CloudSecurity-Private-RT` route table to serve both the application and database subnets within `CloudSecurity-VPC`.

The AWS Management Console confirmed two explicit subnet associations:

- `CloudSecurity-Private-Subnet` – `10.0.2.0/24`
- `CloudSecurity-App-Subnet` – `10.0.3.0/24`

This configuration allows both private subnets to use a shared routing policy while keeping their network segments logically separated.

The application and database tiers will use dedicated security groups to enforce access restrictions between workloads.

## Step 4 – Create Application Tier Security Group

![Application Tier Security Group](images/04-create-application-security-group.png)

**Figure 4 – Application Security Group Configuration:** I created `CloudSecurity-App-SG` within `CloudSecurity-VPC` to establish network access controls for the application tier.

The security group was created with one inbound rule and zero outbound rules, removing the default unrestricted outbound access.

This configuration establishes an initial least-privilege security boundary. The inbound rule will be verified to ensure that only the authorized web tier can initiate connections to the application service on TCP port 8080.

Additional outbound permissions will be configured when establishing secure application-to-database communication.

## Step 5 – Verify Application Tier Security Group

![Application Security Group Verification](images/05-verify-application-security-group.png)

**Figure 5 – Application Tier Security Verification:** I verified that `CloudSecurity-App-SG` permits inbound TCP traffic on port `8080` from the authorized `CloudSecurity-Web-SG`.

This configuration restricts application access to resources associated with the web-tier security group instead of allowing unrestricted public access.

AWS also confirmed that the application's outbound security group rules were successfully updated. The outbound destination and database permissions require separate verification.
