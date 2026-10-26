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

## Medium Security Level — Source Code Analysis

### 1. Overview

The Medium security level introduces additional input handling compared with the Low security level. The application reads the submitted ID from the POST request and applies `mysqli_real_escape_string()` before constructing the SQL query.

However, escaping input does not automatically make a dynamically constructed SQL query safe. The effectiveness of escaping depends on the context in which the value is used.

### 2. Input Handling

The application checks whether the Submit parameter exists and then retrieves the submitted ID:

```php
if (isset($_POST['Submit'])) {
    $id = $_POST['id'];
}
```

**Observation:** The value comes from user-controlled input. A dropdown in the browser is not a security boundary because a client can modify an HTTP request.

**Recommendation:** Validate input on the server. If the ID must be a positive integer, reject values that do not meet that requirement.

### 3. Input Escaping

The code applies:

```php
$id = mysqli_real_escape_string(
    $GLOBALS["___mysqli_ston"],
    $id
);
```

This function escapes certain special characters according to the database connection's character set and SQL rules. It is commonly used when values are inserted into SQL string literals.

In this implementation, however, the ID is inserted into the query without surrounding quotes:

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```

This means escaping alone is not an adequate security control for the numeric SQL expression. The application should validate the expected data type and use a parameterized query.

### 4. SQL Query Analysis

The query retrieves the first and last names of a record whose `user_id` matches the supplied value.

The main concern is that user input is concatenated directly into SQL syntax rather than being passed as a separate parameter.

**Potential impact:** If an attacker can influence the SQL expression, the query may return unintended records or behave differently from what the application intended. The actual impact must be assessed through authorized testing in the local DVWA lab.

### 5. Error Handling

The MySQL branch includes database error details in the response. The SQLite branch also displays exception or database error information.

Detailed database errors can disclose implementation details. Production applications should log diagnostic information securely on the server and return a generic error message to the user.

### 6. Output Encoding

Database values are inserted into HTML output. SQL Injection and Cross-Site Scripting (XSS) are different vulnerabilities, but output encoding is still important.

When displaying untrusted text in HTML, use context-appropriate encoding, such as PHP's `htmlspecialchars()` for HTML text.

### 7. Recommended Mitigations

- Validate the submitted ID on the server as a positive integer.
- Use prepared statements with bound parameters instead of concatenating input into SQL.
- Avoid displaying raw database errors to end users.
- Encode untrusted values before rendering them in HTML.
- Apply least-privilege permissions to the application's database account.
- Test the fix with valid input, invalid input, and controlled security test cases.

### 8. Illustrative Secure MySQL Example

The following example demonstrates the recommended approach. It is an educational example, not a drop-in replacement for DVWA's original source file.

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

This example is specifically for the MySQLi branch. The SQLite branch requires a separate implementation using SQLite's parameter-binding API. The `get_result()` method also requires PHP's `mysqlnd` driver.

### 9. Testing Status

The source code has been reviewed. The code structure indicates that escaping is used, but the ID is still concatenated into the SQL query without quotes.

A practical Medium-level test result should only be documented after the behavior has been verified in the local DVWA lab. No successful bypass or complete remediation is claimed by this source review alone.

### 10. Conclusion

The Medium security level adds input escaping compared with the Low implementation, but the query still relies on dynamically concatenated input. Server-side validation and prepared statements are the recommended primary controls. The original DVWA source remains unchanged for controlled educational testing.
