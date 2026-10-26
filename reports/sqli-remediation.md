# SQL Injection Remediation Report

## 1. Executive Summary

SQL Injection was successfully demonstrated in the local DVWA environment at the Low security level. The application allowed user input to alter the intended SQL query logic.

This report describes the root cause, recommended remediation, and verification plan.

## 2. Affected Component

- Application: Damn Vulnerable Web Application (DVWA)
- Module: SQL Injection
- Security Level: Low
- Source File: `vulnerabilities/sqli/source/low.php`
- Environment: Local Kali Linux laboratory

## 3. Vulnerability Description

The vulnerable code retrieves user input and inserts it directly into an SQL query.

```php
$id = $_REQUEST['id'];

$query = "SELECT first_name, last_name
          FROM users
          WHERE user_id = '$id';";
```

This allows specially crafted input to alter the intended SQL query logic.

## 4. Root Cause

The application constructs an SQL query using untrusted user input instead of separating query structure from parameter values.

## 5. Recommended Remediation

### 5.1. Use Prepared Statements

The following is an illustrative secure alternative using PHP PDO:

```php
$stmt = $pdo->prepare(
    'SELECT first_name, last_name
     FROM users
     WHERE user_id = :id'
);

$stmt->execute(['id' => $id]);

$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

Prepared statements separate SQL query structure from the supplied parameter value.

The PDO connection must be configured separately. This example is a remediation pattern, not a drop-in replacement for the entire DVWA module.

### 5.2. Validate the User ID

If the application expects a positive integer, validate the input before processing it.

```php
$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null || $id < 1) {
    // Reject invalid input.
}
```

Input validation is useful as an additional safeguard but must not replace prepared statements.

### 5.3. Improve Error Handling

- Do not expose detailed SQL errors to end users.
- Record diagnostic details in protected server logs.
- Return a generic error message when a database operation fails.

### 5.4. Apply Least Privilege

The application's database account should have only the permissions required for normal application functionality.

## 6. Verification Plan

The remediation should be tested in a separate development copy or test application.

1. Submit a valid user ID and confirm that the expected record is returned.
2. Repeat the SQL Injection test that previously altered the query logic.
3. Confirm that the input is treated as a parameter value rather than SQL syntax.
4. Verify that invalid IDs are rejected.
5. Confirm that detailed database errors are not exposed to users.

## 7. Expected Outcome

After correct implementation, user input should not be able to alter the intended SQL query logic. Normal valid requests should continue to work.

**Verification status:** Pending implementation and testing of the remediation in a separate test application.

## 8. Evidence

- Vulnerability test: `screenshots/sqli-low-test.png`
- Source code analysis: `notes/source-code-analysis.md`

## 9. Conclusion

Parameterized queries are the primary recommended defense against SQL Injection. Input validation, secure error handling, and least-privilege database permissions provide additional layers of protection.

## 10. Scope

This report documents educational testing performed against the local DVWA laboratory. No external systems were targeted.

## Medium Security Level — Remediation Analysis

### 1. Identified Issue

The Medium security level applies `mysqli_real_escape_string()` to the submitted ID. However, the resulting value is concatenated directly into the SQL query:

```php
$id = mysqli_real_escape_string(
    $GLOBALS["___mysqli_ston"],
    $id
);

$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```

The ID is inserted without SQL string quotes. Escaping alone is not a sufficient defense for this numeric expression. The application should validate the expected input type and use a parameterized query.

### 2. Recommended Fix

Use server-side input validation and a MySQLi prepared statement. The following is an illustrative example for the MySQL branch, not a drop-in replacement for the original DVWA file.

```php
$id = filter_var(
    $_POST['id'] ?? null,
    FILTER_VALIDATE_INT
);

if ($id === false || $id === null || $id < 1) {
    http_response_code(400);
    exit('Invalid input');
}

$stmt = $GLOBALS["___mysqli_ston"]->prepare(
    "SELECT first_name, last_name
     FROM users
     WHERE user_id = ?"
);

$stmt->bind_param("i", $id);
$stmt->execute();

$result = $stmt->get_result();

while ($row = $result->fetch_assoc()) {
    $first = htmlspecialchars(
        $row['first_name'],
        ENT_QUOTES,
        'UTF-8'
    );

    $last = htmlspecialchars(
        $row['last_name'],
        ENT_QUOTES,
        'UTF-8'
    );

    echo "<pre>First name: {$first}\nSurname: {$last}</pre>";
}

$stmt->close();
```

**Why this helps:**

- Input validation rejects missing, invalid, or non-positive IDs.
- Prepared statements keep the parameter separate from the SQL query structure.
- `htmlspecialchars()` encodes returned values for HTML text output.
- Generic user-facing errors reduce unnecessary disclosure of database details.

The SQLite branch must use its own parameter-binding API if that database backend is enabled. The `get_result()` method in this MySQLi example requires PHP's `mysqlnd` driver.

### 3. Additional Security Controls

- Do not expose raw database errors to users; log diagnostic details securely on the server.
- Use a database account with only the permissions the application requires.
- Keep database credentials out of GitHub and never commit `config.inc.php`.
- Validate input on the server even when the interface uses a dropdown.
- Preserve the original vulnerable DVWA code for controlled educational testing; keep secure examples in documentation or a separate demonstration file.

### 4. Verification Plan

After implementing the fix in a separate test application or authorized test copy:

1. Submit a valid existing positive integer ID and confirm the expected record is returned.
2. Submit a nonexistent positive integer ID and confirm that no unrelated records are returned.
3. Submit empty, malformed, negative, and non-integer values and confirm that they are rejected.
4. Run authorized SQL Injection test cases in the isolated lab and verify that input cannot alter the query's intended structure.
5. Check that HTML output is encoded and that detailed database errors are not exposed to users.

### 5. Current Status

The Medium source code has been reviewed and the recommended mitigations have been documented. The remediation example has not been applied to the original DVWA source, and the fix has not yet been independently verified. Testing results should be recorded only after the checks above have actually been performed.

### 6. Conclusion

The Medium implementation adds escaping, but still constructs its SQL query by concatenating user-controlled input. Server-side validation and prepared statements are the primary recommended controls. Output encoding, safe error handling, and least-privilege database access provide additional layers of protection.
