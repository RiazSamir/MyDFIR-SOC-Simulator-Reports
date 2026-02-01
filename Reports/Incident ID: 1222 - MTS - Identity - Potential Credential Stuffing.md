# Incident ID: 1222 - MTS - Identity - Potential Credential Stuffing

## Findings

IOC IP Address:

147[.]124[.]219[.]77 - United States Carrollton Majestic Hosting Solutions LLC

## Investigation Summary 

On 2026-01-26 06:44:05 AM (UTC), brute-force activity was observed on the device "MTS-ContractorPC1". Multiple failed logon attempts were observed for multiple accounts sourcing from the IP Address 147[.]124[.]219[.]77. This was identified as brute-force, as the attacker attempted thousands of guesses across different accounts, primarily the "ADMINISTRATOR" account for privileged access (*Figure 1*). 

Based on the evidence, no successful logins had occurred, and there was no evidence of system compromise at the time. Although this attempt was unsuccessful, the system "MTS-ContractorPC1" has exposed remote services, which present a continuous risk in the event a weak password is used for any accounts

## WHO, WHAT, WHEN, WHERE and WHY

Who 

The IP Address 147[.]124[.]219[.]77 was observed trying to access multiple accounts as trying to brute force the account "ADMINISTRATOR" on "MTS-ContractorPC1". 
	
	
What 

On 2026-01-26 21:46:47 PM (UTC) an alert was generated for potential credential stuffing. Reviewing SecurityEvent logs shows that the IP Address (147[.]124[.]219[.]77) was seen attempting to access multiple accounts on the computer "MTS-ContractorPC1" via a network logon (LogonType 3). Based on the available network logs the attacker is attempting to connect to "MTS-ContractorPC1" via port 3389, which is the port associated with RDP (Figure 2). 

The Attacker was observed trying to brute force access into the account "ADMINISTRATOR" with Event ID 4625 (Invalid Username or Password) being generated 6192 times, as well as other accounts, including "ADMIN", "TEST", "USER" (*Figure 1*). Based on the available evidence, this brute force had taken place on 2026-01-26 06:44:05 AM (UTC) and last seen on 2026-01-27 00:18:34 AM (UTC) based on a 30-day search to present. 

Reviewing for successful logons associated with this IP against Event ID 4624 (successful Login) shows that no successful logons were made at the time of investigation (Figure 3). Based on the available evidence, we can see that the malicious IP had only interacted with the Computer MTS-ContractorPC1 (Figure 4)

The Malicious IP Address is based in the United States, Carrollton Majestic Hosting Solutions LLC and performing an AbuseIPDB lookup shows that the IP Address 3 times with a confidence of abuse at 2%. The last report was 5 days at the time of investigation (2026-01-27 05:10 AM) and was reported for RDP Brute Forcing (Figure 5)


	
### When 

Based on the available evidence, this brute force had taken place on 2026-01-26 06:44:05 AM (UTC) and last seen on 2026-01-27 00:18:34 AM (UTC) based on the last 30 days. 
	
	
### Where 

This has occurred on the endpoint "MTS-ContractorPC1" via a network logon, which can be through services such as RDP, SMB or other remote services. Based on the Device network logs, the attacker is attempting to connect to "MTS-ContractorPC1" via port 3389, which is the port associated with RDP. 
	
	
### Why 

The attacker is attempting to brute force access into different accounts, mainly "ADMINISTRATOR", to gain unauthorised privileged access. 

## Recommendations:

### Immediate

- No evidence of successful compromise was observed; however, as a precaution, consider a password reset for targeted privileged accounts. Furthermore, continuous monitoring for any unauthorised access onto MTS-ContractorPC1 should be done via Event ID 4624 for successful logons as well as 4672 for successful privileged logons.

### Short-Term (24-28 Hours)

- A lockout policy needs to be implemented to prevent attacks such as password spraying and brute forcing after a certain number of incorrect login attempts have been made. 
- Implement MFA for all accounts, especially for Privileged account,s to prevent unauthorised access
- Implement a Firewall ACL rule so that only authorised IP Addresses can RDP onto machines which have RDP opened.

### Long-Term (1-4) Weeks

- Audit Account usage and remove accounts which are not used (if applicable) 
- Implement Local Administrator Password Solution (LAPS) for increased protection for local administrator accounts. This will prevent weak passwords/same password from being used.

## Screenshots

<p align="center">
  <img width="1353" height="132" alt="image" src="https://github.com/user-attachments/assets/a9a07aa1-63ca-4f00-89a1-f89ea2a0271e" />
</p>
<p align="center"><b>Figure 1: Malicious IP Generating multiple invalid Username or Password Events (Event ID 4625) across multiple usernames over the past 30-days.</b></p>


<p align="center">
  <img width="446" height="856" alt="image" src="https://github.com/user-attachments/assets/8c8a933e-87bd-483e-9b22-63f91e536e18" />
</p>
<p align="center"><b>Figure 2: Inbound connections to MTS-ContractorPC1 via RDP initiated by the malicious IP Address.</b></p>


<p align="center">
  <img width="919" height="299" alt="image" src="https://github.com/user-attachments/assets/35f3f32e-7c9c-49f7-9895-b9f01c8b5030" />
</p>
<p align="center"><b>Figure 3: KQL Query to detect if any successful logons were made by the suspicious IP Address over the past 30-days. </b></p>


<p align="center">
  <img width="414" height="299" alt="image" src="https://github.com/user-attachments/assets/a75f1bb7-d899-40e5-b719-0817fbb37335" />
</p>
<p align="center"><b>Figure 4: KQL Query showing that the malicious IP had only interacted with MTS-ContractorPC1 over the past 30-Days </b></p

<p align="center">
  <img width="1263" height="790" alt="image" src="https://github.com/user-attachments/assets/0419eb33-59d6-427d-98e3-1014020357f6" />
</p>
<p align="center"><b>Figure 5: AbuseIPDB Reputation Check Showing Prior Brute-Force RDP Reports for IP 147.124.219.77</b></p
