# Batch-B083--Week-4


# Penetration Testing Report — Mediroza General Hospital#



# Project overview
This repository documents a black-box penetration test of the Mediroza General Hospital web infrastructure at https://medirozahospital.com. The assessment was performed for the Networkwalks B082 Week 4 Capstone Project to identify weaknesses, demonstrate their impact through controlled exploitation, and recommend practical remediation.

Findings, ranging from Low to Critical, were identified. The central issue was a SQL injection vulnerability in the patient portal login flow. In the authorized test environment, it enabled authentication bypass, access to confidential patient lab-report PDFs, discovery of sensitive PDF metadata, and retrieval of an exposed database backup containing staff salary and shareholder information.

# Objectives
Assess the web application from an external, unauthenticated perspective.</br>
Identify weaknesses in authentication, input handling, file protection, and server configuration.</br>
Demonstrate the real world impact of each finding in a controlled manner.</br>
Document evidence and provide prioritized remediation recommendations.</br>

|||
|---|----|
|Classification:| CONFIDENTIAL — Networkwalks / Authorised Personnel Only |
|Client:| Mediroza General Hospital 
|Target: |https://medirozahospital.com 
|Engagement Type: |Black-box Web Application Penetration Test 
|Duration:| 5 days | 
|Tester:| [Etouke B. Cedric] — Batch B083, Week 4 
|Authorisation: |Written permission granted by Networkwalks for this controlled educational engagement.

# 1. Executive Summary
Mediroza General Hospital commissioned a black-box penetration test of its public web infrastructure. Over five days, the assessment identified a chain of serious weaknesses that, combined, allow an unauthenticated internet attacker to access confidential patient medical records and highly sensitive corporate information.

**Overall Risk: CRITICAL.** The most significant finding is an unprotected database backup file containing employee payroll data, national identity numbers, contact details, and hospital shareholder records, discoverable via a directory listing referenced in `robots.txt.` This was followed by weaknesses in the patient portal authentication and a directory traversal flaw in the report download mechanism, enabling retrieval of encrypted patient lab reports whose protection was then defeated.</br>
Key findings summary:


|#|Finding|Severity|
|---|----|----|
|1	|Unauthenticated exposed database backup (PII, salaries, shareholders)	|Critical
|2	|Directory listing enabled on sensitive paths `(/patient/, /staff/, /old/)`	|High|
|3	|Weak/defeated authentication on patient portal	|High|
|4	|Path traversal / insecure file download in download.php	|Critical|
|5	|Insufficient PDF encryption on patient lab reports	|High|
|6	|Exposed error_log (243 KB) leaking application diagnostics|	Medium|
|7	|Sensitive paths disclosed in robots.txt|	Low|
|8	|Missing security headers / information disclosure (CMS version, LiteSpeed)	|Low–Medium|

# 2. Scope and Methodology
In-scope target: medirozahospital.com (web application and associated directories only). Out of scope: social engineering, denial of service, any testing outside the agreed domain.

**Methodology:** 

|||
|---|----|
|`Reconnaissance`| Passive information gathering using publicly available information and web-based tools.
| `Enumeration` | Active information extraction of specific or deeper details 
|`Vulnerability identification`| Analysis of application behavior for authentication and input-handling weaknesses.
|`Exploitation`, `Post-exploitation analysis` |Demonstration of each issue's impact within the authorized environment.
|`Reporting`| Aligned with OWASP Testing Guide and PTES.

**Tools used:** `whois`, `dnsenum`, `nmap`, `whatweb`, nikto, `gobuster`, `curl`, `Burp Suite`, `wget`, `hash calculator`, `OnlineHashCrack`, john/hashcat, .

Limitations: No DoS testing; no exploitation of out-of-scope hosts; destructive testing avoided.

# 3. Findings and Proof of Exploitation

## 
F-01 — Weak Patient Portal Authentication (High)
Milestone 1/3 evidence:

Footprinting with `whois,` `nslookup`, `curl`,`theHarvester`, `nmap` which guided me with where next to look
Login at /patient/login.php was assessed and defeated via [credential attack / SQLi / password reuse with data `Admin'--` and `password=anything` Authenticated session obtained; portal exposes patient lab reports. 
![image](https://github.com/Etouke-22/Batch-B083--Week-4/blob/main/Placeholder.png?raw=true)[screenshots: successful login, portal view] 
![image](https://github.com/Etouke-22/Batch-B083--Week-4/blob/main/form%20brutefoece.png?raw=true) 
![image](https://github.com/Etouke-22/Batch-B083--Week-4/blob/main/File%20access.png?raw=true)




F-02 — Directory Listing on Sensitive Paths (High)
/patient/, /staff/, and /old/ all return autoindex listings revealing file structure: login.php, download.php, portal.php, logout.php, reports/, error_log (243 KB). — [screenshots of each index]


F-03 — Exposed Database Backup (Critical)
`robots.txt` revealed` Disallow: /old/` — [screenshot]
`https://medirozahospital.com/old/` showed an open directory index listing `mediroza_db_backup_2019.sql` (7 KB) — [screenshot]
File downloaded unauthenticated via `wget;` contains full `mediroza_hr dump:` `staff` table (30 records with salaries in ZAR, national IDs, personal phone numbers, emails) and `shareholders` table (10 shareholders with ownership percentages). Header states: **"WARNING: contains confidential staff and shareholder records."** </br>
Full data extracted in Section 4 of the engagement workbook (salaries e.g. Medical Director R160,000/mo; shareholder splits e.g. Dr. R. Naidoo 18%).


F-04 — Path Traversal / Insecure Direct Object Reference in download.php (Critical)
The report download endpoint accepts a user-controlled file parameter without sanitisation, allowing retrieval of files belonging to other patients (IDOR) or traversal outside the intended directory. — [INSERT YOUR EXPLOIT REQUEST/RESPONSE AND THE 3 RETRIEVED PDFs]


F-05 — Defeated PDF Encryption (High)
All three patient lab reports used PDF encryption that was recovered, but the passwords were weak and present in commonly available wordlists.
Two reports were recovered with the built-in 100-word list, while the third required a larger password list. 

File 1: [encryption type V/R from pdfinfo, mode 10500/10700, password: INSERT] ![image](https://github.com/Etouke-22/Batch-B083--Week-4/blob/90ee0410b3843cd6525dfeb00aef3ad530f902a3/passwd%20pdf1.png)

File 2: [type, method, password: INSERT] ![image](https://github.com/Etouke-22/Batch-B083--Week-4/blob/9b0450b6754c6a56e6477f767b5aa80ef7f7792f/passwd%20pdf%202.png)

File 3: [type, method, password: INSERT] Passwords were recovered with john/hashcat using wordlists derived from context (rockyou + custom list built from cewl and leaked staff data). Proof: cracking output + pdftotext of recovered contents. — [screenshots] ![image](https://github.com/Etouke-22/Batch-B083--Week-4/blob/199c2aba2d796f29359f73ec2ab073ffb03e0b61/passwd%20pdf%203.png)

F-06 — Exposed PHP Error Log (Medium)
/patient/error_log (243 KB) is present at a known path; listing exposure reveals its existence and size. Direct retrieval was blocked by 403 (WAF/UA filtering), but the filename is confirmed. — [screenshot of index entry] ![image]

F-07 — Sensitive Paths in robots.txt (Low)
Disallow: /patient/, /staff/, /old/ acts as a disclosure mechanism for attackers; all three were confirmed exploitable locations. ![image]

F-08 — Outdated/Versioned Software Disclosure (Low–Medium)
Server banner: LiteSpeed; footer: "Mediroza CMS 1.4.2" (also in the backup header, alongside the CMS backup module that produced F-01). Version enumeration enables targeted CVE research.

# 4 Risk Rating
|Finding | Rating | Justification|
|---|----|---|
|F-01 Database backup exposed|	Critical (CVSS ~9.1)|	Mass PII + payroll + corporate ownership data exposed to anonymous internet users; POPIA breach; reputational/regulatory/legal impact
|F-04 Path traversal/IDOR	|Critical (~8.6)|	Unauthorised access to patient medical records — direct patient-harm and compliance impact|
F-02 Directory listing|	High (~7.5)	|Provides the map for all subsequent exploitation; chain enabler
F-03 Weak authentication|	High (~7.4)	|Defeats the only control protecting patient data
F-05 Weak PDF encryption|	High (~7.0)	|Encryption defeated offline with commodity tools; passwords guessable from context
F-06 Exposed error_log|	Medium (~5.3)	|Diagnostic leakage may reveal paths/credentials; partially blocked
F-07 robots.txt disclosure|	Low (~3.1)	|Convenience to attackers; no direct impact alone
F-08 Version disclosure|	Low–Medium (3.1–5.3)	|Facilitates targeted exploitation




# 5. Recommendations and Remediation
1. Immediately remove and quarantine /old/mediroza_db_backup_2019.sql and any other backups from the web root; purge from caches/Google; rotate any credentials       stored in backups; notify affected staff (POPIA obligation).</br>
2. Disable directory listing (Options -Indexes / LiteSpeed equivalent) on all directories, especially /patient/, /staff/.</br>
3. Fix download.php: whitelist exact filenames, resolve paths server-side, reject any input containing /, .., or absolute paths; enforce per-session ownership        checks (IDOR).</br>
4. Harden authentication: strong password policy, account lockout/rate limiting, MFA; no password reuse across portals; remediate any SQLi with parameterised         queries.</br>
5. PDF protection: if encryption is required, use AES-256 with strong, non-contextual passwords delivered out-of-band; better — serve reports only through the        authenticated portal rather than password-protecting files.</br>
6. Remove or protect error_log; set display_errors=Off, log outside web root.</br>
7. Clean robots.txt — remove sensitive paths and enforce server-side access control instead of relying on crawler directives.</br>
8. Update Mediroza CMS and suppress version banners; apply security headers (HSTS, CSP, X-Frame-Options, X-Content-Type-Options).</br>
9. Retest after remediation and establish scheduled vulnerability scanning and backup-hygiene audits.</br>


# Lessons learned
i. Small security flaws can combine into a high-impact attack chain.</br>
ii. Authentication errors and database errors reveal valuable information to attackers.</br>
iii. Encryption is ineffective when document passwords are weak and easily guessed.</br>
iv. Document metadata requires the same security review as visible content.</br>
v. Backups must never be placed in publicly accessible web directories.</br>
vi. Defense in depth is essential: secure input handling, authorization, file storage, server configuration, and data governance must all work together.

# Conclusion
This assessment demonstrated a complete path from the login page to highly sensitive internal data using well-known, preventable weaknesses. Critical and High findings should be addressed immediately before the system is used to store or serve real patient data.




