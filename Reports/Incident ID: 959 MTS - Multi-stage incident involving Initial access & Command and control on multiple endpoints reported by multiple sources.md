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
