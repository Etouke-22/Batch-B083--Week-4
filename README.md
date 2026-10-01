# Batch-B083--Week-4


# Penetration Testing Report — Mediroza General Hospital#

|||
|---|----|
|Classification:| CONFIDENTIAL — Networkwalks / Authorised Personnel Only |
|Client:| Mediroza General Hospital 
|Target: |https://medirozahospital.com 
|Engagement Type: |Black-box Web Application Penetration Test 
|Duration:| 5 days | 
|Tester:| [Etouke B. Cedric] — Batch B083, Week 4 
|Authorisation: |Written permission granted by Networkwalks for this controlled educational engagement.




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







|Finding | Rating | Justification|
|---|----|---|
|hdkd |mdksdk |kdcl;|
|F-01 Database backup exposed	Critical (CVSS ~9.1)	Mass PII + payroll + corporate ownership data exposed to anonymous internet users; POPIA breach; reputational/regulatory/legal impact
F-04 Path traversal/IDOR	Critical (~8.6)	Unauthorised access to patient medical records — direct patient-harm and compliance impact
F-02 Directory listing	High (~7.5)	Provides the map for all subsequent exploitation; chain enabler
F-03 Weak authentication	High (~7.4)	Defeats the only control protecting patient data
F-05 Weak PDF encryption	High (~7.0)	Encryption defeated offline with commodity tools; passwords guessable from context
F-06 Exposed error_log	Medium (~5.3)	Diagnostic leakage may reveal paths/credentials; partially blocked
F-07 robots.txt disclosure	Low (~3.1)	Convenience to attackers; no direct impact alone
F-08 Version disclosure	Low–Medium (3.1–5.3)	Facilitates targeted exploitation
