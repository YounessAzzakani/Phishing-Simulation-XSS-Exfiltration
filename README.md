# Phishing Simulation & XSS Exfiltration — Report

**Status:** Educational lab (isolated environment)  
**Filename:** projects/phishing_simulation_xss_exfiltration/report.md

---

## Purpose
This lab demonstrates a chained attack combining social engineering (phishing), site cloning and a reflected XSS vector to exfiltrate session cookies and achieve session hijacking. The exercise was executed in an isolated virtual environment for educational purposes only.

---

## Tools & Techniques
- **SEToolkit** — site cloning and phishing email generation.  
- **Python 3 `http.server`** — lightweight listener on attacker VM to receive exfiltrated data.  
- **Browser extension (CookieManager+)** — manual injection of stolen cookies to validate session hijacking.  
- **Manual web testing** — find reflected XSS sinks (e.g., `search.php?keyword=`), encode payloads and chain them into phishing links.  
- **Other tools:** netcat/curl for quick testing, basic SMTP/lab mailserver for delivery.

**Technique summary:**  
1. Clone a legitimate login page with SEToolkit → create credential harvester.  
2. Craft phishing email containing a URL-encoded reflected XSS payload in a query parameter.  
3. User clicks phishing link → payload executes and sends `document.cookie` to attacker server.  
4. Attacker collects PHPSESSID and imports cookie via CookieManager+ → session hijack.

---

## Key Findings
- **Credential harvesting via cloned pages**: The cloned login page successfully collected submitted credentials when the victim used the page.  
- **Reflected XSS enables cookie exfiltration**: A reflected XSS vulnerability in a search parameter allowed execution of a payload that exfiltrated `PHPSESSID` to the attacker host.  
- **Session hijacking after cookie theft is trivial**: Injecting the stolen session cookie into the attacker browser granted immediate access to the victim’s authenticated session.  
- **Email realism matters**: Crafting realistic headers and using a lab SMTP server significantly increased click-through likelihood.

---

## Recommendations (Prioritized)
### Application-level fixes
- Encode and sanitize all user-supplied output (output encoding) to prevent XSS.  
- Implement a strict **Content Security Policy (CSP)** that blocks inline scripts and restricts script sources.  
- Mark session cookies with **HttpOnly**, **Secure**, and appropriate **SameSite** attributes to prevent access via `document.cookie`.

### Session management & hardening
- Shorten idle session timeouts and implement anomaly detection (IP/geolocation/browser fingerprinting).  
- Invalidate sessions on critical events (password reset, suspicious activity).

### Email & delivery controls
- Enforce **SPF / DKIM / DMARC** to reduce sender spoofing and phishing success.  
- Use anti-phishing/anti-spam gateways and scan or quarantine suspicious messages.

### Monitoring & operations
- Monitor HTTP server logs and DNS/HTTP patterns for suspicious callback requests (exfiltration endpoints).  
- Maintain regular security testing (automated scans + manual reviews of input handling).

### Organizational controls
- Run regular user-awareness and phishing training (simulate phishing campaigns + targeted remediation).  
- Enforce MFA for web applications to reduce the usefulness of stolen credentials.  
- Maintain incident playbooks: reporting steps, session invalidation procedures, credential rotation.

---

## Email & Network Mitigations (additional)
- Publish and enforce SPF/DKIM/DMARC policies to reduce spoofing.  
- Implement SMTP rate limits and alerting for abnormal mass-mailing.  
- Use inbound link rewriting/scanning to check links at click time.  
- Implement network segmentation to limit reach from compromised workstations.

---

## Ethical & Safety Note
This lab was performed in a controlled, isolated environment for educational purposes. Do **not** reproduce these techniques against systems you do not own or have explicit authorization to test. Unauthorized testing is illegal and unethical.

---

## Conclusion
A simple chain (site cloning + phishing + reflected XSS) can lead to full session hijacking when applications expose XSS flaws and session cookies are accessible via JavaScript. Defenses require layered measures: secure coding (prevent XSS), hardened session attributes, robust email authentication, network controls, and ongoing user training. Reproducing such attacks in labs helps validate detection rules and improve incident response playbooks.
