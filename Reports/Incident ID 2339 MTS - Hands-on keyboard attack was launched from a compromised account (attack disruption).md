# Incident ID 2339 Hands-on keyboard attack was launched from a compromised account (attack disruption)

## Findings

### Files:

`C:\ProgramData\AnyDesk `
- SHA256: e6b3182a15c35ab18d8f8e8fa79a8fa069588414b826280df9cb1b733ed971cb

`C:\Users\administrator\Pictures\Advanced_Port_Scanner_2.5.3869.exe` 
- SHA256: 8b9c7d2554fe315199fae656448dc193accbec162d4afff3f204ce2346507a8a

`c:\users\administrator\appdata\local\temp\7\is-rjidg.tmp\advanced_port_scanner_2.5.3869.tmp`
- SHA256: 37e12903cc165abc6e0629195e0eb5a6318c92614952e055cf3444ad35ed4f0f

`C:\Users\administrator\Pictures\64\dump.bat`


`C:\Users\administrator\Pictures\64\64.exe `
- SHA256: 6688fb3039ad6df606d76a897ef1072cdc78b928335c6bfa691d99498caf5c4b

`C:\Users\administrator\Pictures\64\64_log.txt`

`C:\Users\administrator\Desktop\checker\checker (222).exe`
- SHA256: d00f7cf6af68ba832b9d364f28411346cfe66fd3b1f5bcac318766add29ff7f0

### IP Addresses 

	- 45[.]227[.]254[.]154 
	- 66[.]132[.]224[.]236
	- 45[.]147[.]48[.]158 
    - 104[.]143[.]2[.]195


### URLs

- HXXPS://www[.]md5[.]in
- HXXPS://hashes[.]com

### Investigation Summary

Who 
	
- The administrator account on mts-dc.mts.local. This account was used to authenticate via NTLM from the threat actor's IP (45[.]227[.]254[.]154) and was the context under which all subsequent malicious activity was carried out, including the registry modifications, discovery commands, AnyDesk installation, Advanced Port Scanner execution, Mimikatz credential dumping via dump.bat, and the execution of checker (222).exe. Based on DeviceLogonEvents for the preceding 90 days, there is no record of prior authentication attempts from this IP against the administrator account on any device in the mts domain, indicating the threat actor already possessed valid credentials prior to first contact. As the compromised account is a local administrator account on the domain controller rather than a mailbox-enabled user account, email-based phishing of this specific credential is unlikely to be the direct vector. The credential was more likely obtained through reuse of a previously leaked/exposed password
	
- A local account named Guest was created on mts-dc.mts.local and granted administrator rights at approximately 06:35 UTC on 2026-07-02 (Figure 7). No sign-in attempts using this account have been observed to date (Figure 8), indicating it was created as a persistence mechanism rather than used for active operations during this incident.

What 

2026-07-02 06:33 [UTC] XDR had generated an Incident for Hands-on keyboard attack was launched from a compromised account (attack disruption). Prior to this a successful network logon via NTLM was made sourcing from 45[.]227[.]254[.]154 (remote device name - b_114) (Figure 1). Reviewing DeviceLogonEvents for the preceding 90 days, there is no record of prior authentication attempts - successful or failed - from 45[.]227[.]254[.]154 against the administrator account on any device of the mts domain. The first recorded event from this IP is the successful logon itself, indicating this was not preceded by a brute-force campaign against this account and this device, and is more consistent with the attacker already possessing valid credentials prior to first contact (e.g., obtained via phishing, credential reuse). AbuseIPDB has reported this IP Address with 100% confidence with the IP Address still being active till this day (figure 3).
	
2026-07-02 06:34 [UTC] - Remote execution from the device B_114 was observed using Explorer.exe to execute Powershell.exe remotely and then a series of registry commands being executed (Figure 4). Based on Figure 4 we notice a few security weaking mechanisms at play. First of all the last user which logged into the affected asset won't show (DontDisplayLastUsername = 1). Furthermore, (fDenyTSConnections = 0) will allow remote connections to this computer and (UserAuthentication = 0) will disable NLA (network level Autehntication) which will make RDP connections to this device much easier. Another important finding is (WDigest) being enabled which means harvesting credentials will be easier and credentials will be stored in lsass in plain text. A search of DeviceFileEvents scoped to mts-dc.mts.local for .ps1 files confirms no attacker-authored script file was used to deliver the reg.exe registry modification sequence observed at 06:34 UTC. The only .ps1 artifacts present during this window are PowerShell Script Policy Test files (_PSScriptPolicyTest*.ps1), a benign, automatically-generated artifact of PowerShell's execution policy evaluation. This supports the assessment that the registry commands were executed via direct/pasted input to an interactive PowerShell session rather than a saved script (Figure 5). 

- At the same time, the powershell.exe process was observed executing multiple discovery commands. ARP and WMIC commands were used for reconnaissance — ARP.EXE -a was used to view devices this host had previously communicated with, while wmic computersystem get domain was used to obtain the Active Directory domain the host is joined to.

- Furthermore a file named "Anydesk.exe" had been installed silently and then started as a service.  Virus Total have flagged this file as safe with no vendors flagging this as malicious (Figure 6). However because this is anydesk, this could allow the threat actor to maintain persistence and if this isn't a standard application used in the business, it should be removed immediately. 

- A minute later a new user named "Guest" had been created and was given administrator rights (Figure 7). At the time of investigation there has been no attempts of sign ins using this account (Figure 8), however this should be removed due to the threat actor maintaining persistent access to the DC. 
	

2026-07-03 01:16 [UTC] An executable named "Advanced_Port_Scanner_2.5.3869.exe" by the account administrator (Figure 9). Reviewing the SHA256 hash on Virus Total, 11 vendors have flagged this file as malicious (Figure 10). As per Acronis Threat Intelligence reporting Advanced Port Scanner is frequently used by threat actors for network enumeration to locate other hosts and services to help with Lateral Movement [1]. Reviewing network telemetry, we have observed the executable attempting to establish multiple across multiple IPs and multiple ports. This is also known as a port sweep and is a form of reconnaissance which will allow the attacker to learn about the environment (Figure 11). The attacker was able to learn the ports opened showing in figure 12.
	
- Seconds Later a script named "dump.bat" had been executed via cmd which had led to the executable "64.exe" and "86.exe" being ran by the administrator (Figure 13). VirusTotal has flagged this executable as being associated with Mimikatz (Figure 14). Mimikatz is a well-known credential dumping tool that allows an attacker to extract credentials from the Local Security Authority Subsystem Service (LSASS) - the process responsible for handling authentication on Windows as well as from the local SAM database. Depending on system configuration, this can include plaintext passwords. As this is on the domain controller the threat actor may have dumped highly sensitive credentials which can enable them to laterally move through the environment. 

2026-07-03 01:20 [UTC] An executable named "checker (222).exe" is executed via explorer.exe by the administrator. Virus total has flagged this as malicious  (58 Vendors flagging as malicious) and this executable also being linked to Mimikatz (Figure 15). Further research suggest that this executable drops multiple tools including a powershell script, system drivers, DLLs and more (Figure 16). Based on the available telemetry we can see that the executable "checker (222).exe" had dropped similar files (Figure 17).

-  Looking at the available network telemetry, checker (222).exe established a connection to the IPv4 address 45[.]147[.]48[.]158 over port 80 (HTTP) to the site hxxtp://www[.]md5[.]in (Figure 18). This IP is associated with xTom GmbH, a datacenter/VPS hosting provider based in Frankfurt am Main, Germany. There are no reports for this IP address on AbuseIPDB (Figure 19), nor any information found on VirusTotal - this is not necessarily reassuring, as datacenter/VPS infrastructure is commonly used for disposable attacker infrastructure that may simply not have accumulated abuse reports yet.
	
- Furthermore, the executable made a connection to the IP address 104[.]143[.]2[.]195 over HTTPS (port 443), reaching out to hxxps://hashes[.]com (Figure 18). Reviewing this on AbuseIPDB, it has been reported 19 times with an abuse confidence of 0%. This IP address is associated with the United States and linked to GameServerKings[.]com, a game server hosting site. Given the timing of checker (222).exe's execution and these outbound connections, it is likely this process reached out to these domains to retrieve additional hosted files (Figure 17)."

-  Based on our Telemetry, it is observed that from 2026-07-03 to 2026-07-23 the only files executed from the "checker (222).exe" Directory (where the dropped payloads where stored) was mimikatz.exe (figure 20). 

2026-07-24 17:20 [UTC] A check was done to see  if any other artefacts where observed in the environment from 90 days to the time of investigation. At the time of investigation and based on the telemetry provided the IOCs found in the finding section of this report where not observed on other devices apart form mts-dc.mts.local (Figure 21).



When 

The intrusion began 2026-07-02 06:33 UTC with the initial NTLM logon from 45.227.254.154, and hands-on activity continued through 2026-07-03 (registry changes, AnyDesk install, Guest account creation, port scanning, and credential dumping via checker (222).exe/mimikatz). A follow-up hunting check on 2026-07-24 17:20 UTC found no renewed activity — only mimikatz.exe had run from the checker (222).exe directory between 2026-07-03 and 2026-07-23, and no IOCs were found on any other device in the prior 90 days. All times UTC. There is no evidence the activity is still ongoing at the time of investigation.



Where 
All confirmed malicious activity was limited to a single host, mts-dc.mts.local (192.168.10.8), the domain controller. The attacker connected remotely from external IP 45.227.254.154 (device name B_114, geolocated to Belize). A 90-day sweep of the rest of the environment found no related files, processes, or network IOCs on any other device (Figure 21), indicating the activity did not spread beyond the domain controller.

Why 
Telemetry shows the attacker authenticated directly with valid administrator credentials via NTLM, with no record of prior brute-force or failed logon attempts from this IP against this account - indicating the credentials were already compromised prior to first contact rather than obtained via brute force (Figure 2, Figure 3). The exact method by which the credentials were originally obtained could not be determined from the available telemetry.



### Recomendations 

### Immediate actions

- Given mts-dc.mts.local's role as a domain controller, full network isolation may cause significant operational disruption if this is the sole or a critical DC in the environment. It is recommended to first confirm DC redundancy, then apply the most surgical containment feasible — such as blocking external/internet-facing access and the specific malicious IPs identified, or using EDR-based device containment — while prioritizing credential rotation and persistence removal as the immediate risk-reduction steps. Full isolation should be evaluated in coordination with IT leadership if a higher-severity ongoing threat is confirmed
  
- The threat actor enabled RDP (fDenyTSConnections = 0) and disabled NLA (UserAuthentication = 0) as part of their post-compromise activity, likely to establish a more convenient remote access channel for themselves going forward. It is recommended that RDP be disabled on mts-dc.mts.local unless explicitly required, and that NLA be enforced if RDP access is necessary.

- Assume domain-wide credential compromise and rotate the krbtgt account password (twice, per Microsoft's guidance), given WDigest was enabled and a full LSASS dump occurred on a domain controller — any domain account with a cached session at the time of the dump should be considered exposed, not just the local admin account itself.

- Identify and reset credentials for any accounts with active/cached sessions on mts-dc.mts.local at the time of the Mimikatz execution (2026-07-03 01:16 UTC), particularly any domain-privileged accounts.

- Uninstall AnyDesk and remove its associated Windows service (HKLM\SYSTEM\ControlSet001\Services\AnyDesk); confirm whether AnyDesk is an approved/standard tool in this environment - if not, treat its presence as unauthorized remote access tooling and investigate how it was obtained.

- Block the known malicious indicators at the firewall/proxy level: 45[.]227[.]254[.]154, 45[.]147[.]48[.]158, 104[.]143[.]2[.]195, and the associated domains (md5[.]in, hashes[.]com) is applicable.

- Deploy LAPS (Local Administrator Password Solution) across the environment to ensure local administrator passwords are unique per machine and automatically rotated, reducing the impact of any future local credential exposure.

- Hunt for delayed/second-wave lateral movement: the toolkit dropped by checker (222).exe (Invoke-TheHash, Invoke-SMBExec, Invoke-WMIExec, PsExec64) was staged but not confirmed executed. Continue monitoring for SMB/WMI-based authentication anomalies across the environment rather than treating the 90-day sweep as a final closure.

- Continue monitoring source IP 45[.]227[.]254[.]154, as it remains active per AbuseIPDB as of 2026-07-22.
