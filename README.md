# SBT-DF204 – Computer Forensics Case Study 1

## Investigating Harassment Email Traffic With Wireshark

# Case Study 1 – Individual Forensic Investigation

| **Field** | **Details** |
|---|---|
| **Student Name** | Adekunle Ogunyemi |
| **Course** | SBT-DF204 – Computer Forensics Case Studies |
| **Assessment** | Case Study 1 – Individual Forensic Investigation |
| **Case Title** | Investigating Harassment Email Traffic With Wireshark |
| **Operating System** | Kali Linux |
| **Forensic Tool** | Wireshark |
| **Evidence File** | `nitroba.pcap` |


---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Case Background](#case-background)
- [Objectives of the Investigation](#objectives-of-the-investigation)
- [Forensic Environment](#forensic-environment)
- [Evidence Acquisition and Integrity](#evidence-acquisition-and-integrity)
- [Creation of the Working Copy](#creation-of-the-working-copy)
- [Wireshark Analysis](#wireshark-analysis)
- [Finding 1 – Identifying the Client System](#finding-1--identifying-the-client-system)
- [Finding 2 – Identifying the Device](#finding-2--identifying-the-device)
- [Finding 3 – Identifying the Anonymous Email Service](#finding-3--identifying-the-anonymous-email-service)
- [Finding 4 – Harassment Message Submission](#finding-4--harassment-message-submission)
- [HTTP Form Data Analysis](#http-form-data-analysis)
- [Finding 5 – Second Anonymous Email Submission](#finding-5--second-anonymous-email-submission)
- [Finding 6 – Associating the Device With an Email Account](#finding-6--associating-the-device-with-an-email-account)
- [Finding 7 – Chemistry 109 Roster Comparison](#finding-7--chemistry-109-roster-comparison)
- [Timeline of Relevant Activity](#timeline-of-relevant-activity)
- [Evidence Log](#evidence-log)
- [Attribution Assessment](#attribution-assessment)
- [Limitations of the Investigation](#limitations-of-the-investigation)
- [Conclusion](#conclusion)
- [Recommendation](#recommendation)
- [Screenshot Index](#screenshot-index)
- [Final Statement](#final-statement)

---

## Executive Summary

This case study involved the investigation of network traffic relating to a harassment complaint made by Lily Tuckrige, a Chemistry teacher. The purpose of the investigation was to examine the supplied packet capture and determine whether the traffic provided enough evidence to connect the harassment message to a particular device and, where possible, to a person on the Chemistry 109 class roster.

The investigation was carried out using Kali Linux and Wireshark. The supplied nitroba.pcap file was first checked and preserved before analysis. A working copy was created so that the original evidence would not be used directly during the investigation.

The packet capture showed HTTP traffic from the internal IP address 192.168.15.4 to the web service www.willselfdestruct.com. The Ethernet information associated with the traffic showed the MAC address 00:17:f2:e2:c0:ce.

The most important finding was Frame 83601. This packet contained an HTTP POST request to:

http://www.willselfdestruct.com/secure/submit

The POST data contained the recipient's email address, the subject of the message, and the actual harassment message. The recipient was Lilytuckrige@yahoo.com, the subject was “you can't find us”, and the message stated that the recipient should stop teaching and start running.

Further traffic from the same device contained a Gmail cookie showing jcoachj@gmail.com. This account is associated with Johnny Coach, who is included on the Chemistry 109 roster.

Based on the evidence examined, the traffic strongly connects the harassment message to the device using 192.168.15.4 and MAC address 00:17:f2:e2:c0:ce, and the available browser evidence associates that device with jcoachj@gmail.com. However, the evidence does not prove with absolute certainty that Johnny Coach was physically operating the device at the exact time of the message. The open wireless network in the residence is an important limitation.


---

## Case Background

Lily Tuckrige, a Chemistry teacher, reported receiving harassing emails. An earlier email had been traced to an IP address associated with a shared student residence where an open wireless network was being used.

A later message was sent through the web service `willselfdestruct.com`. The university had captured network traffic from the residence and supplied the packet capture for examination.

The main purpose of this investigation was therefore to examine the network traffic and determine:

1. Which client system contacted the web service.
2. What evidence connected the client to the harassment message.
3. Which device made the request.
4. Whether the device could be associated with a person.
5. Whether that person appeared on the Chemistry 109 roster.
6. When the activity occurred.
7. What conclusion could reasonably be made from the available evidence.

---

## Objectives of the Investigation

The objectives of this investigation were:

- To preserve the supplied network capture.
- To verify the integrity of the evidence file.
- To identify the client system that communicated with the anonymous email service.
- To identify the relevant HTTP requests and POST data.
- To identify the MAC address of the device involved.
- To examine browser and cookie information that could help associate the device with a user.
- To compare the identified account with the Chemistry 109 roster.
- To establish a timeline of the relevant activity.
- To provide a conclusion based only on the evidence available in the packet capture.

---

## Forensic Environment

The investigation was performed using the following environment:

| Item | Details |
| --- | --- |
| Operating System | Kali Linux |
| Main forensic tool | Wireshark |
| Evidence format | PCAP |
| Evidence file | `nitroba.pcap` |
| Working file | `nitroba_working.pcap` |
| Network protocol examined | HTTP |
| Main service examined | `www.willselfdestruct.com` |

The analysis was performed on a working copy of the packet capture rather than making changes to the original evidence file.

---

## Evidence Acquisition and Integrity

### Evidence File

The supplied evidence file was:

`nitroba.pcap`

The file was stored in the evidence directory:

`SBT-DF204-Case_Study/evidence/nitroba.pcap`

The Kali Linux terminal showed the file size as approximately **54 MB**.

**Figure 1: Evidence file `nitroba.pcap` and its recorded file size in Kali Linux**

The screenshot shows the terminal command used to list the evidence file, including the file name and size.

<img width="1686" height="240" alt="Fig 4" src="https://github.com/user-attachments/assets/d36ad307-4dba-4b11-bf7d-4181b5150353" />



---

### SHA-256 Verification

The SHA-256 value of the evidence file was calculated using:

```bash
sha256sum evidence/nitroba.pcap
```

The calculated hash was:

`2b77a9eaefc1d6af163d1ba793c96dbccacb04e6befdf1a0b01f8c67553ec2fb`

A working copy was also created and hashed. The SHA-256 value of the original file and the working copy matched.

This shows that the working copy was not changed during the copying process.

**Figure 2: SHA-256 comparison of the original evidence file and the working copy**

<img width="1668" height="381" alt="Fig 8" src="https://github.com/user-attachments/assets/4fba45e4-4cb9-4eb3-9f6a-2b1fe687f5fe" />



---

## Creation of the Working Copy

The working copy was created using the following command:

```bash
sudo cp --preserve=timestamps evidence/nitroba.pcap working/nitroba_working.pcap
```

The `--preserve=timestamps` option was used so that the file timestamps were retained during the copy.

The working copy was then opened in Wireshark:

```bash
wireshark working/nitroba_working.pcap
```

**Figure 3: Opening the preserved working copy in Wireshark**

<img width="1901" height="1195" alt="Screenshot 2026-10-05 114141" src="https://github.com/user-attachments/assets/7b9ef090-4496-4946-a5e0-c3badc7b7e14" />


---

## Wireshark Analysis

The investigation was mainly based on HTTP traffic because the harassment message was sent through a web-based service.

The following Wireshark filters were used during the investigation:

```text
http.request.method == "GET" && http.host == "www.willselfdestruct.com"
```

```text
http.request.method == "POST"
```

```text
data-text-lines contains "secure anonymous E-mail"
```

The HTTP requests were then examined individually to identify the relevant connection and message.

---

## Finding 1 – Identifying the Client System

The first stage of the analysis was to identify the computer that communicated with the anonymous email service.

The following filter was used:

```text
http.request.method == "GET" && http.host == "www.willselfdestruct.com"
```

The results showed HTTP requests from:

**Client IP:** `192.168.15.4`

to:

**Service IP:** `69.25.94.22`

The traffic included requests to:

`www.willselfdestruct.com/secure/submit`

This showed that the client at `192.168.15.4` was communicating with the web service involved in the investigation.

**Figure 4: HTTP requests from 192.168.15.4 to the willselfdestruct.com web service**

<img width="1661" height="926" alt="Fig 10" src="https://github.com/user-attachments/assets/6f8d6251-fdcc-4e48-920b-98f374a54637" />


---

## Finding 2 – Identifying the Device

After identifying the client IP address, the Ethernet information of the packets was examined.

The source MAC address associated with the relevant traffic was:

`00:17:f2:e2:c0:ce`

The Wireshark packet details showed:

**Source MAC:** `00:17:f2:e2:c0:ce`  
**Source IP:** `192.168.15.4`  
**Destination IP:** `69.25.94.22`

The MAC address was useful because it provided a device-level identifier for the traffic.

However, the MAC address does not identify the person who was physically using the computer. This distinction is important because the residence network was shared and open.

**Figure 5: Ethernet and IP details showing the MAC address and client IP**

<img width="1431" height="1028" alt="Fig 22" src="https://github.com/user-attachments/assets/1e34e529-453d-49fb-b898-b7021b27557b" />



---

## Finding 3 – Identifying the Anonymous Email Service

The HTTP traffic showed that the client accessed:

`www.willselfdestruct.com`

The page content identified the service as a secure anonymous email service.

The response contained page information referring to secure anonymous email and self-destructing messages.

This helped confirm that the traffic was related to the type of service described in the case scenario.

**Figure 6: Web page content identifying the secure anonymous email service**

<img width="1455" height="1147" alt="Fig 23" src="https://github.com/user-attachments/assets/1f1aa8b9-890e-4797-829d-d3d2b0a3467e" />


---

## Finding 4 – Harassment Message Submission

The most important packet in the investigation was **Frame 83601**.

The packet contained an HTTP POST request:

```text
POST /secure/submit HTTP/1.1
```

The request was sent from:

`192.168.15.4`

to:

`69.25.94.22`

The source MAC address was:

`00:17:f2:e2:c0:ce`

The HTTP request contained form data showing:

**Recipient:**

`Lilytuckrige@yahoo.com`

**Subject:**

`you can't find us`

**Message:**

`and you can't hide from us.`  
`Stop teaching.`  
`start running.`

This was direct evidence that the client generated an HTTP request containing the harassment message described in the case.

**Figure 7: Frame 83601 showing the HTTP POST containing the harassment message**

<img width="1455" height="1147" alt="Fig 23" src="https://github.com/user-attachments/assets/dca1abe2-7c6f-4c39-b4a1-0c5d2ec54f6e" />



---

## HTTP Form Data Analysis

The HTTP POST request was examined further by expanding the form data in Wireshark.

The form fields included:

| Field | Value |
| --- | --- |
| `to` | `Lilytuckrige@yahoo.com` |
| `subject` | `you can't find us` |
| `message` | `and you can't hide from us. Stop teaching. start running.` |
| `type` | `0` |
| `ttl` | `30` |

The contents of these fields are important because they show that the packet was not simply a normal visit to the website. It contained data submitted to the service.

**Figure 8: HTTP form fields extracted from Frame 83601**

<img width="1081" height="957" alt="Fig 14" src="https://github.com/user-attachments/assets/9146dfc8-816f-45b2-9692-346ae5283354" />



---

## Finding 5 – Second Anonymous Email Submission

The investigation also identified earlier HTTP POST traffic to:

`www.sendanonymousemail.net/send.php`

The POST request contained information including:

**Recipient:**

`Lilytuckrige@yahoo.com`

**Sender:**

`the_whole_world_is_watching@nitroba.org`

**Subject:**

`Your class stinks`

The message content also contained negative comments about the class and teaching.

This traffic was important because it showed that the same client was also using another anonymous email service to send a message to the same recipient.

**Figure 9: HTTP POST to sendanonymousemail.net containing an earlier message to Lily Tuckrige**

<img width="1081" height="957" alt="Fig 14" src="https://github.com/user-attachments/assets/4ee3a509-eb1a-4473-ac86-bd2aad378dd8" />



---

## Finding 6 – Associating the Device With an Email Account

The next stage was to determine whether the device could be associated with a particular user.

Traffic from the same client was examined for browser cookies.

One HTTP request to Gmail contained the following cookie information:

`gmailchat=jcoachj@gmail.com/475099`

The request was generated by the same client IP:

`192.168.15.4`

and the same MAC address:

`00:17:f2:e2:c0:ce`

The browser user-agent was also consistent with the other traffic:

```text
Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)
```

This provided evidence associating the device with the Gmail account:

`jcoachj@gmail.com`

**Figure 10: Gmail cookie showing the account `jcoachj@gmail.com` on the investigated device**

<img width="1437" height="1162" alt="Fig 25" src="https://github.com/user-attachments/assets/b28de711-b6d7-431d-b439-90308deda68b" />



---

## Finding 7 – Chemistry 109 Roster Comparison

The account found in the network traffic was:

`jcoachj@gmail.com`

The account is consistent with the name:

**Johnny Coach**

Johnny Coach appears on the Chemistry 109 roster provided with the case materials.

Therefore, the available evidence gives the following connection:

```text
192.168.15.4
        ↓
00:17:f2:e2:c0:ce
        ↓
Browser traffic
        ↓
jcoachj@gmail.com
        ↓
Johnny Coach
        ↓
Chemistry 109 roster
```

This does not mean that the IP address itself proves that Johnny Coach sent the message. The association comes from combining the network address, MAC address, browser information and account information.

**Figure 11: Chemistry 109 roster showing Johnny Coach**

<img width="1197" height="602" alt="Screenshot 2026-10-05 120417" src="https://github.com/user-attachments/assets/09e3ca98-5f12-4957-8b29-6e294e219fff" />


---

## Timeline of Relevant Activity

The important activity in the packet capture can be arranged in the following order.

| Evidence | Activity | Significance |
| --- | --- | --- |
| HTTP GET requests | Client accessed `willselfdestruct.com` | Shows the client visited the anonymous email service |
| Frame 83037 | Client requested content from `willselfdestruct.com` | Confirms the client and MAC address involved |
| Frame 83601 | HTTP POST to `/secure/submit` | Contains the harassment message |
| Frame 83604 | HTTP response | Response to the message submission |
| Gmail traffic | Cookie containing `jcoachj@gmail.com` | Associates the device with the Gmail account |

The relevant harassment submission was contained in Frame 83601.

The packet capture timestamps should be reported exactly as displayed in Wireshark. Where a UTC conversion is required, the conversion should be stated clearly rather than changing the original timestamp without explanation.

**Figure 12: Wireshark timeline showing the relevant HTTP activity surrounding the harassment submission**

<img width="1886" height="966" alt="Fig 24" src="https://github.com/user-attachments/assets/babdfbd6-2334-4a52-8733-21936051df1f" />


---

## Evidence Log

| Evidence ID | Evidence / Packet | Finding | Why it matters | Screenshot |
| --- | --- | --- | --- | --- |
| E01 | `nitroba.pcap` | Original evidence file | Establishes the source evidence | Figure 1 |
| E02 | SHA-256 comparison | Original and working copy have matching hashes | Shows integrity of the working copy | Figure 2 |
| E03 | HTTP traffic to `69.25.94.22` | Client `192.168.15.4` accessed the service | Identifies the client system | Figure 4 |
| E04 | Ethernet details | MAC `00:17:f2:e2:c0:ce` | Identifies the device associated with the traffic | Figure 5 |
| E05 | Frame 83601 | POST containing the harassment message | Directly connects the client traffic to the message | Figure 7 |
| E06 | HTTP form data | Recipient, subject and message recovered | Shows the contents submitted to the service | Figure 8 |
| E07 | Earlier anonymous email POST | Message sent to Lily Tuckrige | Shows related activity from the same client | Figure 9 |
| E08 | Gmail HTTP cookie | `jcoachj@gmail.com` | Associates the device with an account | Figure 10 |
| E09 | Chemistry 109 roster | Johnny Coach listed | Supports the account-to-roster comparison | Figure 11 |
| E10 | Packet sequence | Relevant activity around Frame 83601 | Establishes the order of events | Figure 12 |

---

## Attribution Assessment

The evidence can be divided into three levels.

### Direct Evidence

The direct evidence shows that:

- `192.168.15.4` communicated with `69.25.94.22`.
- The device MAC address was `00:17:f2:e2:c0:ce`.
- The client submitted an HTTP POST to `/secure/submit`.
- The POST contained the recipient `Lilytuckrige@yahoo.com`.
- The subject was `you can't find us`.
- The message contained the words about not being able to hide, stopping teaching and starting to run.
- The same device traffic contained a Gmail cookie showing `jcoachj@gmail.com`.

### Reasonable Inference

From the evidence, it is reasonable to associate the device with the Gmail account `jcoachj@gmail.com`.

The account name is consistent with Johnny Coach, who appears on the Chemistry 109 roster.

This makes Johnny Coach the main person associated with the device in the available evidence.

### What the Evidence Does Not Prove

The evidence does not prove with absolute certainty that Johnny Coach was physically sitting at the computer and typing the message at the exact moment it was sent.

The open wireless network is an important limitation. Another person could potentially have used the same network or device.

For this reason, the report should not state that the IP address alone proves the identity of the sender.

---

## Limitations of the Investigation

### Shared Open Wireless Network

The residence used an open wireless network. This means that other people may have been able to connect to the network.

Therefore, the IP address `192.168.15.4` should not be treated as proof of a person's identity.

### Device Does Not Equal Person

The MAC address `00:17:f2:e2:c0:ce` identifies the network interface associated with the traffic. It does not prove who was physically using the device.

### Browser Account Information

The Gmail cookie provides useful evidence connecting the device to `jcoachj@gmail.com`, but an account being present on a computer does not by itself prove who was using the computer at that particular time.

### Packet Capture Limitations

The investigation is based on the traffic that was captured. Any activity that was not recorded in the PCAP cannot be examined from this evidence.

---

## Conclusion

The investigation of `nitroba.pcap` identified network traffic from client IP address `192.168.15.4` to the web service `www.willselfdestruct.com`, whose service IP in the relevant traffic was `69.25.94.22`.

The traffic was associated with MAC address `00:17:f2:e2:c0:ce`.

The strongest evidence was Frame 83601. This packet contained an HTTP POST request to `/secure/submit` and included the recipient `Lilytuckrige@yahoo.com`, the subject **"you can't find us"**, and the message telling the recipient to stop teaching and start running.

The same device was also associated with browser traffic containing the Gmail cookie:

`gmailchat=jcoachj@gmail.com`

The account corresponds to Johnny Coach, who is listed on the Chemistry 109 roster.

Based on these findings, the evidence strongly supports that the device associated with `192.168.15.4` and MAC address `00:17:f2:e2:c0:ce` was used to submit the harassment message and that the device was associated with the Gmail account `jcoachj@gmail.com`, corresponding to Johnny Coach.

However, I would not describe the evidence as absolute proof that Johnny Coach personally operated the device when the message was sent. The open wireless network and the difference between identifying a device and identifying a person are important limitations.

Overall, the evidence provides a strong connection between the harassment activity, the investigated device, the `jcoachj@gmail.com` account and Johnny Coach, but the final attribution should be stated with the limitations described above.

---

## Recommendation

For stronger attribution, additional evidence could be examined if it were available. This could include:

- Wireless access-point association records.
- DHCP records.
- Authentication records.
- Physical access records for the residence.
- Examination of the computer itself.
- Browser history and saved account information.
- Additional network captures from the same period.

These additional sources could help establish who was actually using the device at the time of the message.

---

## Screenshot Index

The following screenshots should be included in the final report or submitted in a separate evidence appendix.

| Figure | Screenshot description |
| --- | --- |
| Figure 1 | Evidence file and file size |
| Figure 2 | SHA-256 comparison of original and working copy |
| Figure 3 | Working PCAP opened in Wireshark |
| Figure 4 | HTTP requests to willselfdestruct.com |
| Figure 5 | Client IP and MAC address |
| Figure 6 | Secure anonymous email service page content |
| Figure 7 | Frame 83601 containing the harassment POST |
| Figure 8 | Form fields containing recipient, subject and message |
| Figure 9 | Earlier anonymous email POST |
| Figure 10 | Gmail cookie containing `jcoachj@gmail.com` |
| Figure 11 | Chemistry 109 roster showing Johnny Coach |
| Figure 12 | Timeline of relevant packet activity |

### Evidence Folder

Place your screenshots in the `evidence/` folder using the filenames below so the links in this README resolve correctly:

```text
evidence/
├── figure-01-evidence-file.png
├── figure-02-sha256-verification.png
├── figure-03-working-pcap-wireshark.png
├── figure-04-willselfdestruct-http.png
├── figure-05-client-ip-mac.png
├── figure-06-anonymous-email-service.png
├── figure-07-frame-83601.png
├── figure-08-http-form-data.png
├── figure-09-earlier-anonymous-email.png
├── figure-10-gmail-cookie.png
├── figure-11-chem109-roster.png
└── figure-12-packet-timeline.png
```

---

## Repository Structure

```text
SBT-DF204-Case-Study-01/
│
├── README.md
│
├── evidence/
│   ├── nitroba.pcap
│
├── screenshots/
│   ├── figure-01-evidence-file.png
│   ├── figure-02-sha256-verification.png
│   ├── figure-03-working-pcap-wireshark.png
│   ├── figure-04-willselfdestruct-http.png
│   ├── figure-05-client-ip-mac.png
│   ├── figure-06-anonymous-email-service.png
│   ├── figure-07-frame-83601.png
│   ├── figure-08-http-form-data.png
│   ├── figure-09-earlier-anonymous-email.png
│   ├── figure-10-gmail-cookie.png
│   ├── figure-11-chem109-roster.png
│   └── figure-12-packet-timeline.png
│
├── report/
│   └── file_compared_sha256  
│   └── file_info.txt
│   └── nitroba_pcap_sha256  
│   └── nitroba_pcap_stat
│
└── working/
    └── client_system.pcap
    └── nitroba_working.pcap  
    
```

> **Note:** Do not upload passwords, private credentials, API keys, or other sensitive information to a public repository.

---

## Final Statement

This investigation was carried out using the supplied PCAP evidence and Wireshark. The original evidence was preserved, a working copy was created and the hash values were compared before analysis.

The packet capture provided enough information to trace the harassment message to a particular device and to associate that device with the account `jcoachj@gmail.com`. The account corresponds to Johnny Coach on the Chemistry 109 roster.

The evidence therefore provides a strong association between Johnny Coach and the device used for the harassment activity, while the open wireless network remains a limitation when making a final statement about the person who physically sent the message.

---

## Author  

**Adekunle Ogunyemi**  
**SBT-DF204 – Computer Forensics Case Studies**  
**Case Study 1 – Investigating Harassment Email Traffic With Wireshark**
