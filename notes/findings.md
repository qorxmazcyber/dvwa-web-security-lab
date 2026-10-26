# DVWA Security Findings

## 1. SQL Injection — Initial Observation

### Objective
Understand how DVWA processes user input during a SQL Injection exercise.

### Environment
- Platform: Kali Linux
- Target Application: DVWA (Damn Vulnerable Web Application)
- Security Level: Low
- Module: SQL Injection
- Target URL: `http://127.0.0.1/dvwa/`

### Test
Submitted the User ID value `1` and observed the application's response.

### Observation
The application returned user information associated with the submitted ID.

### Status
Completed.

---

## 2. SQL Injection — Low Security Level

### Test Objective
Determine whether user input can alter the intended SQL query logic in DVWA.

### Test Environment
- Operating System: Kali Linux
- Target Application: DVWA
- Security Level: Low
- Module: SQL Injection
- Target: Local laboratory (`127.0.0.1`)

### Test Input
```sql
1' OR '1'='1' #
```

### Expected Observation
If the payload successfully changes the SQL query logic, the application may return multiple user records instead of only the record associated with a specific user ID.

### Actual Result
The payload successfully altered the query logic. The application displayed all user records instead of returning only the record associated with the specified user ID.

### Finding
The SQL Injection test demonstrated that user input can modify the intended SQL query logic at the Low security level.

### Security Impact
SQL Injection may allow unauthorized retrieval of database records. Depending on the application's database permissions and query structure, the impact may extend to other database operations.

### Recommended Mitigation
- Use parameterized queries (prepared statements).
- Validate input according to the expected data type.
- Apply the principle of least privilege to database accounts.
- Avoid displaying detailed database errors to users.
- Perform security testing at multiple application security levels.

### Evidence
- Screenshot: `screenshots/sqli-low-test.png`

### Status
Completed in the local DVWA laboratory.

---

## 3. Scope and Safety

All tests documented in this file were performed against the local DVWA laboratory.

DVWA is intentionally vulnerable and should not be exposed to the public internet. Security testing must only be performed on systems owned by the tester or where explicit authorization has been granted.

---

## 4. Next Steps

- Compare SQL Injection behavior at different DVWA security levels.
- Study how parameterized queries prevent SQL Injection.
- Document additional vulnerabilities, their impact, and mitigation techniques.
- Capture screenshots to support test findings.

## 3. SQL Injection — Medium Security Level

### Objective
Observe how DVWA handles user input at the Medium security level.

### Test Environment
- Application: DVWA
- Security Level: Medium
- Module: SQL Injection
- Target: Local DVWA instance (`127.0.0.1`)

### Test
Selected User ID `1` from the dropdown list and submitted the request.

### Actual Result
The application returned the user information associated with the selected ID.

### Observation
The User ID field is presented as a dropdown containing numeric options. The normal request completed successfully.

### Security Consideration
A dropdown restricts the values selected through the normal browser interface, but this alone does not prove that the server-side code is secure against SQL Injection.

### Status
Normal input test completed. Further security analysis is pending.
