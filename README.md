# Phishing Simulation & XSS Exfiltration — report.md

## Purpose
This lab demonstrates a chained attack combining social engineering (phishing), site cloning and a reflected XSS vector to exfiltrate session cookies and perform session hijacking. The exercise is strictly educational and was executed in an isolated virtual environment. :contentReference[oaicite:1]{index=1}

---

## Tools and Techniques
- **SEToolkit (Social-Engineer Toolkit)** — site cloner and mass mailer used to create a credential-harvesting page and to send phishing emails.
- **Python 3 `http.server`** — lightweight HTTP server on the attacker VM to receive exfiltrated data (cookies).  
- **Browser extensions (CookieManager+)** — for manual injection of stolen session cookies to validate session hijacking. 
- **Manual web testing** — identification of reflected XSS points (e.g. `search.php?keyword=`), URL-encoding payloads and chaining them into phishing links. :contentReference[oaicite:5]{index=5}

**Technique summary:** clone a login page → lure victim via phishing email → victim clicks link containing URL-encoded XSS payload → the payload executes in victim's browser and sends `document.cookie` to attacker server → attacker extracts cookie and injects it into their browser to hijack the session.

---

## Key Findings
1. **Cloned login pages effectively harvest credentials** — the credential harvester captured credentials submitted by the victim to the cloned page.   
2. **Reflected XSS enabled cookie exfiltration** — a reflected XSS vulnerability in a search parameter allowed execution of a payload that exfiltrated the `PHPSESSID` to the attacker host.  
3. **Session hijacking is trivial after cookie theft** — importing the stolen session cookie into the attacker browser (via CookieManager+) granted access to the victim’s authenticated session.
4. **Email delivery increases success likelihood** — crafting realistic email headers and using a lab SMTP server makes users more likely to click the phishing link. 

---

## Recommendations
(High-level, prioritized actions to reduce risk.)

1. **Fix application-level vulnerabilities**
   - Escape and sanitize all user-supplied output (output encoding) to prevent XSS vectors. :contentReference[oaicite:11]{index=11}  
   - Implement and enforce a strict **Content Security Policy (CSP)** that disallows inline scripts and restricts allowed script sources. 
   - Set session cookies with `HttpOnly`, `Secure`, and appropriate `SameSite` attributes to block access via `document.cookie` and limit cross-site exfiltration. 

2. **Harden session management**
   - Reduce session idle timeouts and implement anomaly detection for session usage (IP/geolocation/browser fingerprinting). :contentReference[oaicite:14]{index=14}  
   - Invalidate sessions after critical events (password change, suspicious activity).

3. **Improve email authentication & filtering**
   - Deploy SPF, DKIM and DMARC to reduce sender spoofing and phishing success.
   - Use anti-phishing/anti-spam gateways and quarantine suspicious messages before delivery.

4. **Operational & monitoring**
   - Monitor HTTP server logs and DNS/HTTP patterns for suspicious callback requests (exfiltration endpoints).
   - Maintain regular security testing (including automated scans and periodic manual review of input handling).

---

## Email & Network Mitigations
- **SPF / DKIM / DMARC** — publish and enforce policies so recipient servers can detect spoofed senders and reject or quarantine phishing messages. 
- **SMTP rate/volume monitoring** — detect abnormal mass-mailing from internal servers and block suspicious senders.   
- **Inbound link rewriting / scanning** — rewrite external links in emails and scan them at click time for known malicious indicators.  
- **Network segmentation** — separate user workstations, web servers and mail infrastructure so successful phishing on a workstation cannot directly reach sensitive servers. 

---

## Organizational / User-focused Mitigations
- **User awareness & phishing training** — regular simulated phishing campaigns combined with targeted training to improve detection of cloned domains, suspicious attachments and abnormal requests.}  
- **Enforce MFA** — require multi-factor authentication for all web applications when possible; stolen credentials become far less useful.  
- **Policy & incident playbooks** — define clear steps for reporting suspected phishing, invalidating sessions, and rotating affected credentials.

---

## Conclusion
The lab demonstrates how a relatively simple chain (site cloning + phishing + reflected XSS) can lead to full session hijacking in an environment with vulnerable web applications and weak email protections. Defense requires a layered approach: secure coding (prevent XSS), strong session handling (HttpOnly/Secure/SameSite), robust email authentication (SPF/DKIM/DMARC), network controls and ongoing user training. Reproducing such attacks in a controlled lab is valuable to validate defenses and refine detection capabilities. 

---

**Original (full) lab report:** `Rapport TP1 - Simulation d’une attaque de phishing.pdf` (included in this repository).
