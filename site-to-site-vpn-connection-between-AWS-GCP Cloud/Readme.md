# Site to Site VPN Connection Between AWS Cloud & GCP CLoud

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

