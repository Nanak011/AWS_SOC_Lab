# AWS & Splunk SOC Lab Guide
## Step-by-Step Infrastructure Setup & Documentation

**Project Lab Design**

This document presents the implementation guide to build a functional cybersecurity home lab. The environment contains two main instances deployed inside an isolated cloud network:

**Target Instance (t3.micro):** Hosts a vulnerable application via a Docker container wrapper, isolating vulnerabilities from the host operating system.

**Splunk Instance (m7i-flex.large):** Acts as the centralized SIEM workspace, collecting container application logs over a secure internal subnet stream.

- **Prerequisites: WSL ubuntu (download from Microsoft store) and Remote Desktop Connection**

---

## Phase 1: Setting Up the Network & Splunk

### Step 1: Build the Custom VPC

• Navigate to the AWS VPC Dashboard and click Create VPC.
• Check VPC and more to automatically bundle network segments.
• Set Name tag auto-generation to: Cyber-Lab
• Match IPv4 CIDR block strictly to: 10.0.0.0/16
• Set Availability Zones (AZs) to 1, Public Subnets to 1, and Private Subnets to 0.
• Turn off NAT Gateways and VPC Endpoints, and ensure DNS hostnames/resolution remain active.

### Step 2: Establish the Security Group (Firewall)

Generate a new Security Group named 'splunk-siem-sg' assigned directly to your 'Cyber-Lab' VPC network loop. Define the inbound parameters precisely as follows:

| **Access Profile** | **Protocol Type** | **Target Port** | **Source Boundary** |
| --- | --- | --- | --- |
| SSH Management access | SSH | 22 | Your Workstation Public IP |
| RDP Graphic access | RDP | 3389 | Your Workstation Public IP |
| Internal Forwarder Feed | Custom TCP | 9997 | VPC Subnet Block (10.0.0.0/16) |

### Step 3: Provision the Splunk Host Computer

• Launch a new EC2 Instance named 'Splunk-SIEM-GUI'.
• Base Image Selection: Ubuntu Server 24.04 LTS (HVM).
• Instance Compute Sizing: m7i-flex.large (allocates the required 8 GB RAM environment).
• Network configuration: Bind to 'Cyber-Lab' VPC, turn Auto-assign Public IP to enabled, attach 'splunk-siem-sg', and provision 20 GiB gp3 storage space.

### Step 4: Initialize the Graphic Desktop Engine

Connect to your server via your console window(open ubuntu wsl, run the command in same folder as your access key) and run these configurations:

```
sudo apt update && sudo apt upgrade -y
sudo apt install xfce4 xfce4-goodies xrdp -y
echo "xfce4-session" > ~/.xsession
sudo systemctl restart xrdp
sudo passwd ubuntu
```

### Step 5: Install Splunk Enterprise Core

**Open Remote Desktop connection application and enter the IP of the Splunk instance.**

Download splunk then:

```
cd ~/Downloads
sudo dpkg -i splunk*.deb
cd /opt/splunk/bin
sudo ./splunk start --accept-license –run-as-root
```

Open Splunk via your dashboard layout at http://localhost:8000, navigate to Settings > Forwarding and receiving > Configure receiving, click Add New, input port 9997, and save.

---

## Phase 2: Setting Up the Vulnerable Target Range

### Step 1: Deploy the Web Target Node

Provision an EC2 instance as 'Web-App-Target' size t3.micro inside your 'Cyber-Lab' network. Attach a security group named 'web-target-sg' that allows Port 22 from your Home IP and open Port 80 to global web traffic access.

### Step 2: Initialize the Vulnerable Web Application Container

SSH into the web server machine then:

```
sudo apt update && sudo apt install docker.io -y
sudo systemctl start docker && sudo systemctl enable docker
sudo docker run -d -p 80:80 --name dvwa-app vulnerables/web-dvwa
```

Verify by loading http://YOUR_TARGET_PUBLIC_IP in your local web browser, logging in with credentials 'admin' and 'password', and running 'Create / Reset Database' at the page base.

### Step 3: Install the Splunk Universal Forwarder

Pull and configure the local collection log engine agent on your target operating system space:

```
wget -O splunkforwarder.deb "https://download.splunk.com/products/universalforwarder/releases/10.2.1/linux/splunkforwarder-10.2.1-c892b66d163d-linux-amd64.deb"
sudo dpkg -i splunkforwarder.deb
cd /opt/splunkforwarder/bin
sudo ./splunk start --accept-license –run-as-root
```

### Step 4: Configure Linux Permission Passages and Link the Feed

Point your collector stream directly to the private network address of your Splunk master instance. Manage directory access properties to avoid pipeline blocks:

```
sudo ./splunk add forward-server YOUR_SPLUNK_PRIVATE_IP:9997
sudo chmod -R 644 /var/lib/docker/containers/
sudo /opt/splunkforwarder/bin/splunk add monitor /var/lib/docker/containers/ -index main -sourcetype dvwa:json
sudo /opt/splunkforwarder/bin/splunk restart
```

---

## Phase 3: Threat Hunting Execution Playbook

### Exercise 1: Reconnaissance Mapping

Offensive Execution Command (Kali Terminal):

```
gobuster dir -u http://YOUR_TARGET_IP -w /usr/share/wordlists/dirb/common.txt
```

Defensive Threat Hunting String (Splunk SPL Search Box):

```
index=main sourcetype="dvwa:json" "Gobuster" │ stats count by log
```

**Detection Signature:** Exposes high-speed automated brute-force discovery loops, tracking 404 status layout anomalies inside the log space.

### Exercise 2: Path Traversal Interception (Confidential System File Exploits)

Offensive Execution Command (Kali Terminal):

```
curl -A "Hacker-User-Agent" "http://YOUR_TARGET_IP/index.php?page=../../../../etc/passwd"
```

Defensive Threat Hunting String (Splunk SPL Search Box):

```
index=main "etc/passwd" │ table log
```

**Detection Signature:** Isolates explicit attempts to dump system file structures, tracking the unique user-agent context values used to fire the request.
