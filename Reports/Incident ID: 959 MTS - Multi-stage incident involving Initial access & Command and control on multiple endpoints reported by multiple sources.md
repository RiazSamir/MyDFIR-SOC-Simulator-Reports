# Incident ID: 959- MTS - Multi-stage incident involving Initial access & Command and control on multiple endpoints reported by multiple sources

### IOC IP Addresses:
 
- `185[.]98[.]171[.]250` — Proton VPN, Canada — Logged into Azure Portal using Zach Balrog's account and RDP onto Desktop-1 as soc-administrator
- `185[.]98[.]171[.]249` — Proton VPN, Logged into Root on mts-web as well as Zach Balrogs Outlook Web Access (OWA) 
- `182[.]121[.]245[.]103` — Outbound wget request for bin.sh

### IOC File:
 
- `bin.sh`
  - SHA256: `4293c1d8574dc87c58360d6bac3daa182f64f7785c9d41da5e0741d2b1817fc7`
 
### IOC Email Address:
 
- `Dawis19729[AT]nctime[.]com`

### IOC URLs:
- `hxxp://182[.]121[.]245[.]103:46094/bin.sh`

## Investigation Summary
 
This incident involved unauthorised access into cloud and on-premises resources through compromised credentials, using IP addresses associated with Proton VPN (`185[.]98[.]171[.]250` and `185[.]98[.]171[.]249`). The threat actor accessed the Azure Portal and Outlook Web Access (OWA) using the credentials of Zach Balrog, resulting in a Business Email Compromise (BEC). A forwarding rule was created to exfiltrate sensitive emails to the external address `Dawis19729[AT]nctime[.]com`.
 
The threat actor accessed Desktop-1 using the account soc-administrator over RDP; however, no malicious activity was observed on this host at the time of investigation. The threat actor subsequently accessed mts-web using the root account over SSH. Discovery commands were executed and a search for files named "passwords" was performed. A malicious shell script (`bin.sh`) was downloaded via wget and was granted execute permissions via chmod. At the time of investigation, no execution of the script was observed; however, the file is associated with the Mozi malware family, linked to botnet activity. A persistence mechanism was also identified — a new user account named "newuser" was created on mts-web with sudo privileges.


## WHO, WHAT, WHEN, WHERE and WHY

### Who
 
An unknown external threat actor accessed multiple accounts and systems using Proton VPN IP addresses `185[.]98[.]171[.]249` and `185[.]98[.]171[.]250`. Impacted identities included:
 
- **Zach Balrog** — Azure Portal and OWA access
- **soc-administrator** — RDP access onto Desktop-1
- **root** — SSH access onto mts-web


### What
 
- **2025-12-23 15:18 PM (UTC)** — An alert was generated for an Anonymous IP Address Sign-In associated with the user Zach Balrog, sourcing from IP `185[.]98[.]171[.]250`. During this time, Zach Balrog's account was used to sign into the Azure Portal. No brute force activity was observed prior to the successful logon, indicating credential stuffing or prior credential theft. The IP address is associated with Proton VPN, suggesting the use of a VPN or proxy to mask the attacker's true origin *(Figure 1)*.
- **2025-12-23 15:21 PM (UTC)** — A successful RDP session was initiated from the same IP address `185[.]98[.]171[.]250` onto the device "Desktop-1" using the account "soc-administrator". No malicious activity was observed on Desktop-1 at the time of investigation *(Figure 2)*.
- **2025-12-23 15:28 PM (UTC)** — A related IP address `185[.]98[.]171[.]249`, also associated with Proton VPN and sourcing from Canada, successfully SSH'd onto the device "mts-web" using the root account *(Figure 3)*. On **2025-12-23 15:35 PM (UTC)**, the root account was observed performing reconnaissance activity, including reviewing OS information, reviewing auth.log, listing running processes, and searching for files named "passwords". A new user account named "newuser" was created with sudo privileges. At the time of investigation, this account had not been accessed *(Figure 5)*.
- **2025-12-23 15:35 PM (UTC)** — A wget request was made from mts-web to IP `182[.]121[.]245[.]103` over port 46094, retrieving a shell script named `bin.sh`, stored at `/root/bin.sh`. A VirusTotal lookup returned 47 vendor detections, classifying the file under the **Mozi** malware family — a botnet capable of providing an attacker with remote control of an infected device. The file was granted execute permissions via chmod; however, no execution of the script was observed at the time of investigation based on available evidence. The file was not observed on any other device *(Figure 6)*.
- **2025-12-23 15:38 PM (UTC)** — A `Set-Mailbox` rule was created with the action `DelivertoMailboxAndForward`, forwarding emails to the malicious address `Dawis19729[AT]nctime[.]com`, initiated from IP `185[.]98[.]171[.]249`. Shortly after, a mailbox rule named "zzzzzz" was created, configured to forward emails containing the keywords **passwords, password, sensitive, invoice, finance, and payroll** to the malicious address and move them to the "Deleted Items" folder. This constitutes a **Business Email Compromise (BEC)**.
