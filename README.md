
# AWS & Splunk SOC Lab

This project is a step-by-step guide to building a secure cybersecurity cloud lab in AWS. It connects an offensive attacking environment with defensive monitoring tools.

## 🗺️ Lab Architecture
Here is how the network, firewalls, and data logging streams are set up across the cloud:
<img width="980" height="535" alt="c" src="https://github.com/user-attachments/assets/e694bac3-8ea1-427a-8d9b-62c9ec565ed1" />

## 🚀 How It Works
The lab separates the attack target from the monitoring station to mirror a real enterprise network:
* **Target Web App:** An Ubuntu cloud server (`t3.micro`) that runs a vulnerable web application (DVWA) inside a Docker container.
* **Splunk SIEM Server:** A heavy cloud server (`m7i-flex.large`) that hosts Splunk Enterprise to collect and look through security data.
* **The Log Pipeline:** A Splunk Universal Forwarder is installed on the web app server. It safely captures and streams live application logs over a private AWS network straight to the Splunk server on Port 9997.

## 🏹 Threat Hunting Scenarios Covered
1. **Web Scanning Detection:** Using Splunk search queries to spot automated scanners (`Gobuster`) trying to brute-force web pages.
2. **File Stealing Interception:** Spotting hackers trying to steal internal system files (`/etc/passwd`) by finding their unique signatures in the logs.

---

## 📂 Download the Full Guide
The complete manual has all the step-by-step screenshots, firewall rules, and exact terminal commands you need to build this yourself:

## Video:
https://www.linkedin.com/posts/gurunanakadhikari_aws-security-lab-i-built-a-security-lab-in-ugcPost-7461451066044289024-3EG-?utm_source=social_share_send&utm_medium=member_desktop_web&rcm=ACoAAEtEqhwBtd9Rjbr84IsWwWRE8ExCL1UNzXU
