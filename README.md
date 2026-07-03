# Improper-session-management-web-app-hacking
"Improper Session Management" is a standard vulnerability category. In CTF environments, this usually means the web application relies on cookies or session tokens that are predictable, fail to expire, or can be manipulated to impersonate another user (usually the admin)
Vulnerability & Exploitation Report
Target: SecureCorp Employee Portal

Vulnerability: Improper Session Management & Privilege Escalation

Prepared for: TuteDude CyberSecurity Lab Assignment





1.	Executive Summary

This document outlines the successful exploitation of an improper session management vulnerability on the SecureCorp Employee Portal. By analyzing client-side code, hardcoded credentials were recovered. Subsequently, traffic interception via Burp Suite revealed an insecure, client-modifiable authorization cookie. By tampering with this cookie, privilege escalation was achieved, granting full administrative access and revealing the hidden capture-the-flag (CTF) value.

2.	Reconnaissance & Client-Side Analysis

Finding Initial Credentials

The attack methodology began with a thorough review of the application's source code on the login
page (index.php). Inspecting the HTML source revealed a developer comment mistakenly left in the production environment.





This resulted in identifying valid low-privileged credentials: Username: John | Password:
babayaga

 

3.	Authentication & Traffic Interception

Identifying the Session Management Flaw

Using the discovered credentials, a login attempt was made. Burp Suite was utilized with Intercept On to capture the server's response. Upon successful authentication, the server issued the following HTTP response headers:






Analysis of the Flaw: The application tracks authorization state using a plaintext cookie
(user_role=employee). Because the application trusts the client-provided cookie without server-side validation or cryptographic signatures (such as a JWT or secure session token), it is
vulnerable to Insecure Direct Object Reference (IDOR) and Privilege Escalation via session tampering.
 









4.	Exploitation & Privilege Escalation

Modifying the Session Cookie

To exploit this vulnerability, the next request to /dashboard.php was intercepted in Burp Suite before it reached the server. The request header originally contained the standard cookie:




In Burp Suite's Repeater/Proxy tab, the cookie value was manipulated to escalate privileges to the administrator role:
The modified request was then forwarded to the server. Because the server blindly trusts the cookie value to determine access levels, the authorization check was successfully bypassed.
 






5.	Proof of Concept: Flag Capture

Upon forwarding the tampered request, the server responded by loading the administrative dashboard. The application rendered an administrative component containing the requested CTF flag, proving successful privilege escalation.



 






6.	Conclusion and Attack Methodology Summary

The successful breach of the system followed a clear three-step methodology:

•	Information Disclosure: Analyzing front-end code to extract hardcoded default credentials.
•	Traffic Analysis: Using an interception proxy (Burp Suite) to inspect state-tracking mechanisms, revealing a plaintext role cookie.
•	Session Tampering (Exploitation): Modifying the insecure cookie from employee to
admin, resulting in vertical privilege escalation.


