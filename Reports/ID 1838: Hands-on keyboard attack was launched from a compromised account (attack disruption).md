# ID 1838: Hands-on keyboard attack was launched from a compromised account (attack disruption)

## Findings
 
### IOC Files:
 
- `C:\Windows\ZVDEcBZe.exe`
  - SHA256: `3c2fe308c0a563e06263bbacf793bbe9b2259d795fcc36b953793a7e499e7f71`
- `C:\Windows\CruQgYgT.exe`
  - SHA256: `3c2fe308c0a563e06263bbacf793bbe9b2259d795fcc36b953793a7e499e7f71`
- `C:\Windows\Temp\upinstalled.exe`
  - SHA256: `bdbfa96d17c2f06f68b3bcc84568cf445915e194f130b0dc2411805cf889b6cc`
- `C:\Windows\Temp\ttt.exe`
  - SHA256: `B771267551961ce840a1fbcd65e8f5ecd0a21350387f35bbcd4c24125ec04530`
  - Dropped by `upinstalled.exe`. Copies itself to `C:\Windows\SysWOW64\wmiex.exe`
- `C:\Windows\Temp\svchost.exe`
  - SHA256: `60b6d7664598e6a988d9389e6359838be966dfa54859d5cb1453cbc9b126ed7d`
- `C:\Windows\Temp\tmp.vbs`
- `C:\Windows\Temp\m.ps1`
### IOC Registry Keys:
 
- `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Xpuq` — Auto-starts `ZVDEcBZe.exe`
- `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\WIgA` — Points to `CruQgYgT.exe`
### IOC IP Addresses:
 
- `95[.]189[.]49[.]66` — Russia
### IOC URLs:
 
- `beahh[.]com`
- `abbny[.]com`
- `haqo[.]net`
- `hxxp://v[.]beahh[.]com/v`
- `hxxp://oo[.]beahh[.]com`
### IOC Scheduled Tasks:
 
- `\Microsoft\windows\Bluetooths`

## Investigation Summary
 
This incident involved a multi-stage intrusion targeting the domain controller mts-dc.mts.local. The attack began with the disabling of Microsoft Real-Time Protection, followed by unauthorised access from a Russian IP address (`95[.]189[.]49[.]66`) via credential stuffing. The threat actor deployed multiple malicious executables including RemCom tools for lateral movement and remote code execution, a backdoor trojan (`ttt.exe` / `wmiex.exe`) communicating with attacker-controlled C2 infrastructure, and tooling associated with Impacket and Mimikatz for credential dumping via lsass.exe.
 
Persistence was established through two registered services, a scheduled task beaconing to attacker infrastructure every 50 minutes, and the creation of a backdoor account ("geshli") with administrator and remote desktop privileges. Lateral movement was observed from mts-dc to mts-contractorpc1 using the local administrator account. At the time of investigation, the threat actor may still retain access to the environment via the backdoor account and active C2 beaconing.


## WHO, WHAT, WHEN, WHERE and WHY
 
### Who
 
The threat actor is unknown; however, the attack originated from the IP address `95[.]189[.]49[.]66`, which resolves to Russia. Compromised accounts included the local administrator account on mts-dc and a backdoor account created under the name "geshli".


### What
 
- **2026-04-15 16:46 PM (UTC)** — `MsMpEng.exe` disabled Microsoft Real-Time Protection on mts-dc, launched as a service. This was likely the first action taken by the threat actor to weaken defences ahead of further malicious activity *(Figure 1)*.
- **2026-04-16 03:09 AM (UTC)** — A successful network logon was recorded on mts-dc sourcing from IP `95[.]189[.]49[.]66`, resolving to Russia *(Figure 2)*. An AbuseIPDB lookup returned a confidence of abuse of 7%, with the last report being one month prior for unauthorised access *(Figure 3)*. No brute force activity was observed prior to the successful logon, indicating credential stuffing or prior credential theft as the likely initial access vector.
- **2026-04-16 03:09 AM (UTC)** — The file `ZVDEcBZe.exe` was dropped under `C:\Windows` by the SYSTEM account. VirusTotal identifies this as **RemCom** — a tool used for remote process execution and lateral movement. One minute later, a service was registered under `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Xpuq` to auto-start this executable, establishing persistence for the RemCom tool.
- **2026-04-16 05:49 AM (UTC)** — A second file, `CruQgYgT.exe`, was created by the SYSTEM account via `ntoskrnl.exe` at `C:\Windows\`. VirusTotal identifies this as the same RemCom executable as `ZVDEcBZe.exe`. A corresponding service was registered under `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\WIgA`.
- **2026-04-16 06:09 AM (UTC)** — `upinstalled.exe` was created by the SYSTEM account via `ntoskrnl.exe` and flagged as malicious on VirusTotal *(Figure 4)*. This executable dropped `ttt.exe` *(Figure 5)* — a backdoor/trojan that copies itself to `C:\Windows\SysWOW64\wmiex.exe`, masquerading as a legitimate Windows binary *(Figure 6)*. According to VirusTotal, `wmiex.exe` communicates with `beahh[.]com`, `abbny[.]com`, and `haqo[.]net` to download payloads disguised as `.png` files. Multiple such `.png` files were observed in the logs, all flagged as malicious on VirusTotal and linked back to `wmiex.exe` *(Figure 14)*. Shortly after, `wmiex.exe` was observed establishing a connection to `oo[.]beahh[.]com` *(Figure 15)*. At the time of investigation, no other devices had made connections to this domain *(Figure 16)*.
- **2026-04-16 06:09 AM (UTC)** — `CruQgYgT.exe` executed `tmp.vbs` via `wscript.exe` *(Figure 7)*. The VBScript used Task Scheduler to create a scheduled task named `\Microsoft\windows\Bluetooths`, configured to execute as SYSTEM every 50 minutes. The task contained an encoded payload that, when decoded, uses `IEX` (Invoke-Expression) to reach out to `hxxp://v[.]beahh[.]com/v` and download a PowerShell script executed entirely in memory — fileless execution. VirusTotal flags this URL as malicious *(Figure 8)*. The scheduled task command also appended `&&c:\windows\temp\svchost.exe`, immediately executing the malicious binary upon task creation to ensure instant payload execution alongside future persistence. This `svchost.exe` is located in `C:\Windows\Temp` rather than the legitimate path `C:\Windows\System32`, indicating masquerading. VirusTotal flags this file as malicious, with community notes linking it to **Impacket** and **Mimikatz** *(Figure 9)*.
  - The malicious `svchost.exe` executed `m.ps1`, which VirusTotal flags as malicious and associates with **Mimikatz** — a credential dumping tool that abuses `lsass.exe` to extract credentials from memory *(Figure 9)*.
- **2026-04-16 06:13 AM (UTC)** — `m.ps1` performed a Windows API call to read the memory contents of `lsass.exe`, confirming that credential dumping occurred. This was executed under the SYSTEM account, providing the necessary privileges to access lsass *(Figure 10)*.
- **2026-04-17 12:27 AM (UTC)** — A successful network logon was recorded on mts-contractorpc1, originating from the private IP address of mts-dc using the local administrator account *(Figures 11 & 12)*. No brute force activity was observed prior to the successful logon. Given that credentials were dumped from mts-dc, it is likely the local administrator password was reused across machines, facilitating lateral movement without the need for brute force.
- **2026-04-17 16:38 PM (UTC)** — A backdoor account named "geshli" was created and added to both the Administrators group and Remote Desktop Users *(Figure 13)*. At the time of investigation (2026-04-17 18:38 PM UTC), no logon attempts had been observed for this account.

### When
 
| Time (UTC) | Event |
|---|---|
| 2026-04-15 16:46 | Microsoft Real-Time Protection disabled on mts-dc via MsMpEng.exe |
| 2026-04-16 03:09 | Successful network logon from `95[.]189[.]49[.]66` (Russia) onto mts-dc |
| 2026-04-16 03:09 | ZVDEcBZe.exe (RemCom) dropped; Xpuq service registered for persistence |
| 2026-04-16 05:49 | CruQgYgT.exe (RemCom) dropped; WIgA service registered for persistence |
| 2026-04-16 06:09 | upinstalled.exe dropped; ttt.exe dropped and copied to SysWOW64 as wmiex.exe |
| 2026-04-16 06:09 | tmp.vbs executed via wscript.exe; Bluetooths scheduled task created; svchost.exe executed |
| 2026-04-16 06:13 | m.ps1 performs lsass memory read — credential dumping confirmed |
| 2026-04-17 12:27 | Lateral movement — mts-dc logs into mts-contractorpc1 as local administrator |
| 2026-04-17 16:38 | Backdoor account "geshli" created with administrator and RDP privileges |
 
The attack began on 2026-04-15 at 16:46 UTC and activity was last observed on 2026-04-17 at 16:38 UTC. At the time of investigation, the threat actor may still retain access to the environment via the backdoor account "geshli" and the scheduled task beaconing to `hxxp://v[.]beahh[.]com/v` every 50 minutes.



 ### Where
 
The attack was primarily focused on **mts-dc.mts.local** (the domain controller), with lateral movement observed to **mts-contractorpc1**. All malicious file artifacts are detailed in the Findings section above.


### Why
 
The exact motive is unknown; however, based on the observed techniques — credential dumping, backdoor account creation, C2 beaconing, fileless PowerShell execution, and lateral movement — this is consistent with a targeted intrusion likely aimed at establishing long-term persistent access to the environment, potential data exfiltration, or pre-ransomware staging.



 
## Recommendations
 
### Immediate Actions
 
- Due to the critical role of mts-dc.mts.local as the domain controller, full network isolation is not recommended as this would cause significant business disruption. Instead, it is recommended to block malicious traffic at the perimeter firewall, implement enhanced monitoring on the DC, and engage a senior incident response team to assess the feasibility of a controlled remediation window.
- Disable and delete the backdoor account "geshli".
- Reset the local administrator password on both mts-dc and mts-contractorpc1.
- Block `95[.]189[.]49[.]66` at the perimeter firewall.
- Block all C2 domains at the DNS and firewall level, including all subdomains:
  - `*.beahh[.]com`
  - `*.abbny[.]com`
  - `*.haqo[.]net`
- Delete the scheduled task `\Microsoft\windows\Bluetooths`.
- Remove all identified file and registry IOCs listed in the Findings section, as all have been confirmed malicious by VirusTotal.
- Identify and remove the masquerading `.png` payload files observed in Figure 14, as these are assessed to be staged payloads or data exfiltration artifacts linked to `wmiex.exe`.
### Long-Term Actions
 
- Implement **LAPS (Local Administrator Password Solution)** to randomise local administrator passwords on a per-machine basis. The absence of brute force activity during lateral movement to mts-contractorpc1 strongly suggests password reuse from the compromised domain controller, which LAPS would mitigate.
- Implement **Attack Surface Reduction (ASR) Rules** to:
  - Block processes from reading lsass.exe memory, mitigating credential dumping via Mimikatz and similar tooling.
  - Block execution of encoded PowerShell commands, preventing fileless execution techniques such as those observed via `tmp.vbs` and the Bluetooths scheduled task.
 
## Screenshots


<p align="center">
  <img width="886" height="418" alt="image" src="https://github.com/user-attachments/assets/9992c018-238b-4996-b007-a959feb85603" />
</p>
<p align="center"><b>Figure 1:  MsMpEng.exe modifying the registry key SOFTWARE\Microsoft\Windows Defender\Real-Time Protection to set DisableRealtimeMonitoring to 1, disabling Microsoft Defender Real-Time Protection</b></p>

<p align="center">
  <img width="670" height="221" alt="image" src="https://github.com/user-attachments/assets/2cdc2ff2-6967-41ab-901b-f7f3ae689877" />
</p>
<p align="center"><b>Figure 2: WHOIS lookup for IP 95.189.49.66 confirming attribution to the Russian Federation, Irkutsk.</b></p>

<p align="center">
  <img width="1238" height="812" alt="image" src="https://github.com/user-attachments/assets/7e19c6c9-926c-433f-b4ac-d4377cceadc5" />
</p>
<p align="center"><b>Figure 3: AbuseIPDB lookup for 95.189.49.66 showing 22 reports from 13 distinct sources with a 7% confidence of abuse, previously reported for port scanning and brute-force activity against port 445 (Microsoft-DS), with the most recent report one month prior to the time of investigation.</b></p>

<p align="center">
  <img width="1719" height="409" alt="image" src="https://github.com/user-attachments/assets/32e404bb-d645-4386-8807-6d7cb84547b9" />
</p>
<p align="center"><b>Figure 4: VirusTotal analysis of upinstalled.exe flagged as malicious by 65/71 vendors, classified as trojan.fsysna/adzpf with threat categories of trojan, dropper, and spyware — behavioural tags include persistence, calls-wmi, spreader, and detect-debug-environment.</b></p>

<p align="center">
  <img width="720" height="738" alt="image" src="https://github.com/user-attachments/assets/792e5ca1-4dab-4383-9b3f-af3ccdd9f8de" />
</p>
<p align="center"><b>Figure 5: upinstalled.exe dropping ttt.exe onto the host.</b></p>

<p align="center">
  <img width="692" height="605" alt="image" src="https://github.com/user-attachments/assets/15adea0b-6ac4-4c88-9e21-d2eaf918bec7" />
</p>
<p align="center"><b>Figure 6: ttt.exe copying itself to C:\Windows\SysWOW64\wmiex.exe, masquerading as a legitimate Windows binary.</b></p>


<p align="center">
  <img width="1313" height="106" alt="image" src="https://github.com/user-attachments/assets/522ffe80-81ad-4016-a171-22b216448a14" />
</p>
<p align="center"><b>Figure 7: CruQgYgT.exe executing tmp.vbs via wscript.exe.</b></p>

<p align="center">
  <img width="2265" height="162" alt="image" src="https://github.com/user-attachments/assets/d256005c-9147-4471-94e2-39d0d2acc85d" />

  <img width="1451" height="185" alt="image" src="https://github.com/user-attachments/assets/23494e33-a73b-40fb-a3e6-467755f1895b" />
</p>
<p align="center"><b>Figure 8: wscript.exe executing tmp.vbs under the SYSTEM account, which in turn spawns cmd.exe to create the scheduled task "\Microsoft\windows\Bluetooths" containing an encoded PowerShell payload and appending c:\windows\temp\svchost.exe for immediate execution.</b></p>

<p align="center">
  <img width="1269" height="607" alt="image" src="https://github.com/user-attachments/assets/d97bbc8c-bd77-44ae-a3a9-f30aefc02fe0" />
</p>
<p align="center"><b>Figure 9: VirusTotal analysis of m.ps1 flagged as malicious by 41/63 vendors, with YARA community signatures matching ps1_toolkit_Invoke_Mimikatz (authored by Florian Roth) and power_pe_injection indicating PowerShell with PE Reflective Injection — authored by Benjamin Delpy (gentilkiwi), the creator of Mimikatz — confirming this script as a Mimikatz-based credential dumping tool.</b></p>


<p align="center">
  <img width="906" height="824" alt="image" src="https://github.com/user-attachments/assets/d64189a9-b8a4-4904-b121-2ccb26fa6b93" />
</p>
<p align="center"><b>Figure 10: m.ps1 performing a Windows API call to read the memory contents of lsass.exe under the SYSTEM account, confirming credential dumping.</b></p>


<p align="center">
  <img width="731" height="580" alt="image" src="https://github.com/user-attachments/assets/38127afa-4ab0-4fc4-a014-78e691e0c24d" />
</p>
<p align="center"><b>Figure 11: Successful network logon recorded on mts-contractorpc1 using the local administrator account, originating from mts-dc.</b></p>


<p align="center">
  <img width="370" height="1179" alt="image" src="https://github.com/user-attachments/assets/1a0258e4-81c7-4b80-8fa7-f259c550f12b" />
</p>
<p align="center"><b>Figure 12: Private IP address of mts-dc confirmed as the source of the lateral movement logon to mts-contractorpc1.</b></p>


<p align="center">
  <img width="729" height="205" alt="image" src="https://github.com/user-attachments/assets/f0de8a14-9854-419a-9e7e-78eb6787a464" />
</p>
<p align="center"><b>Figure 13: Backdoor account "geshli" created and added to the Administrators group and Remote Desktop Users.</b></p>


<p align="center">
  <img width="1664" height="845" alt="image" src="https://github.com/user-attachments/assets/bdb560d0-7b37-4070-b613-53e9d0604bd0" />
</p>
<p align="center"><b>Figure 14: Malicious .png payload files observed in logs, flagged by VirusTotal and linked to wmiex.exe in the SysWOW64 directory.</b></p>


<p align="center">
  <img width="914" height="343" alt="image" src="https://github.com/user-attachments/assets/35ba4cde-1bc9-4502-9d56-8110289a5701" />
</p>
<p align="center"><b>Figure 15: wmiex.exe observed establishing a connection to oo[.]beahh[.]com.</b></p>


<p align="center">
  <img width="1389" height="445" alt="image" src="https://github.com/user-attachments/assets/b18b8e76-34ea-488f-892f-4ebf4f946db8" />
</p>
<p align="center"><b>Figure 16: KQL query confirming no other devices in the environment had made connections to oo[.]beahh[.]com at the time of investigation.</b></p>
