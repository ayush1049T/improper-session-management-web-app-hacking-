# Vulnerability Assessment Report (SQL injection)

Target: SecureCorp Portal (192.168.1.14)
Vulnerability Type: SQL Injection (UNION-Based) – Employee Directory Search
Severity: Critical

1. Executive Summary

A critical SQL Injection vulnerability was identified in the Employee Directory search feature of the SecureCorp Portal. The search parameter is concatenated directly into a backend MySQL/MariaDB query without sanitization or parameterization. This allows an attacker to manipulate the underlying SQL statement using UNION-based injection to enumerate database schema information and extract sensitive data, including internal usernames, directly from the users table.

2. Vulnerability Description
The endpoint /search.php accepts a q parameter used to query the internal employee directory by name. User input is passed into the SQL query without escaping or the use of parameterized statements. Submitting a single quote (') into the search field breaks the query's string context and triggers a MySQL syntax error, confirming the field is directly interpolated into the SQL statement. By appending SQL comment syntax (-- -) after a crafted UNION SELECT clause, an attacker can append arbitrary SELECT statements to the original query, causing the application to render attacker-controlled data (including live database contents) inside the normal employee card UI.

4. Steps to Reproduce
   
1.	Baseline Injection Test: Navigate to the Employee Directory search (/search.php) and submit the payload '-- - in the search field. The query executes without error and the directory falls back to displaying its default full listing (admin, abhishek, john), confirming the input is being concatenated into the SQL statement and that the trailing comment successfully neutralizes the rest of the original query.

<img width="859" height="483" alt="image" src="https://github.com/user-attachments/assets/106e2aa3-2b93-4dae-967a-ba28a90f2bd3" />

2.	Comment Syntax Confirmation: An alternate comment style, '#, is tested and produces the same default result set, further confirming the injection point and that the backend is MySQL/MariaDB (as also indicated by the application's own footer label).

<img width="859" height="483" alt="image" src="https://github.com/user-attachments/assets/423f9822-da93-45c4-9227-a67f9099aacb" />
 
3.	Column Count Enumeration: The payload ' UNION SELECT NULL,NULL,NULL-- - is submitted. The query executes successfully and an additional “N/A” card is rendered alongside the default employee results, confirming the original query returns exactly three columns and that the injected UNION SELECT is column-count compatible.

 <img width="859" height="483" alt="image" src="https://github.com/user-attachments/assets/38461935-1f0a-4052-a622-d2f1a3a6a577" />

4.	Schema Enumeration – Tables: Using the confirmed 3-column UNION, the payload ' UNION SELECT table_name,NULL,NULL FROM information_schema.tables-- - is submitted. This dumps the names of every table visible to the database user, including MySQL system views (e.g. ADMINISTRABLE_ROLE_AUTHORIZATIONS, APPLICABLE_ROLES, CHARACTER_SETS) as well as application-specific tables.

 <img width="859" height="483" alt="image" src="https://github.com/user-attachments/assets/bad3fa30-ce26-4c7a-b27c-ee6dd91e5812" />

5.	Schema Enumeration – Columns: Targeting the application's users table, the payload ' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'-- - is submitted. This reveals the table's column structure: id, username, role, sid, and department.

 <img width="859" height="483" alt="image" src="https://github.com/user-attachments/assets/686842df-9bbe-42c2-b0c0-dffde910c903" />

6.	Data Extraction: With the column names known, the payload ' UNION SELECT username,id,NULL FROM users-- - is submitted. The response discloses the actual contents of the users table — usernames (admin, abhishek, john) alongside their internal numeric IDs (1, 2, 3) — confirming full read access to sensitive application data via the injection point.

<img width="859" height="483" alt="image" src="https://github.com/user-attachments/assets/75ee5c2e-12c9-470b-8e3c-43120266ab7b" />

4. Impact

This vulnerability allows an unauthenticated or low-privileged attacker to read arbitrary data from the application's database, including the full contents of the information_schema (exposing the complete database structure) and the users table. Depending on additional columns not yet enumerated (e.g. password hashes, tokens, or PII in other tables), this could lead to full credential compromise, privilege escalation, and complete loss of confidentiality for all data stored in the database.

6. Remediation Recommendations

•	Use Parameterized Queries / Prepared Statements: Replace all direct string concatenation of user input into SQL statements with parameterized queries (e.g. PDO prepared statements in PHP) so user input is always treated as data, never executable SQL.
•	Apply the Principle of Least Privilege: Ensure the database account used by the web application has the minimum privileges required (e.g. no access to information_schema or other applications' tables) to limit the blast radius of any future injection flaw.
•	Input Validation: Enforce strict allow-list validation on the search field (e.g. restrict to expected characters for names) as a defense-in-depth measure alongside parameterized queries.
•	Generic Error Handling: Disable verbose SQL error messages in production responses; return a generic error page instead of raw SQLSTATE/driver error text that reveals database type and query structure.
•	Web Application Firewall (WAF): Deploy a WAF rule set to detect and block common SQL injection payloads (UNION SELECT, information_schema references, comment sequences) as an additional layer of defense.
