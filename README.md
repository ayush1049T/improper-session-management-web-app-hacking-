# ADMIN ACESS PANEL FLAW

Target: SecureCorp  Management Portal ( 192.168.1.14 )
Vulnerability Type: Improper Session Management / Broken Access Control (Cookie Tampering)
Objective: Escalate privileges to Administrator and capture the hidden flag.
1. Executive Summary
   
This report details the successful privilege escalation attack against the SecureCorp internal portal. By intercepting web traffic and identifying an insecure authorization mechanism, the attacker successfully modified a client-side session cookie to bypass access controls. This resulted in full access to the Administrator's "Infrastructure Management" dashboard, exposing critical environment variables, internal network nodes, system event logs, and the target CTF flag.

3. Exploitation Methodology: Session Tampering
   
The application was found to be vulnerable to Improper Session Management. Instead of securely validating user roles on the server-side against a secure session token, the application relied on a plaintext, easily modifiable cookie ( user_role ).

Step 1: Traffic Interception
Using Burp Suite Community Edition, the HTTP GET request directed to the server (requesting access to the dashboard/admin panel) was intercepted before it left the browser.

Step 2: Modifying the Payload
Upon inspecting the intercepted request headers, the Cookie header was identified as containing role-based information. To exploit this, the user_role parameter was manually altered to escalate privileges.
Original/Standard Request Header (Example): Cookie:
PHPSESSID=kksodqflm71013nru8lvb3qqe6; user_role=employee
Modified Malicious Request Header: Cookie: PHPSESSID=kksodqflm71013nru8lvb3qqe6; user_role=admin
The modified request was then forwarded to the server.

 <img width="1920" height="1080" alt="Screenshot (245)" src="https://github.com/user-attachments/assets/70f11f6d-7cbe-4892-ba5f-a7c3e1f6cffa" />

Figure 1: Burp Suite Proxy intercepting the HTTP request. The user_role=admin payload is successfully injected into the Cookie header to bypass authorization checks.

3. Post-Exploitation & Flag Capture

Because the backend server blindly trusted the tampered cookie without cryptographic validation, it responded with a 200 OK status and rendered the highly restricted Administrator view.
Proof of Access
The browser successfully loaded the Infrastructure Management panel at http://192.168.1.14/dashboard.php . An "Admin Session: Active" badge confirmed the
successful privilege escalation.
                 
Flag Capture

Within the "Core Environment Variables" table, the target flag was successfully recovered under the JWT_SECRET variable.
CAPTURED FLAG: FLAG{ADMIN_PANEL_ACCESS}

 <img width="1920" height="1080" alt="Screenshot (244)" src="https://github.com/user-attachments/assets/b67da669-a41f-4174-89d2-37562a81bb68" />


Figure 2: The SecureCorp Management dashboard accessed via session tampering, explicitly showing the active admin session and the captured flag.

4. Remediation Recommendations

To secure the application against this attack vector:
1.	Never trust client-side data for authorization: Remove the user_role cookie entirely.
2.	Implement Secure Sessions: Store user roles in a backend database associated with a cryptographically secure, unpredictable session identifier (e.g., a properly configured PHP session or a signed JWT).
3.	Enforce Server-Side Checks: For every request to a privileged endpoint (like /admin.php or administrative views in /dashboard.php ), the backend must query the database to verify the privileges of the currently authenticated session ID.
