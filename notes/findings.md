# SQL Injection — Security Findings

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

`1' OR '1'='1' #`

### Expected Observation

If the payload successfully changes the SQL query logic, the application may return multiple user records instead of only the record associated with a specific user ID.

### Actual Result

The payload successfully altered the query logic. The application displayed all user records instead of returning only the record associated with the specified user ID.

### Finding

The test demonstrated that user input can modify the intended SQL query logic at the Low security level.

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

## 3. SQL Injection — Medium Security Level

### Objective

Evaluate normal input handling and review the SQL Injection implementation at the Medium security level.

### Test Environment

- Operating System: Kali Linux
- Target Application: DVWA
- Security Level: Medium
- Module: SQL Injection
- Target: Local laboratory (`127.0.0.1`)

### Test Procedure

1. Set the DVWA security level to Medium.
2. Selected User ID `1` from the dropdown and submitted the request.
3. Selected User ID `2` and submitted the request.
4. Observed the application's response for each selection.

### Actual Result

- **ID 1:** The application returned information for one user.
- **ID 2:** The application returned information for one user.

### Observation

Both normal input tests completed successfully. Each selection returned the information associated with a single user.

### Source Code Analysis

The Medium implementation retrieves the submitted value using `$_POST['id']` and applies `mysqli_real_escape_string()`.

The MySQL query is constructed as follows:

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```

The value is concatenated into the SQL query without being enclosed in string quotes. Escaping alone is not an adequate security control for this numeric SQL expression.

### Finding

The source code review identifies a potential SQL Injection risk because user-controlled input is concatenated directly into the SQL query. The normal input tests confirmed expected behavior for IDs `1` and `2`, but did not establish whether SQL Injection is possible or prevented at the Medium security level.

### Recommended Mitigation

- Validate the submitted ID on the server as a positive integer.
- Use prepared statements with bound parameters.
- Avoid exposing detailed database errors to users.
- Encode untrusted values before displaying them in HTML.
- Apply least-privilege permissions to the database account.

### Evidence

- Screenshot: `screenshots/sqli-medium-test.png`
- Source code analysis: `source-code-analysis.md`

### Status

Normal input testing completed. Further controlled security testing is pending.

---

## 4. Scope and Safety

All tests documented in this file were performed against the local DVWA laboratory.

DVWA is intentionally vulnerable and should not be exposed to the public internet. Security testing must only be performed on systems owned by the tester or where explicit authorization has been granted.

---

## 5. Next Steps

- Perform additional controlled tests at the Medium security level.
- Compare the behavior of Low and Medium implementations.
- Study how parameterized queries prevent SQL Injection.
- Document additional vulnerabilities, their impact, and mitigation techniques.
- Capture screenshots to support verified test findings.

