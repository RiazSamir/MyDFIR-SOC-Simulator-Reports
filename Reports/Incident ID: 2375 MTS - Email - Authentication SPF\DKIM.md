# Incident ID: 2375 MTS - Email - Authentication SPF\DKIM

## Findings

### Email Addresses:
- Distribution[at]rodolfomedrano[.]dev

### URLs 

- hxxps://policy-shared-doc1804471[.]vercel[.]app/
	
- hxxp[:]//schaeffer-onlinenet73248[.]activehosted[.]com/proc[.]php?c=1&m=3&s=5576a22744cfd295fad41b3e49250684&d=7&v=2&nl=3&runid=36573&act=unsub

- hxxps://webmail[.]danr[.]ca/cpsess6702448767/3rdparty/roundcube/?_task=mail&_framed=1&_caps=&_uid=129966&_mbox=INBOX&_safe=1&_action=preview#v1

- emkei.cz 

### IP Addresses:

- 216[.]198[.]79[.]3 - Goes under the domain name nextjs.org and is a Content Delivery Network. The Autonomous System Number is AS16509
- 114[.]29[.]236[.]247 - Sending server of the fake mailer emkei[.]cz. Located in Hong Kong and Autonomous System Number being AS64022

## Investigation Summary

On 2026-07-02 [UTC], a phishing email spoofing "Canada Life" was sent to `Recipient A`, triggering an SPF/DKIM/DMARC authentication failure alert. Header analysis confirmed the email originated from emkei.cz, a free, publicly available fake mailer that allows anyone to forge the "From" address of an outgoing email without needing to compromise or control the domain being impersonated — in this case, rodolfomedrano.dev. All three authentication checks (SPF, DKIM, DMARC) failed, yet the email was still delivered to the recipient's inbox rather than being blocked or quarantined at the mail-flow stage.

The email used a classic document-lure subject ("Your Document Is Ready & Available To Download") to entice the recipient to click an embedded link. Ryan Adams did click the link, which pointed to a phishing page hosted on Vercel (hxxps://policy-shared-doc1804471[.]vercel[.]app/) presenting a fake, password-protected Adobe Acrobat PDF viewer. However, Safe Links successfully blocked the click before the user reached the page, and no evidence of credential entry, payload execution, or further compromise was identified.

A 90-day environment-wide sweep found this same phishing campaign had also targeted a second, unrelated recipient, `Recipient B` approximately two weeks earlier (2026-06-17), using the identical sender domain and subject line. That email was correctly quarantined and never reached the recipient's inbox. No other recipients or interactions with the malicious link were identified across the environment.
The recurrence of this identical lure against two unrelated organizations two weeks apart suggests this is part of a broader, non-targeted phishing campaign using emkei.cz, rather than an attack aimed specifically at either environment.

## Who, What, When, Where, Why

### Who 

- 2026-07-02 18:53 [UTC] The recipient `Recipient A` had received an email from Distribution[at]rodolfomedrano[.]dev
- 2026-07-17 17:20 [UTC] The Recipient `Recipient B` had also received a similar email from Distribution[at]rodolfomedrano[.]dev

### What 

2026-07-02 18:53 [UTC] An alert was generated for "Email - Authentication SPF/DKIM Fail." The recipient `Recipient A` had received an email from distribution[at]rodolfomedrano[.]dev. As the alert suggests, SPF, DKIM, and DMARC had all failed, as seen in both the alert and the email header (Figures 1 & 2). The Reply-To address remains consistent with the sender, so nothing to flag there (Figure 2). The display name "Canada Life" is observed in the email header a sign of display name spoofing, as this bears no relation to the actual sending domain. Reviewing the email in Threat Explorer, the subject header reads "Your Document Is Ready & Available To Download," a classic lure tactic to entice users to click the embedded link (see URLs in Findings Section). Strong evidence suggests this is phishing, as one of the sending mail servers is emkei[.]cz (Figure 2). Reviewing this site on VirusTotal, 6 vendors have flagged it as malicious, linked to a fake mailer — a tool/service used to send email with a forged "From" address, making it appear to come from any sender regardless of actual ownership (Figure 9). Abuse IPDB has 5 reports with abuse of confidence of 0%, however people have reported due to Spam and Phishing (Figure 10)



2026-07-02 19:07 [UTC] The user `Recipient A` interacted with the link hxxps://policy-shared-doc1804471[.]vercel[.]app/; this action was blocked (Figure 3). Per urlscan.io, this domain currently resolves to 64[.]29[.]17[.]67 (AWS/Amazon-02, United States), and the site presents a fake, password-protected Adobe Acrobat PDF viewer (Figure 4). VirusTotal has flagged the domain as potentially malicious (2/91 vendors) (Figure 5). As a precautionary measure, given the user interacted with a confirmed malicious link, a password reset and MFA token revocation is recommended.

- A 90 day search was done to see if anyone had interacted with this link however the initial user (Ryan Adams), was observed interacting with is (Figure 6)


2026-07-17 17:20 [UTC] A 90 day search was done to see if the sender had emailed anyone else's inbox. Recipient `Recipient B` had also received a similar email from Distribution[at]rodolfomedrano[.]dev, (Figure 8) However this was placed in Quarantine and should be removed.

2026-07-26 15:30 [UTC - Investigation Date] A 90 day search was done from the current date to see if the subject header "Your Document Is Ready & Available To Download" was observed anywhere else in the environment. From the date of investigation figure 8 returns the same results with the 2 recipients receiving this email (`Recipient A` & `Recipient B`)

### When

All timestamps are UTC. The earliest observed instance of this campaign occurred on 2026-06-17 (`Recipient B`), followed by the Kerning City Dental email on 2026-07-02 at 18:53, with the user interaction (blocked click) at 19:07 the same day. A follow-up 90-day sweep on 2026-07-26 confirmed no additional recipients beyond these two. There is no evidence this campaign is still active against this environment as of the time of investigation; both known emails were quarantined/blocked, and no successful compromise (credential entry, payload execution) has been identified.

### Where 

Two mailboxes were affected: `Recipient A` and `Recipient B`. No endpoint/device-level compromise has been identified — both incidents were contained at the email/URL layer (quarantine and Safe Links block, respectively).

### Why

The emails were sent using emkei.cz, a free, publicly available fake mailer that allows forging the "From" address without needing to compromise or control the impersonated domain. This does not require any prior access to the target organization — it is a low-effort, low-cost phishing technique relying on the recipient trusting the display name ("Canada Life") rather than verifying the actual sending domain. No evidence suggests this was a targeted attack against either organization specifically; the recurrence of the identical subject line and lure two weeks apart across two unrelated recipients (a dental practice and a tax solutions firm) suggests a broader, non-targeted phishing campaign rather than an attack aimed specifically at this environment.


## Recommendations 

- Reset the password and revoke/re-register MFA tokens for `Recipient A` as a precaution, given the user interacted with a confirmed malicious link. Review the account's recent sign-in activity for any anomalies following the click.
	
- Delete the quarantined email sent to `Recipient B` rather than leaving it in quarantine indefinitely, once confirmed no further investigation is needed.
	
- Block the known malicious indicators at the email gateway/firewall level: the domain rodolfomedrano[.]dev, the sending infrastructure emkei[.]cz, and the phishing URL hxxps://policy-shared-doc1804471[.]vercel[.]app/ as well as the toerh email artefacts found in the findings section. 
	
- Educate affected users (and consider a broader awareness reminder) on display name spoofing  specifically, checking the actual sending address/domain rather than trusting the display name shown by the mail client, since "Canada Life" bore no resemblance to the real sending domain.
	
- Consider tightening the DMARC/SPF/DKIM enforcement action - this message failed all three checks yet was still delivered to the inbox rather than being blocked outright at the mail flow level. Review current anti-phishing/anti-spam policy actions for messages that fail composite authentication (CompAuth: fail) to determine whether stricter action (quarantine or reject) should apply automatically.

- Continue monitoring for recurrence of this specific lure (subject line, sender domain pattern, or emkei.cz-originated mail) given it was observed twice, two weeks apart, against unrelated organizations - suggesting this is an ongoing, broader campaign.




## Screenshots

<p align="center">
  <img width="2006" height="981" alt="image" src="https://github.com/user-attachments/assets/4fb70eeb-4020-4557-9087-708a4cdbaea4" />
</p>
<p align="center"><b>Figure 1: Alert Generation for SPF/DKIM/DMARC Failure via Defender XDR Pannel </b></p>



<p align="center">
  <img width="523" height="730" alt="image" src="https://github.com/user-attachments/assets/9ed5ca91-9c3d-4689-b553-6331eb024aae" />
</p>
<p align="center"><b>Figure 2: Email Header Analysis – SPF/DKIM/DMARC Failure with Spoofed "Canada Life" as display name and fake mailer sending server.</b></p>


<p align="center">
  <img width="956" height="355" alt="image" src="https://github.com/user-attachments/assets/477a8dac-d27a-4d1b-930e-e14ce6197ebd" />
</p>
<p align="center"><b>Figure 3: KQL Query – UrlClickEvents Showing Recipient A Blocked Click on Phishing Link.</b></p>



<p align="center">
  <img width="2348" height="1113" alt="image" src="https://github.com/user-attachments/assets/e4bf53ab-a281-476c-9f97-dd2fc9df0c7c" />
</p>
<p align="center"><b>Figure 4: urlscan.io Result for hxxps://policy-shared-doc1804471[.]vercel[.]app/</b></p>



<p align="center">
  <img width="2429" height="435" alt="image" src="https://github.com/user-attachments/assets/2c2eea3c-9527-44f9-8ea2-9ce2a4b8a8d2" />
</p>
<p align="center"><b>Figure 5: VirusTotal Detection Results for policy-shared-doc1804471.vercel[.]app (2/93 Vendors Flagged Malicious)</b></p>

<p align="center">
  <img width="800" height="492" alt="image" src="https://github.com/user-attachments/assets/a04c4608-d6ac-44a2-99cb-6828bd4a6e2d" />
</p>
<p align="center"><b>Figure 6: KQL Query – 90-Day UrlClickEvents Search for Interactions with Malicious URLs</b></p>



<p align="center">
  <img width="737" height="742" alt="image" src="https://github.com/user-attachments/assets/e8b538c4-3739-4c73-b274-a0c1bee2e375" />
</p>
<p align="center"><b>Figure 7: KQL Query – EmailEvents Showing Second `Recipient B` Receiving Phishing Email</b></p>

<p align="center">
  <img width="1276" height="374" alt="image" src="https://github.com/user-attachments/assets/9f811941-e203-478f-96cf-ace6cd86a073" />
</p>
<p align="center"><b>Figure 8: KQL Query – 90-Day EmailEvents Search for Phishing Subject Across Environment</b></p>

<p align="center">
  <img width="1133" height="1036" alt="image" src="https://github.com/user-attachments/assets/9206b07e-4929-43d5-aba1-4225b383549d" />
</p>
<p align="center"><b>Figure 9: VirusTotal Community Detections and Comments for emkei.cz (6/91 Vendors Flagged Malicious)</b></p>



<p align="center">
  <img width="1207" height="1020" alt="image" src="https://github.com/user-attachments/assets/b04bfefa-546b-4021-b9e9-8766bb4e183c" />

</p>
<p align="center"><b>Figure 10: AbuseIPDB Report for IP 114.29.236.247 (emkei.cz Sending Server) showing that it is linked Phishing/Spam</b></p>
