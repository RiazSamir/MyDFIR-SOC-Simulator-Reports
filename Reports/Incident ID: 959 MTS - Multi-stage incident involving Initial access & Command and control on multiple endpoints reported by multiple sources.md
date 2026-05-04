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



### When
 
| Time (UTC) | Event |
|---|---|
| 2025-12-23 15:18 | Anonymous IP sign-in alert — Zach Balrog logs into Azure Portal via `185[.]98[.]171[.]250` |
| 2025-12-23 15:21 | Successful RDP onto Desktop-1 as soc-administrator via `185[.]98[.]171[.]250` |
| 2025-12-23 15:28 | Successful SSH onto mts-web as root via `185[.]98[.]171[.]249` |
| 2025-12-23 15:35 | Reconnaissance commands executed on mts-web; newuser created; bin.sh downloaded via wget |
| 2025-12-23 15:38 | BEC mailbox forwarding rule created on Zach Balrog's OWA account |
| 2025-12-27 19:30 | Time of investigation — no further activity observed from either IP |
 
At the time of investigation, no further activity was observed from `185[.]98[.]171[.]249` or `185[.]98[.]171[.]250`. However, due to the confirmed compromise of mts-web and the full BEC compromise of Zach Balrog's Outlook account, the threat actor retains the capability to re-establish access at any time.


### Where
 
This incident affected multiple platforms and hosts:
 
- **Azure Portal** — Unauthorised sign-in using Zach Balrog's credentials
- **Desktop-1** — Unauthorised RDP session as soc-administrator
- **mts-web** — Unauthorised SSH session as root; malware download; persistence mechanism created
- **Outlook (OWA)** — BEC compromise; malicious forwarding rule created

### Why
 
Valid user credentials were compromised prior to this incident. The threat actor accessed Azure and multiple hosts without any brute force activity, indicating credential stuffing, password reuse, or prior credential theft. MFA was either not enabled or not enforced, allowing for full account takeover without triggering additional verification.
 
The use of Proton VPN (IP addresses `185[.]98[.]171[.]249` and `185[.]98[.]171[.]250`) allowed the threat actor to mask their true origin and bypass geo-based detection controls. Both IP addresses belong to the same VPN provider and fall within the same subnet range. Given the timing, shared provider, and coordinated malicious actions across both addresses, it is assessed with high confidence that both IPs were controlled by the same threat actor.


## Recommendations
 
### Immediate
 
- Reset credentials for all compromised accounts:
  - **Zach Balrog** (`zbalrog`)
  - **soc-administrator** on Desktop-1
  - **root** on mts-web
  - Consider enforcing the use of a password manager to generate strong passphrases, reducing the risk of credential stuffing and password reuse attacks.
- Review and remove the malicious mailbox forwarding rules on Zach Balrog's OWA account. Block the email address `Dawis19729[AT]nctime[.]com` where applicable.
- Remove the "newuser" account created on mts-web with sudo privileges, as this is assessed to be a persistence mechanism left by the threat actor to maintain access.
- Review and remove `bin.sh` located at `/root/bin.sh` on mts-web. Prior to deletion, the file should be forensically reviewed to determine whether it established any additional persistence mechanisms or staged further payloads — even though no execution was observed, dormant artifacts should not be assumed benign.
- Conduct a full malware scan on both mts-web and Desktop-1 for any additional malicious artifacts or tools left by the threat actor.
### Short-Term (24–48 Hours)
 
- Enforce MFA on all accounts where possible, with priority on privileged accounts and any accounts with access to cloud resources, to prevent unauthorised access via compromised credentials.
- Block access to the malicious URL `hxxp://182[.]121[.]245[.]103:46094/bin.sh`. Establish monitoring and alerting for suspicious use of wget and curl to download files from external IP addresses, particularly into privileged directories such as `/root/`.
- SSH and RDP should not be publicly accessible. Restrict both services to internal IP addresses only, or enforce access via a VPN. Exposing remote access services to the internet, when combined with stolen credentials and no MFA, provides trivial initial access for threat actors.
### Long-Term (1–4 Weeks)
 
- Review and audit account privileges across cloud and on-premises resources. Remove or disable accounts that are no longer required.
- Implement alerting for Anonymous IP Sign-Ins and sign-ins from unfamiliar geographies in Azure AD/Entra ID — this incident was detected via such an alert, and tuning these detections will improve future response times.
- Implement Conditional Access Policies in Azure to restrict access based on compliant devices, trusted locations, or named IP ranges.
- Audit SSH configurations on Linux hosts — root login over SSH should be disabled. Use `PermitRootLogin no` in `/etc/ssh/sshd_config` and enforce key-based authentication over password-based authentication.

## Screenshots
<p align="center">
  <img width="644" height="483" alt="image" src="https://github.com/user-attachments/assets/92772fbd-31bd-461c-b0c4-0b698d4c00d4" />
</p>
<p align="center"><b>Figure 1: WHOIS lookup for IP 185.98.171.250 confirming attribution to Proton VPN, Canada (Toronto), registered under ASN AS212238 CDNEXT Datacamp Limited.</b></p>
