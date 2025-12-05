# Splunk Zeek Connection Log Analysis

## Objective
This project demonstrates how to ingest and analyze Zeek connection logs using Splunk.  
The goals of this lab were to:

- Upload and search Zeek connection logs.
- Identify top clients and servers.
- Determine the most common services used in the network.
- Detect long-running connections and potentially suspicious traffic.

---

## Log Source
**Zeek Conn Log File:**  
[Download zeek_conn_logs.json](https://raw.githubusercontent.com/0xrajneesh/30-Days-SOC-Challenge-Beginner/refs/heads/main/zeek_conn_logs.json)

- Saved as: `zeek_conn_logs.json`  
- Uploaded into Splunk  
- **Index:** `main`  
- **Source type:** `json`  
Initial test search:
source="zeek_conn_logs.json" host="alphin-VirtualBox" index="main" sourcetype="_json"
<img width="949" height="706" alt="Screenshot From 2025-12-05 02-07-10" src="https://github.com/user-attachments/assets/bb2621f2-7314-41d7-b62f-6da11390501b" />


## Action 1 - Top 10 Client IPs (id.orig_h)

index="main" sourcetype="_json"
| stats count by id.orig_h
| sort -count
| head 10

<img width="1790" height="471" alt="Screenshot From 2025-12-05 02-14-08" src="https://github.com/user-attachments/assets/d0cf1d59-dd95-4c1c-bca6-7dbdd4252f0b" />

This identifies which client IPs are generating the most outbound connections.
High-volume clients may indicate-

  Compromised hosts

  Malware beaconing

  Automated scanning tools

   Misconfigured systems

## Action 2 - Most Common Services

index="main" sourcetype="_json"
| stats count by service
| sort -count

<img width="1790" height="471" alt="Screenshot From 2025-12-05 02-20-04" src="https://github.com/user-attachments/assets/17876fbf-3144-4f7b-abc4-4f0fe7f6c162" />


This reveals which network services are most used.
High traffic to internal servers or uncommon services may indicate-

  Lateral movement

  Unauthorized access

   Misconfigured applications

## Action 3 - Connections Lasting More Than 1 Second

index="main" sourcetype="_json" duration>1
| table ts id.orig_h id.resp_h service duration
| sort -duration

<img width="1788" height="754" alt="Screenshot From 2025-12-05 02-26-41" src="https://github.com/user-attachments/assets/e0f73eda-6d28-4c93-b24b-cbc5ecde42b5" />

Why Duration Matters-

  Most scanning and reconnaissance tools (e.g., Nmap) generate very short connections (<1 second).

   Filtering for connections >1 second helps remove noise and highlight:

    Potential exfiltration

    Long-running data transfers

    Active remote sessions

    Non-secure services being used

### Security Note

The logs show several insecure services such as-

    HTTP

    IMAP

    SMTP

    FTP

    DHCP

A recommendation is to replace them with secure equivalents:
HTTPS, IMAPS, SMTPS, STARTTLS, etc., and harden DHCP to reduce MITM/ On The Path Attack risks.

## Action 4 - Most Accessed Internal Servers (id.resp_h)

index="main" sourcetype="_json"
| stats count by id.resp_h
| sort -count
| head 10

<img width="1791" height="508" alt="Screenshot From 2025-12-05 02-43-52" src="https://github.com/user-attachments/assets/51c4a6ee-4ec0-498d-91cb-722600302db1" />

This identifies which servers are receiving the most connections.
Useful for detecting:

    High-value targets

    C2 traffic

    Potential brute force attacks

    Misconfigured internal services

   ---

## Summary of Findings

After analyzing the Zeek connection logs in Splunk, the following insights were gathered:

    Several client IPs generated a number of outbound connections but nothing out of the oridinary.

    Numerous long-duration connections (>1 sec) could indicate active sessions or potential data transfer.

    Several insecure services were detected and should be upgraded to their secure versions.

 Create Splunk alerts e.g:

    High connection counts

    Long-duration sessions

    New or unusual services
