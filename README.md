# Insecure Direct Object Reference (IDOR)

Target: SecureCorp Employee Portal (192.168.1.14)
Vulnerability: Insecure Direct Object Reference (IDOR)
Prepared for: TuteDude CyberSecurity Lab Assignment

Executive Summary

This document outlines the successful exploitation of an Insecure Direct Object Reference (IDOR) vulnerability within the SecureCorp Employee Portal. By chaining the site's search functionality with an insecure profile viewing mechanism, it was possible to enumerate administrative identifiers and bypass access controls. This allowed an unprivileged user to access highly sensitive administrative records and retrieve the hidden capture-the-flag (CTF) value.
Reconnaissance & Attack Methodology
The successful breach of the administrator's profile followed a specific methodology combining multiple application features:
1. Information Disclosure via Search Functionality

The attack began by identifying how the application references user accounts. By utilizing the built-in search feature (/search.php?q=admin), the application processes the query and inadvertently leaks the unique identifiers associated with other users. Through this enumeration, the administrator's unique target ID was identified as SID10001.

3. Identifying the IDOR Vulnerability

Traffic interception via Burp Suite revealed that the application fetches user profiles using a direct URL parameter. When viewing a standard employee profile (e.g., Abhishek Researcher), the GET request relies on the id parameter:
GET /profile.php?id=SID10582 HTTP/1.1
It was observed that the server implicitly trusts the client-provided id parameter without properly validating if the currently authenticated session is authorized to view that specific record.

 <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/5f33f14f-7ac9-4a82-a758-9f1fdb66f2f8" />

3. Exploitation & Parameter Manipulation

 To exploit the IDOR flaw, the target ID discovered during the search phase was substituted into the vulnerable endpoint. Additionally, if the backend enforces role-based checks on the profile page, combining this IDOR with session cookie manipulation (modifying the user_role cookie to admin, as you noted) guarantees the bypass of any secondary authorization filters.

 <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/0c3f851c-c79a-4c07-bcdc-b84da15f7523" />
 
•	Target Payload: GET /profile.php?id=SID10001

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/21fb61a6-7d49-4046-8def-7de1b693d828" />

Proof of Concept: Flag Capture

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/2281df19-1840-4573-bc61-cb5b08554c83" />

By forwarding the manipulated request, the server successfully returned the restricted profile page for Akshay Admin (CEO).
Because of the lack of robust server-side access controls, sensitive internal data was fully exposed to the attacker, including:
•	Payroll Information: Current Salary: $500,000
•	Internal Security Notes: FLAG{IDOR_SENSITIVE_LEAK}
Captured Flag: FLAG{IDOR_SENSITIVE_LEAK}
 
Conclusion

The root cause of this vulnerability is the application's reliance on client-provided input (id=SID...) to retrieve sensitive records without performing adequate server-side authorization checks. Because user IDs are easily enumerable via the search page, an attacker can systematically harvest these IDs and feed them into the /profile.php endpoint to compromise the confidentiality of any user on the platform.

