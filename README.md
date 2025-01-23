# Steps to create a GCP VM with 2 Network Interfaces:

# Step1: Set Up VPC Networks:
**1. Create the First VPC Network:**
- Navigate to **VPC Network** > **VPC Networks** in the GCP Console.
- Click Create **VPC Network**.
- Provide a name for the network (e.g., **vpc-1**).
- Under Subnet creation mode, choose **Custom**.
- Add a subnet (e.g., **subnet-1**), **Region**: ```us-central1``` with a specified IP range (e.g., **10.0.0.0/24**).
- Leave the rest of the settings as default and click Create.


**2. Create the Second VPC Network:**
- Repeat the same steps to create another VPC network (e.g., **vpc-2**).
- Add a subnet (e.g., **subnet-2**), **Region**: ```us-central1``` with a different IP range (e.g., **10.1.0.0/24**).

# Step 2: Create Firewall Rules
- Add firewall rules to allow SSH, HTTP, or other traffic you need for your blog.
- For each VPC network:
1. Go to **VPC Network** > **Firewall Rules**.
2. Click Create **Firewall Rule**.
3. Configure the rule, e.g.:
- Name: ```allow-http```

- Network: ```vpc-1``` (repeat for ```vpc-2```).

- Targets: ```All instances in the network```.

- Protocols and Ports: Select **Specified protocols and ports** and add ```tcp:22, tcp:80```
4. Save the rule.

# Step 3: Create the VM Instance
**1. Navigate to VM Instances:**

- Go to **Compute Engine** > **VM Instances**.
- Click **Create Instance**.

**2. Configure the VM Instance:**

- **Name**: Enter a name for the VM (e.g., ```router-vm```).
- **Region and Zone**: Select the region and zone where your VPC networks were created.
- **Machine Configuration**: Choose the machine type (e.g., ```e2-medium```).
- **Boot Disk**: Select an OS (e.g., ```Ubuntu 22.04```).
- **Size(GB)**: ```20```

**3. Add the First Network Interface:**

- Under **Network Interfaces**, the first interface is pre-filled.
- Choose the first VPC network (e.g., ```vpc-1```) and subnet (e.g., ```subnet-1```).

**4. Add the Second Network Interface:**

- Click Add **Network Interface**.
- Select the second VPC network (e.g., ```vpc-2```) and subnet (e.g., ```subnet-2```).

**5. Finalize and Create:**
- Review your settings and click **Create**.



**```ssh into the VM and follow below steps to Install Docker and Pull Router Docker image from GCP Artifact Registry```**
```
sudo apt update
sudo apt install docker.io -y
docker version
sudo docker pull us-east1-docker
```
