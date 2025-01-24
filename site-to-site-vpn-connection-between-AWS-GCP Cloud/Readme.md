# Site to Site VPN Connection Between AWS Cloud & GCP CLoud

![image](https://github.com/user-attachments/assets/2a0c5cc3-69e0-48ee-847b-8ca783e671bc)


# Objectives
- Create a VPC network on Google & AWS Cloud.

- Create an HA VPN gateway and Cloud Router on Google Cloud.

- Create customer gateways on AWS.

- Create a VPN connection with dynamic routing on AWS.

- Create an external VPN gateway and VPN tunnels on Google Cloud.

- Verify and test the VPN connection between VPC networks on Google Cloud and AWS.

# Step 1 — Create a VPC network on Google & AWS Cloud.

**VPC on Google Cloud:**

```
gcloud compute networks create gcp-vpc \
   --subnet-mode=custom \
   --bgp-routing-mode=global


gcloud compute networks subnets create gcp-vpc-sub1-us-central1 \
  --network gcp-vpc  \
  --range 172.21.1.0/24 \
  --region us-central1 \
  --enable-flow-logs \
  --enable-private-ip-google-access

gcloud compute networks subnets create gcp-vpc-sub2-euro-west2 \
  --network gcp-vpc  \
  --range 172.21.2.0/24 \
  --region europe-west2 \
  --enable-flow-logs \
  --enable-private-ip-google-access


gcloud compute networks list
gcloud compute networks describe gcp-vpc
gcloud compute networks subnets list --filter gcp-vpc


gcloud compute firewall-rules create gcp-vpc-ssh-allow \
    --network gcp-vpc \
    --action allow \
    --direction ingress \
    --rules tcp:22,icmp \
    --source-ranges 39.51.35.31/32 \
    --priority 1000 \
    --enable-logging \
    --target-tags gcp-vpc-ssh-allow


gcloud compute firewall-rules create gcp-vpc-internal-allow \
    --network  gcp-vpc \
    --action allow \
    --direction ingress \
    --rules tcp,udp,icmp,ipip \
    --source-ranges 172.21.0.0/16 \
    --priority 1100 

```

**Create Two VMS (Public & Private VMs)**

```

gcloud compute instances create public-us-vm \
  --image-family ubuntu-2204-lts \
  --image-project ubuntu-os-cloud \
  --boot-disk-size 20GB \
  --subnet gcp-vpc-sub1-us-central1 \
  --private-network-ip 172.21.1.11 \
  --zone us-central1-b \
  --project optimum-beach-440914-n9 \
  --tags gcp-vpc-ssh-allow



gcloud compute instances create private-euro-vm \
   --image-family centos-stream-9 \
   --image-project centos-cloud \
   --machine-type e2-medium \
   --boot-disk-size 20GB \
   --subnet gcp-vpc-sub2-euro-west2 \
   --private-network-ip 172.21.2.20 \
   --zone europe-west2-a \
   --project optimum-beach-440914-n9 \
   --no-address


```


# VPC on AWS Cloud

# 1. Create a VPC
```
aws ec2 create-vpc --cidr-block 192.168.0.0/16 --output table

```
- **What it does:** Creates a Virtual Private Cloud (VPC) with a CIDR block of ```192.168.0.0/16```.

**Options:**
- ```--cidr-block```: Specifies the range of IP addresses for the VPC.
- ```--output table```: Displays the output in a table format.

# 2. Modify VPC Attributes
```
aws ec2 modify-vpc-attribute --vpc-id vpc-0adfd6b9e963d85e7 --enable-dns-support "{\"Value\":true}"

```
- **What it does**: Enables DNS resolution in the VPC.

**Options:**
- ```--enable-dns-support "{\"Value\":true}"```: Ensures instances within the VPC can resolve domain names to IP addresses.

```
aws ec2 modify-vpc-attribute --vpc-id vpc-0adfd6b9e963d85e7 --enable-dns-hostnames "{\"Value\":true}"

```

- **What it does**: Enables DNS hostnames for instances in the VPC.

# 3. Add Tags to a VPC
```
aws ec2 create-tags --resources vpc-0adfd6b9e963d85e7 --tags Key=Name,Value=AWS-VPC

```
- **What it does**: Adds a tag (```Name=AWS-VPC```) to the VPC for easier identification.

# 4. Create Subnets
```
aws ec2 create-subnet --vpc-id vpc-0adfd6b9e963d85e7 --cidr-block 192.168.1.0/24 --availability-zone us-east-1a

```
- **What it does**: Creates a subnet in the specified VPC with a CIDR block of ```192.168.1.0/24 in Availability Zone us-east-1a```.

```
aws ec2 create-subnet --vpc-id vpc-0adfd6b9e963d85e7 --cidr-block 192.168.2.0/24 --availability-zone us-east-1b
```
- Creates another subnet in ```us-east-1b```.

# 5. Add Tags to Subnets
```
aws ec2 create-tags --resources subnet-07c82f715b726e54f --tags Key=Name,Value=AWS-VPC-PubSub1
```

- ***What it does**: Tags the subnet ```subnet-07c82f715b726e54f``` as ```AWS-VPC-PubSub1```.

```
aws ec2 create-tags --resources subnet-0cac30b89fc5e3ec5 --tags Key=Name,Value=AWS-VPC-PvtSub1
```

- Tags the subnet ```subnet-0cac30b89fc5e3ec5``` as ```AWS-VPC-PvtSub1```.

# 6. Enable Public IP Assignment for Subnet (provide public-subnet-id)
```
aws ec2 modify-subnet-attribute --subnet-id subnet-07c82f715b726e54f --map-public-ip-on-launch
```
- **What it does**: Configures the subnet to assign a public IP address to instances at launch. This is typically done for public subnets.

# 7. Create an Internet Gateway
```
aws ec2 create-internet-gateway --output json
```

- **What it does**: Creates an **Internet Gateway (IGW)** for the VPC to allow communication with the internet.

**Options**:
- ```--output json```: Displays the output in JSON format.

# 8. Add Tags to the Internet Gateway
```
aws ec2 create-tags --resources igw-09a332ea3a9ec7f0f --tags Key=Name,Value=AWS-IGW
```

- **What it does**: Tags the Internet Gateway as ```AWS-IGW```.

# 9. Attach the Internet Gateway to a VPC
```
aws ec2 attach-internet-gateway --vpc-id vpc-0adfd6b9e963d85e7 --internet-gateway-id igw-09a332ea3a9ec7f0f --region us-east-1
```

- **What it does**: Attaches the IGW to the VPC, allowing internet traffic to route through the VPC.

# 10. List Internet Gateways
```
aws ec2 describe-internet-gateways --output table
```

- **What it does**: Displays all Internet Gateways in the account in a table format.

# 11. Create Route Tables
```
aws ec2 create-route-table --vpc-id vpc-0adfd6b9e963d85e7
```

- **What it does**: Creates a route table associated with the VPC for managing routing rules.

# 12. Add Tags to Route Tables
```
aws ec2 create-tags --resources rtb-06db711fbb21b8891 --tags Key=Name,Value=PublicRT
```

- **What it does**: Tags the route table ```rtb-06db711fbb21b8891``` as ```PublicRT``` (used for public subnet routing).

```
aws ec2 create-tags --resources rtb-041d2652c6f984e15 --tags Key=Name,Value=PrivateRT
```

- Tags another route table as ```PrivateRT```.

# 13. Create Routes (provide public-RT-id)
```
aws ec2 create-route --route-table-id rtb-06db711fbb21b8891 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-09a332ea3a9ec7f0f

```

- **What it does**: Adds a route to the ```PublicRT``` route table to direct traffic with the destination ```0.0.0.0/0``` (all traffic) to the Internet Gateway.

# 14. Describe Route Tables
```
aws ec2 describe-route-tables --route-table-id rtb-06db711fbb21b8891
```

- **What it does**: Displays details about the ```publicRT``` route table.

# 15. Associate Route Tables with Subnets (provide public-subnet-id and public-RT-id)
```
aws ec2 associate-route-table --subnet-id subnet-07c82f715b726e54f --route-table-id rtb-06db711fbb21b8891
```

- **What it does**: Associates the ```PublicRT``` route table with the ```AWS-VPC-PubSub1``` (public subnet).

```
aws ec2 associate-route-table --subnet-id subnet-0cac30b89fc5e3ec5 --route-table-id rtb-041d2652c6f984e15
```

- Associates the ```PrivateRT``` route table with the ```AWS-VPC-PvtSub1``` (private subnet).

# 16. Create a VPC in GCP
```
gcloud compute networks create nw1-vpc --subnet-mode custom
```

- **What it does**: Creates a custom VPC in Google Cloud Platform (GCP) named ```nw1-vpc```.

**Options**:
- ```--subnet-mode custom```: Specifies that custom subnets will be created instead of automatically creating subnets in all regions.
  

```
Create VMs on 2 subnets
```



