# Email Protocols & Authentication Lab: SPF, DKIM, DMARC in Action

<img width="1147" alt="Email header analysis-SPF DKIMconfigured " src="https://github.com/user-attachments/assets/bfc9c53d-e25a-4d54-a5db-39505d0a7a3b" />

## Tools
- **Mailgun** – for email sending and simulation
- **SPF / DKIM / DMARC** – email authentication protocols
- **DNS (Hostinger)** – domain record configuration
- **Azure VM (Ubuntu 24.04)** – hosting test environment
- **Email Header Forensics** – analyzing delivery outcomes and protocol alignment

**Focus Areas:**  
- Secure Email Delivery 
- Protocol Troubleshooting 
- Email Security Alignment

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

With the DNS authentication records correctly configured, the next step was to verify successful email delivery using Mailgun’s API.


I generated a private API key from the Mailgun dashboard, which enables programmatic sending of messages through the domain.

**Mailgun API Key Panel:**
<img width="1161" alt="GeneratingAPIkey-Mailgun" src="https://github.com/user-attachments/assets/8eda53ae-d8d2-4104-9afa-7d02528d47a4" />


### Sending a Test Email via `curl`
Using `curl`, I sent a simulated test message that aligned with SPF, DKIM, and DMARC records to confirm it would pass through security filters and arrive in the inbox:

```bash

sudo curl -s --user 'api:720d3c0facfdce03ebb93499b49e2627-6d5bd527-xxxxxx' \
https://api.mailgun.net/v3/richard-demetrius.com/messages \
-F from=info@richard-demetrius.com \
-F to=sec.user.acc@gmail.com \
-F subject='Emial Protocol test' \
-F text='This is a test email via Mailgun with SPF/DKIM/DMARC configured!'
``` 

### Delivery Confirmation
The email was successfully delivered to the primary inbox (not spam), verifying that authentication records were working as expected.

<img width="1417" alt="Gmail-email1recieved-to inbox" src="https://github.com/user-attachments/assets/08ed4792-361f-47aa-b4f3-616955492091" />



<img width="1361" alt="Contents of 1st email recieved" src="https://github.com/user-attachments/assets/d7f6bfc7-da94-4ef7-951e-cff3f9a27b64" />



## Step 4: Analyze Email Header

Once the test email was delivered, I retrieved the full email header from Gmail to verify whether it successfully passed all authentication checks (SPF, DKIM, DMARC) and aligned with secure sending best practices.

Email Delivered To: sec.user.acc@gmail.com
Sender: info@richard-demetrius.com
Mailgun Sending IP: 159.135.228.58

Email Authentication Results Summary:
- SPF = Softfail – google.com: domain ... does not designate 159.135.228.58 as permitted sender (likely because hostinger added dns records to accomodate the new email account I added to the domain in between my first spf verification and sending the email, however it only 'softfailed' and the email was still delivered to the main inbox.
- DKIM = Pass – Verified for both richard-demetrius.com and mailgun.org
- DMARC = Pass – DMARC validation passed with policy = p=NONE
- Return-Path Alignment = Aligned – Return-path domain matches the From header (richard-demetrius.com)
- Envelope Sender Match = Match – Envelope sender aligns with From domain
- Mailgun IP Verified = Yes – IP 159.135.228.58 is a known valid Mailgun outbound IP
- Spam Score = Unknown – No explicit score in header; Gmail delivered to inbox (likely low score)

Original Header information: 

```txt
Delivered-To: sec.user.acc@gmail.com
Received: by 2002:a05:7022:65aa:b0:a3:fc22:5a87 with SMTP id bd42csp6527373dlb;
        Sun, 6 Jul 2025 13:50:02 -0700 (PDT)
X-Google-Smtp-Source: AGHT+IHii5FmiSb50zB8bDCPaESKHC2UPvTB4bdkzSB1gwaSQIwV04ftYR+YiP5TcUuDulDwrkVZ
X-Received: by 2002:ad4:5f0a:0:b0:6fb:59de:f8ab with SMTP id 6a1803df08f44-702c8b72059mr173879136d6.10.1751835002701;
        Sun, 06 Jul 2025 13:50:02 -0700 (PDT)
ARC-Seal: i=1; a=rsa-sha256; t=1751835002; cv=none;
        d=google.com; s=arc-20240605;
        b=i5cG8KyWrRo8kwB5Xfh6IpFzJ17+jLJOlirs57QVwMvl/s/ZcAUFxQeFEMG0vXfSGC
         aChBiguuiTMDqf/Qx+RxGTxKiarXJtWBEU7KDv0YPNwVy1JVMbCj6BVMUPeuRlx+R8zv
         vxAC6WtfSkyJoNdk6Hfgy5wGhfnBN30Jc3Nw+kgQoMjvyDaAhSPu0LCyi1iUHqYMVBMz
         +CC+BjZVmARGF/jFkd/381+FqbWzsUmaTA6NkKL7Z0d6Y0FELWfCzVxACTL7vDPYPe9V
         jETlWFraVH0LlzABnIoCCWfeuTSauBI7rKx1mgXSpQxC/aoOp79c9irpKgnu2LS801bC
         LsnA==
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20240605;
        h=content-transfer-encoding:message-id:to:from:subject:mime-version
         :date:sender:dkim-signature:dkim-signature;
        bh=5aUrTd9VatdCzrmHgSsZVtkLwzWmVFx+/XV1SxSJUoM=;
        fh=UjQwB7gVzF3ejA4wJRhvG95yIXd8ZfDVUQ9TVhkjl2Y=;
        b=biPXE2wGDySzmgOXom9L+kS7hU4s5Kz1QfohwM1fPfGuWyDM8wZRK/z2I+1sBePX8T
         ttarD0BPmw/Ccd+qYCt79EIsCpvLUQVmtFJHUafxNxbUbQh8HEkjdg+2csgseeCKnwcW
         OQA0w1k6nrrgd4FdEbjM+UCAv6pua5wFbap78y9MopqU/h1I7DNMJ5Io88Z6A5pacFhq
         iYXB3H9CZkqGT9/HGi2QCXYPI2zR3K4bfQTcBrmKJJOVIx3IRyWV+saB/ghsvKp3eLtR
         Opw7kugf5GOt1I5i+Ra+ps8fBj5+kNPDraIKD6vq4EXwR4iC5BRyie396PXTxRfTvt4L
         WVjA==;
        dara=google.com
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@richard-demetrius.com header.s=pic header.b=dLNqRpUJ;
       dkim=pass header.i=@mailgun.org header.s=mg header.b=P1oY7X6C;
       spf=softfail (google.com: domain of transitioning bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com does not designate 159.135.228.58 as permitted sender) smtp.mailfrom="bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com";
       dmarc=pass (p=NONE sp=NONE dis=NONE) header.from=richard-demetrius.com
Return-Path: <bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com>
Received: from m228-58.mailgun.net (m228-58.mailgun.net. [159.135.228.58])
        by mx.google.com with UTF8SMTPS id 6a1803df08f44-702c4d645e0si63868346d6.460.2025.07.06.13.50.02
        for <sec.user.acc@gmail.com>
        (version=TLS1_2 cipher=ECDHE-ECDSA-AES128-GCM-SHA256 bits=128/128);
        Sun, 06 Jul 2025 13:50:02 -0700 (PDT)
Received-SPF: softfail (google.com: domain of transitioning bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com does not designate 159.135.228.58 as permitted sender) client-ip=159.135.228.58;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@richard-demetrius.com header.s=pic header.b=dLNqRpUJ;
       dkim=pass header.i=@mailgun.org header.s=mg header.b=P1oY7X6C;
       spf=softfail (google.com: domain of transitioning bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com does not designate 159.135.228.58 as permitted sender) smtp.mailfrom="bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com";
       dmarc=pass (p=NONE sp=NONE dis=NONE) header.from=richard-demetrius.com
DKIM-Signature: a=rsa-sha256; v=1; c=relaxed/relaxed; d=richard-demetrius.com; q=dns/txt; s=pic; t=1751835001; x=1751842201; h=Content-Type: Content-Transfer-Encoding: Message-Id: To: To: From: From: Subject: Subject: Mime-Version: Date: Sender: Sender: X-Feedback-Id; bh=5aUrTd9VatdCzrmHgSsZVtkLwzWmVFx+/XV1SxSJUoM=; b=dLNqRpUJNYRJpRbkzL0LhJtwpigmnDjR+8UbuM+90Wlth8dxtNYl2/ufBmSw394uz5BlPX6unXyErDPYwrRu016emClZ1XxrFlLUI+idCrTTg16YP55pCSh8gn0opUQK77W19GXkXqAJn32xuPPxKFuRw55brhehfJ3dM+yXFaA=
DKIM-Signature: a=rsa-sha256; v=1; c=relaxed/relaxed; d=mailgun.org; q=dns/txt; s=mg; t=1751835001; x=1751842201; h=Content-Type: Content-Transfer-Encoding: Message-Id: To: To: From: From: Subject: Subject: Mime-Version: Date: Sender: Sender: X-Feedback-Id; bh=5aUrTd9VatdCzrmHgSsZVtkLwzWmVFx+/XV1SxSJUoM=; b=P1oY7X6CG1pAuCCe4tJBkthrTIPOgPcl7ENfJEXxQ3pLuN3YRuoJP4400prVcUT4pMErRLZU9eWpRenuvgZ4fKP36ixxRzmcN93IB6tUPfXQ9dffUb1T2asxXyCwqrJNoQIWXMveGXehEvDz9swHj/+KdGwbBY6ic+gABvswVTE=
X-Mailgun-Sid: WyIxMDE4NSIsInNlYy51c2VyLmFjY0BnbWFpbC5jb20iLCI3OGFlMzMiXQ==
X-Feedback-Id: info@richard-demetrius.com::686ab126b6b24407032a24c9:mailgun
Received: by b32329ba7d13 with HTTP id 686ae1797323c0a6053a1f44; Sun, 06 Jul 2025 20:50:01 GMT
X-Mailgun-Sending-Ip: 159.135.228.58
Sender: info@richard-demetrius.com
Date: Sun, 06 Jul 2025 20:50:01 +0000
Mime-Version: 1.0
Subject: Emial Protocol test
From: info@richard-demetrius.com
To: sec.user.acc@gmail.com
Message-Id: <20250706205001.ec83915da6068a76@richard-demetrius.com>
Content-Transfer-Encoding: 7bit
Content-Type: text/plain; charset=ascii

This is a test email via Mailgun with SPF/DKIM/DMARC configured!
```

<img width="1293" alt="Emailheader-googleoriginalmessage-email1" src="https://github.com/user-attachments/assets/53be759b-f523-4798-89c9-ce5d75b39fef" />


## Step 5: Simulate & Resolve Failed Delivery
This step intentionally breaks authentication to simulate a real-world misconfiguration scenario that support engineers often troubleshoot.


Intentional Misconfiguration
To observe how misconfigured DNS records impact email delivery, I removed the SPF and DKIM records from the domain's DNS settings and attempted to send another email using the same Mailgun API command with slightly different content information (subject and text) :

**In this screenshot you can see there is no DKIM key present and the email was not sent and instead resulted in an error:**

<img width="1439" alt="error-sendingemail-removed DKIM SPF config" src="https://github.com/user-attachments/assets/07a90cee-b38c-4a67-86d5-3e29a77da2e7" />

```json
{"message":"Domain richard-demetrius.com is not allowed to send: The domain is unverified and requires DNS configuration. Log in to your control panel to view required DNS records."}
```

### Reconfigure and Resend
To resolve the issue:

- SPF and DKIM records were restored using the correct values in the DNS provider
- DNS propagation was verified using nslookup
- Sent a third test email after fixing DNS:

``` bash
sudo curl -s --user 'api:720d3c0facfdce03ebb93499b49e2627-6d5bd527-xxxxxxx' \
https://api.mailgun.net/v3/richard-demetrius.com/messages \
-F from='Info <info@richard-demetrius.com>' \
-F to=sec.user.acc@gmail.com \
-F subject='Third Email Test' \
-F text='In this test email I reconfigured SPF and DKIM to see if the sending error will be corrected, if it goes to the main inbox and to analyze the email header'
```

<img width="1359" alt="Contents of third email" src="https://github.com/user-attachments/assets/33dd44f2-c790-4ebf-bced-c0a227a0fbec" />


### 🔍 Header Analysis – Post-Reconfiguration (Third Email)

After restoring the correct SPF and DKIM records, the third test email was successfully delivered. A review of the email header shows complete alignment across all email authentication mechanisms.

| Security Check              | Result       | Details                                                                                   |
|----------------------------|--------------|-------------------------------------------------------------------------------------------|
| **SPF**                    |  **Pass**   | `richard-demetrius.com` correctly authorizes `159.135.228.14` (Mailgun IP) as sender. No more **softfail** as seen in the first email. |
| **DKIM (richard-demetrius)** |  **Pass**   | Valid DKIM signature using the `pic` selector.                                            |
| **DKIM (mailgun.org)**     |  **Pass**   | Additional valid DKIM signature from Mailgun using the `mg` selector.                     |
| **DMARC**                  | **Pass**   | DMARC passed; domain alignment checks completed successfully.                             |
| **Return-Path Alignment**  |  **Aligned**| `Return-Path` domain matches `From` domain.                                               |
| **Envelope Sender Match**  |  **Match**  | Envelope sender properly aligned with domain.                                             |
| **Mailgun IP Verified**    |  **Yes**    | Delivered by `159.135.228.14` (Mailgun), verified via `Received-SPF`.                     |
| **Spam Score**             | **Low**    | Message delivered to **Inbox** (not Spam); consistent with proper configuration.          |

> **📝 Comparison Note:**  
> In the **first test email**, SPF resulted in a `softfail`, meaning the Mailgun IP was not explicitly authorized in the SPF record (`~all`).  
> After correcting the DNS configuration, SPF now returns a **full pass**, eliminating potential filtering or spam flags.

This analysis confirms that the corrected SPF and DKIM records restored full deliverability and alignment. The email successfully passed through modern email security checks — a critical validation step for any email simulation platform or security training environment.

**Email Header info for reference**

```txtDelivered-To: sec.user.acc@gmail.com
Received: by 2002:a05:7022:65aa:b0:a3:fc22:5a87 with SMTP id bd42csp6551653dlb;
        Sun, 6 Jul 2025 15:05:31 -0700 (PDT)
X-Google-Smtp-Source: AGHT+IEGV7sfus7xzXCC0ANYcgENxKJUpIa/2niaIcjDTsdKUbTLYPkgd/VR4rLJmPv86EYRY57D
X-Received: by 2002:a05:620a:248f:b0:7d0:9be0:2e35 with SMTP id af79cd13be357-7d5ef5d8d12mr786989985a.10.1751839530892;
        Sun, 06 Jul 2025 15:05:30 -0700 (PDT)
ARC-Seal: i=1; a=rsa-sha256; t=1751839530; cv=none;
        d=google.com; s=arc-20240605;
        b=l2TjjC9T5laV6SoD00qTJ7vxBCc9ZH7C6HlhhzH3718DuEPOzOP11LEWHLTbdfTojb
         Fd65XPaN9YLNybzADmsEhdbu23Z0q/OecIH3oShApEAXEbojK3U2YMz/tD6OcKYvdmpK
         nd+r1BY/WAGpqOCinJK+3lqFDqHkyRQYE7juoiNPe47r1doe0lB+qkT2ISdgnwA3a5kX
         P0gErNlMcY60/+8x6wM3Me0/bivVB8AlRUNPcHxVKBbCTxxMky2XwxKwTaFJUufbW7a+
         66uA/XDR5ZwWJ8ZZLe9fTS/CrvTB/vkB0+s6f5ntGtAn4uKkm0x0Td0L14V7pHt1JbqC
         ONaw==
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20240605;
        h=content-transfer-encoding:message-id:to:from:subject:mime-version
         :date:sender:dkim-signature:dkim-signature;
        bh=Z091ZYaQ2E/r1Vm3ilc+tB1NodAkF9UqFaFzorN7fLM=;
        fh=UjQwB7gVzF3ejA4wJRhvG95yIXd8ZfDVUQ9TVhkjl2Y=;
        b=VfX1b7tV+6QCkE3GP3O5mvo5mItvWuyiJ6FQKpH3HG9tsI/IVXcIdZm8ZzozUPbwXk
         039CVdwnj+k5EmlW4lVyIiai3zdRMpsZijlKKpZ5FtbU3ILfK45r5I3hu+d65rj9w3D5
         1Nt8fJnNWGZXwaeZaWB+JdIYY1Lx+5g/f9jv00m+VBuJYM+32hiQCAXoEolKPuwuhbXD
         vyKc/OkAPBSs8iNFrmWWIYp8/rhfDf22IONjJKHQDW+v6V1zqLrGkVKJxy9tTCftC0fU
         udmnBQpm2/DwUO28XXZFDmm+1J5eJh1h1bCV5sYeTG6Vr4GmgNvIfJI6/zREd5gYNQhh
         PSnA==;
        dara=google.com
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@richard-demetrius.com header.s=pic header.b=rWhh8Aac;
       dkim=pass header.i=@mailgun.org header.s=mg header.b=TQW8JN7A;
       spf=pass (google.com: domain of bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com designates 159.135.228.14 as permitted sender) smtp.mailfrom="bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com";
       dmarc=pass (p=NONE sp=NONE dis=NONE) header.from=richard-demetrius.com
Return-Path: <bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com>
Received: from m228-14.mailgun.net (m228-14.mailgun.net. [159.135.228.14])
        by mx.google.com with UTF8SMTPS id af79cd13be357-7d5dbf04221si641540185a.644.2025.07.06.15.05.30
        for <sec.user.acc@gmail.com>
        (version=TLS1_2 cipher=ECDHE-ECDSA-AES128-GCM-SHA256 bits=128/128);
        Sun, 06 Jul 2025 15:05:30 -0700 (PDT)
Received-SPF: pass (google.com: domain of bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com designates 159.135.228.14 as permitted sender) client-ip=159.135.228.14;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@richard-demetrius.com header.s=pic header.b=rWhh8Aac;
       dkim=pass header.i=@mailgun.org header.s=mg header.b=TQW8JN7A;
       spf=pass (google.com: domain of bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com designates 159.135.228.14 as permitted sender) smtp.mailfrom="bounce+bb7518.78ae33-sec.user.acc=gmail.com@richard-demetrius.com";
       dmarc=pass (p=NONE sp=NONE dis=NONE) header.from=richard-demetrius.com
DKIM-Signature: a=rsa-sha256; v=1; c=relaxed/relaxed; d=richard-demetrius.com; q=dns/txt; s=pic; t=1751839530; x=1751846730; h=Content-Type: Content-Transfer-Encoding: Message-Id: To: To: From: From: Subject: Subject: Mime-Version: Date: Sender: Sender: X-Feedback-Id; bh=Z091ZYaQ2E/r1Vm3ilc+tB1NodAkF9UqFaFzorN7fLM=; b=rWhh8AacdCSI6rX7gaz+F+2VQNNDvva/bhbGbqjhNQyIpbTW6GMvoxsNRcasd2fE6bizs61TJ4OShQMq/W0XvatNAZlZ44AR209+YyI2iUf4JuD0qSNV9GHwGmALZReL8vVqw8fTIlWzFJp8aIKHPfRqCtrc9wqecoJQuuVfXqY=
DKIM-Signature: a=rsa-sha256; v=1; c=relaxed/relaxed; d=mailgun.org; q=dns/txt; s=mg; t=1751839530; x=1751846730; h=Content-Type: Content-Transfer-Encoding: Message-Id: To: To: From: From: Subject: Subject: Mime-Version: Date: Sender: Sender: X-Feedback-Id; bh=Z091ZYaQ2E/r1Vm3ilc+tB1NodAkF9UqFaFzorN7fLM=; b=TQW8JN7AQSf3mHkAJyVUexXC33fIZEKJRdszKxcS02wIiHN/6MjQ7vGcoT3eior6mlaoEpyiVTNiyIEL9B/YbZccVTBwnYiqd2/rOT2Ibwj52eosrpYfj9ozOho+S12wwQ0A1gYFNQDaZmXAb32ji11KAQBGULLDzJLaal2YhWw=
X-Mailgun-Sid: WyIxMDE4NSIsInNlYy51c2VyLmFjY0BnbWFpbC5jb20iLCI3OGFlMzMiXQ==
X-Feedback-Id: info@richard-demetrius.com::686ab126b6b24407032a24c9:mailgun
Received: by cb79a5d26fd6 with HTTP id 686af32a466414c36e66c160; Sun, 06 Jul 2025 22:05:30 GMT
X-Mailgun-Sending-Ip: 159.135.228.14
Sender: info@richard-demetrius.com
Date: Sun, 06 Jul 2025 22:05:30 +0000
Mime-Version: 1.0
Subject: Third Email Test
From: Info <info@richard-demetrius.com>
To: sec.user.acc@gmail.com
Message-Id: <20250706220530.d12ce7ffdd9d2a14@richard-demetrius.com>
Content-Transfer-Encoding: 7bit
Content-Type: text/plain; charset=ascii

In this test email I reconfigured SPF and DKIM to see if the sending error will be corrected, if it goes to the main inbox and to analyze the email header
```

## Key Takeaways for Technical Support & Security Roles

- **SPF, DKIM, and DMARC Must Be Aligned**: Proper DNS configuration is essential for achieving secure and trusted email delivery. Misalignments, even partial like an SPF `softfail`, can still result in delivery — but they introduce risk and reduce reputation.

- **DNS Changes Take Time to Propagate**: Misconfiguration errors (like missing or outdated SPF/DKIM records) can take time to correct due to DNS TTL settings. Use tools like `nslookup` and Mailgun’s dashboard to confirm propagation before retesting.

- **Email Header Analysis is a Critical Diagnostic Tool**: Reading the full email header allows teams to verify SPF/DKIM/DMARC results, identify the sending IP, and troubleshoot alignment or policy issues — critical for email spoofing prevention and forensic response.

- **Simulated Failure Conditions Build Real Readiness**: Intentionally removing authentication records mid-lab replicates real-world troubleshooting scenarios where emails mysteriously stop delivering due to DNS or mail server misconfigurations.

- **Vendor-Specific Email Behavior Matters**: Despite SPF softfails, Gmail still delivered the message to the inbox. Understanding how different providers interpret email authentication outcomes is key for tuning sender reputation and support response.

- **Support Teams Must Validate from multiple angles**:
  - Was the sending IP authorized in SPF?
  - Did the DKIM selector match what the recipient domain expects?
  - Did DMARC pass, and if not, what action did the recipient server take?
  
  These questions should drive both initial setup and incident response workflows.

---

## Final Thoughts

This lab effectively simulates a real-world deployment of secure email protocols — not just from a theoretical perspective, but through practical testing, error simulation, and deep header inspection.

By leveraging Mailgun, DNS tools, and email forensics, I recreated a support ticket scenario where:
- Emails stopped delivering due to record removal.
- The issue was diagnosed using error messages and header analysis.
- DNS records were restored and validated before retesting.

In both **technical support** and **cybersecurity operations**, these skills directly translate into resolving customer-facing email issues, hardening organizations against spoofing, and enabling secure SaaS or phishing simulation platforms.


> n.b - before this lab is published the API key used and displayed throughout was deleted and is no longer functional :)
> <img width="603" alt="APIkey-deleted" src="https://github.com/user-attachments/assets/4559732f-5522-4361-bb08-c60789c462f4" />



