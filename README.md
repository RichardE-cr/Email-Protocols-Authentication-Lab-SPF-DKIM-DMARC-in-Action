# Email Protocols & Authentication Lab: SPF, DKIM, DMARC in Action

<img width="1147" alt="Email header analysis-SPF DKIMconfigured " src="https://github.com/user-attachments/assets/bfc9c53d-e25a-4d54-a5db-39505d0a7a3b" />

## Tools
- **Mailgun** – for email sending and simulation
- **SPF / DKIM / DMARC** – email authentication protocols
- **DNS (Hostinger)** – domain record configuration
- **Azure VM (Ubuntu 24.04)** – hosting test environment
- **Email Header Forensics** – analyzing delivery outcomes and protocol alignment

**Focus Areas:**  
Secure Email Delivery · Protocol Troubleshooting · Email Security Alignment

---

## Overview
This lab demonstrates the full lifecycle of configuring, sending, and troubleshooting simulated phishing emails using Mailgun and a custom domain. It walks through how SPF, DKIM, and DMARC records are implemented and validated, then applies header forensics to verify protocol alignment and diagnose delivery issues.

The workflow mirrors common real-world scenarios that support engineers face—particularly in roles that assist organizations in ensuring their email-based security awareness tools function properly. This includes resolving issues like:
- Emails being flagged as spam
- Misconfigured DNS or authentication records
- Failures in spoofing protections

By mimicking these practical challenges, the lab strengthens core skills in email protocol troubleshooting, secure delivery, and client-facing technical communication.


## Step 1: Environment Setup (Azure VM)
* Provisioned an Ubuntu 24.04 VM (email-rich) in Microsoft Azure
* Configured the VM with:
    * Remote Desktop Protocol (RDP) via xrdp
* Opened necessary ports:
    * TCP 3389 (RDP access)
    * TCP 25 (SMTP delivery test)

<img width="1422" alt="Downloading xrdp-email-rich" src="https://github.com/user-attachments/assets/e04bbfad-5159-416c-8bad-9cc319fcc437" />


## Step 2: Configure Domain Email Authentication
Domain Used: richard-demetrius.com 
DNS Manager: Hostinger
 Email Provider: Mailgun

After creating a Mailgun account and adding my domain from Hostinger, I accessed the necessary DNS records (SPF, DKIM, and DMARC) that needed to be configured for secure email authentication. These records authorize Mailgun to send emails on behalf of my domain and protect against spoofing or tampering. 

**Mailgun DNS Records Panel:** 

<img width="1317" alt="Unverified SPF   DKIM-Mailgun" src="https://github.com/user-attachments/assets/65707991-6266-4c62-91e9-6d661b63fb96" />

### DNS Record Purposes & Verification

Once the records were added, I used `nslookup` to verify that the changes had propagated correctly:

```bash 
# SPF – Authorizes Mailgun to send emails from the domain. 
nslookup -type=txt richard-demetrius.com

# DKIM – Cryptographically signs messages to validate integrity and authenticity. 
nslookup -type=txt pic._domainkey.richard-demetrius.com

# DMARC – Instructs receiving servers how to handle SPF/DKIM failures (e.g., quarantine)
nslookup -type=txt _dmarc.richard-demetrius.com
```

**Verification Output (SPF / DKIM / DMARC):** 
<img width="1438" alt="Verifying SPF-DKIM-DMARC-using-nslookup" src="https://github.com/user-attachments/assets/dfa6e3ff-de77-43be-a4e3-97f9902fe990" />




## Step 3: Send a Valid Test Email via Mailgun

Generate API key in Mailgun to send the email via the CLI

<img width="1161" alt="GeneratingAPIkey-Mailgun" src="https://github.com/user-attachments/assets/8eda53ae-d8d2-4104-9afa-7d02528d47a4" />


Command:
```bash

sudo curl -s --user 'api:720d3c0facfdce03ebb93499b49e2627-6d5bd527-xxxxxx' \
https://api.mailgun.net/v3/richard-demetrius.com/messages \
-F from=info@richard-demetrius.com \
-F to=sec.user.acc@gmail.com \
-F subject='Emial Protocol test' \
-F text='This is a test email via Mailgun with SPF/DKIM/DMARC configured!'
``` 

Email delivered successfully to the inbox

<img width="1417" alt="Gmail-email1recieved-to inbox" src="https://github.com/user-attachments/assets/08ed4792-361f-47aa-b4f3-616955492091" />




<img width="1361" alt="Contents of 1st email recieved" src="https://github.com/user-attachments/assets/d7f6bfc7-da94-4ef7-951e-cff3f9a27b64" />





## Step 4: Perform Header Forensics
Header analysis validated that security controls and trust alignment were working as expected:
Field	Result
SPF	- pass
DKIM	- pass
DMARC	- pass
Return-Path Alignment	- yes
Envelope Sender Match	- yes
Mailgun IP Verified	- yes
Spam Score	- Low

<img width="1293" alt="Emailheader-googleoriginalmessage-email1" src="https://github.com/user-attachments/assets/53be759b-f523-4798-89c9-ce5d75b39fef" />


Tools/Techniques used:
* Analyzed Authentication-Results fields
* Checked SPF/DKIM/DMARC status and selector
* Verified sender IP reputation and domain alignment
* Simulated what customers would do when checking why an email landed in spam or failed to arrive

## Step 5: Simulate & Resolve Failed Delivery
Intentional Misconfiguration
* Removed SPF/DKIM records to simulate a misconfigured domain
* Sent email using the same command:
Expected Result: Rejected Response:
json
CopyEdit
"Domain richard-demetrius.com is not allowed to send: The domain is unverified and requires DNS configuration."

<img width="1439" alt="error-sendingemail-removed DKIM SPF config" src="https://github.com/user-attachments/assets/07a90cee-b38c-4a67-86d5-3e29a77da2e7" />



### Reconfigure and Resend
* Restored correct SPF and DKIM records in DNS
* Sent a new email:

``` bash
sudo curl -s --user 'api:720d3c0facfdce03ebb93499b49e2627-6d5bd527-xxxxxxx' \
https://api.mailgun.net/v3/richard-demetrius.com/messages \
-F from='Info <info@richard-demetrius.com>' \
-F to=sec.user.acc@gmail.com \
-F subject='Third Email Test' \
-F text='In this test email I reconfigured SPF and DKIM to see if the sending error will be corrected, if it goes to the main inbox and to analyze the email header'
```

Delivery Successful  Header Re-analysis confirmed correct alignment and security posture

<img width="1359" alt="Contents of third email" src="https://github.com/user-attachments/assets/33dd44f2-c790-4ebf-bced-c0a227a0fbec" />



## Key Takeaways for Technical Support & Security Roles
* Hands-on with SPF, DKIM, DMARC: Understanding not just how to configure these records but how to verify and troubleshoot them when mail delivery fails
* Practical troubleshooting scenarios: Simulating issues customers face when security email campaigns are misclassified or rejected
* Forensic-level email header analysis: Skill in diagnosing deliverability, spam filtering, and authentication failures
* Support-readiness: This lab mirrors the type of real-time diagnostics technical support engineers and network security staff provide when collaborating with customers and internal teams


## Final Thoughts
Email deliverability is a critical factor in phishing simulations and user training campaigns. This lab strengthened my technical troubleshooting skills, deepened my understanding of authentication protocols, and reinforced how firewall rules, DNS, and spam controls interact during real-world email delivery.



n.b - before this lab is published the API key used and displayed throughout was deleted and is no longer functional (my appologies to the hackers out there)

<img width="603" alt="APIkey-deleted" src="https://github.com/user-attachments/assets/4559732f-5522-4361-bb08-c60789c462f4" />



