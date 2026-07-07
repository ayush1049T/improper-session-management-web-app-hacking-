## XSS(cross-site-scripting)

**Target:** SecureCorp Portal / Lab Environment

**Vulnerability Type:** Reflected Cross-Site Scripting (XSS)

**Severity:** Medium to High (Depending on session configurations)

### 1. Executive Summary

A Cross-Site Scripting (XSS) vulnerability was identified within the web application's input handling mechanisms. The application fails to properly sanitize or encode user-supplied input before rendering it back in the HTTP response. This allows an attacker to inject and execute arbitrary JavaScript code within the context of the victim's browser session, potentially leading to session hijacking, defacement, or unauthorized actions on behalf of the user.

### 2. Vulnerability Description

Cross-Site Scripting occurs when an application includes untrusted data in a web page without proper validation or escaping. In this specific scenario, a user-controlled input field (such as a search box, comment section, or URL parameter) accepts raw input and reflects it directly into the HTML document object model (DOM).

Because the input is not neutralized via HTML entity encoding, the web browser interprets the submitted text string as executable code (scripts), triggering an immediate execution when the page loads.

### 3. Steps to Reproduce (Conceptual)

1. **Identification:** Locate an input parameter or injection point that reflects user text back onto the webpage interface.
2. **Payload Submission:** Input a standard proof-of-concept script tag designed to verify execution without causing operational disruption:
```html
<script>alert(1)</script>

```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/df015598-ef95-4609-82e3-f7559d51f139" />


3. **Execution:** Submit the form or request. The browser processes the returned HTML template, parsing the injected `<script>` tags as standard executable commands, causing a modal alert dialog box to pop up.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a9563361-e51d-4c18-8216-87850d4efaee" />


5. **Flag Retrieval:** Upon execution within the vulnerable challenge context, the application validates the active JavaScript session execution and displays the corresponding completion token or flag.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ed655d49-ffed-4006-bf52-4511f999f8d5" />


### 4. Impact

The execution of arbitrary JavaScript within a victim's browser context can lead to severe security compromises, including:

* **Session Hijacking:** Accessing and exfiltrating session tokens or cookies (e.g., `document.cookie`), allowing attackers to impersonate authorized users.
* **Credential Theft:** Injecting fake login forms or dialogue boxes to harvest user credentials via social engineering.
* **Malicious Redirection:** Automatically redirecting users to external phished or malicious domains.
* **Data Exfiltration:** Reading sensitive information displayed on the page that the user has authorization to view.

### 5. Remediation Recommendations

To protect the application against script injection, implement robust input handling and contextual output encoding strategies:

* **Context-Aware Output Encoding:** Ensure all user-supplied data is properly encoded before being rendered into the HTML body, attributes, JavaScript variables, or CSS. For HTML context, convert characters such as `<`, `>`, `&`, `"`, and `'` into their respective HTML entities (e.g., `&lt;`, `&gt;`).
* **Implement a Content Security Policy (CSP):** Configure a strong CSP HTTP header to restrict the sources from which scripts can be executed and prevent the execution of inline scripts (e.g., prohibiting `unsafe-inline`).
* **Input Validation:** Employ strict white-list validation on all incoming data to ensure it conforms to expected formats (alphanumeric, specific lengths, etc.) before processing.
* **Utilize Modern Frameworks:** Rely on modern web development frameworks (like React or Angular) that inherently handle contextual auto-escaping by default.
