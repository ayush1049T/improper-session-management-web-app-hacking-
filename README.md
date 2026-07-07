# Vulnerability Assessment Report(OTP)
Target: SecureCorp Portal (192.168.1.14)
Vulnerability Type: Insecure Authentication / Lack of Rate Limiting (Brute Force)
Severity: High

1. Executive Summary
A critical vulnerability was identified in the password reset mechanism of the SecureCorp Portal. The application utilizes a 4-digit One-Time Password (OTP) for account recovery but fails to implement adequate rate limiting, attempt thresholds, or expiration policies on the verification endpoint. This allows an attacker to easily brute-force the OTP and achieve complete account takeover.

3. Vulnerability Description
The endpoint /verify_otp.php processes the submitted OTP for account recovery. The system lacks protections against automated, high-frequency guessing attacks. Because the OTP space is limited to 10,000 possibilities (0000 through 9999) and the application explicitly does not enforce attempt limits or code expiration, an attacker can programmatically iterate through all possible combinations until the correct code is identified and accepted.
4. Steps to Reproduce
1.	Initiation: Navigate to the password reset page (/forgot_password.php) and request a reset for a target account (e.g., the admin user).

 <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/acebf7fd-55f3-44de-9ff8-dddc2c0ab60f" />

2.	Observation: The application prompts for a 4-digit code on /verify_otp.php. Developer/Lab notes on the page explicitly state: "codes do not expire and there is no attempt limit."
 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d25101a0-055b-4984-91f6-632b12d389ab" />


3.	Interception: Use an interception proxy (like Burp Suite) to capture the POST request submitted to /verify_otp.php. The captured request reveals the parameter otp and the active session cookie (PHPSESSID).
 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0a384c08-8d21-46b3-9712-5ceb27a8b323" />


4.	Exploitation: Create an automated script (e.g., using Python and the requests library) to iterate through values 0000 to 9999. The script sends a POST request for each value using the captured session cookie.
 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3a4d34b7-6577-47dd-8356-165a66791990" />


5.	Validation: The script identifies a successful authentication attempt by analyzing the HTTP response. A valid OTP (in this instance, 1337) results in a 302 Found redirect and a response length of 0, whereas invalid attempts return a 200 OK status with a larger content length containing the error message.
 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9b824c2-3f77-4833-8b8c-b187fe966f11" />
 
6.	Confirmation: Submitting the discovered code (1337) grants access to the successful account recovery page, yielding the flag: FLAG{BRUTE_FORCE_BYPASS_SUCCESS}.

 <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f8232bed-5bac-4b88-989b-193389db70cd" />

4. Impact
This vulnerability allows an unauthenticated attacker to bypass the account recovery security controls and gain unauthorized access to any user account on the portal. Because it was demonstrated against the admin account, the impact includes complete administrative compromise of the application.
5. Remediation Recommendations
To mitigate this vulnerability, the following security controls should be implemented on the OTP verification process:
•	Implement Rate Limiting & Account Lockout: Restrict the maximum number of failed OTP attempts (e.g., 3 to 5 maximum attempts). Once the threshold is reached, invalidate the current OTP and temporarily lock the account recovery feature for that user to stall automated attacks.
•	Implement Time-Based Expiration: Ensure that generated OTPs have a short, strict validity window (e.g., 5 to 10 minutes) before they automatically expire.
•	Increase Code Complexity (Optional but Recommended): Increase the length of the OTP (e.g., 6 or 8 digits) or introduce alphanumeric characters to exponentially increase the time required for a successful brute-force attack.
•	Implement CAPTCHA: Integrate a CAPTCHA mechanism on the verification form to prevent automated scripts from submitting requests.

