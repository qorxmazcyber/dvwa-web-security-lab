# SQL Injection — Source Code Analysis

## 1. Objective

Analyze the source code of DVWA's SQL Injection module at the Low security level, identify the root cause of the vulnerability, and understand how to prevent it.

## 2. Environment

- Operating System: Kali Linux
- Target Application: Damn Vulnerable Web Application (DVWA)
- Module: SQL Injection
- Security Level: Low
- Source File: `vulnerabilities/sqli/source/low.php`
- Target: Local laboratory (`127.0.0.1`)

## 3. Vulnerable Code

The following code retrieves the user-supplied ID:

```php
$id = $_REQUEST['id'];
```

The application then constructs an SQL query:

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

## 4. Code Analysis

### 4.1. User Input Handling

```php
$id = $_REQUEST['id'];
```

The application reads the `id` parameter from the request without validating that it contains an expected value.

`$_REQUEST` may contain data from different request sources, depending on PHP configuration. Explicitly using the expected request method and validating input improves clarity and control.

### 4.2. Unsafe SQL Query Construction

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

The user-controlled value is inserted directly into the SQL query string.

Because the input is enclosed in SQL quotes without appropriate protection, specially crafted input can change the intended query logic.

### 4.3. Root Cause

The root cause is the use of string concatenation or interpolation to construct an SQL query using untrusted input, without parameterized query handling.

### 4.4. Observed Result

During testing at the Low security level, the payload below caused the application to display all user records:

```sql
1' OR '1'='1' #
```

This demonstrated that user input could alter the intended SQL query logic in the local DVWA laboratory.

### 4.5. Security Impact

Depending on the query and database permissions, SQL Injection may allow an attacker to retrieve records beyond those intended by the application. Other potential impacts depend on the database operations available to the vulnerable code.

## 5. Recommended Remediation

### 5.1. Use Prepared Statements

Use parameterized queries so that user input is handled as a value rather than being interpreted as part of the SQL syntax.

Example using PHP PDO:

```php
$stmt = $pdo->prepare(
    'SELECT first_name, last_name
     FROM users
     WHERE user_id = :id'
);

$stmt->execute(['id' => $id]);

$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

This is an illustrative secure-code example. The PDO connection must be configured separately, and the application must render returned values safely.

### 5.2. Validate Input

If the application expects a positive integer, validate the input accordingly before using it.

Example:

```php
$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null || $id < 1) {
    // Reject invalid input.
}
```

Input validation is an additional safeguard; it does not replace prepared statements.

### 5.3. Handle Errors Securely

Do not expose detailed database errors to end users. Record diagnostic details in protected server logs and display a generic error message in the application.

### 5.4. Apply Least Privilege

The application's database account should have only the permissions required for its intended functionality.

## 6. Verification Plan

After implementing a fix in a separate test application or controlled development copy:

1. Verify that a valid user ID returns the expected record.
2. Repeat the previously successful SQL Injection test.
3. Confirm that user input cannot change the intended query logic.
4. Confirm that invalid IDs are rejected appropriately.
5. Check that detailed database errors are not exposed to the user.

The original DVWA source file is intentionally vulnerable and should remain unchanged for laboratory exercises.

## 7. Conclusion

The Low security level demonstrates how directly embedding untrusted input in an SQL query can introduce SQL Injection. Prepared statements, appropriate input validation, secure error handling, and least-privilege database permissions are important defensive measures.

## 8. Scope

All testing was performed against the local DVWA laboratory. The findings are for educational purposes and do not represent an assessment of any external application.
