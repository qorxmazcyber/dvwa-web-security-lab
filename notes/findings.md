# SQL Injection Lab — Initial Observation

## Objective
Understand how DVWA processes user input in a SQL Injection exercise.

## Environment
- Platform: Kali Linux
- Target: Local DVWA instance
- Security Level: Low
- Module: SQL Injection

## Test
- Submitted the User ID value `1`.
- Observed the application's response.

## Observation
The application returned user information for the submitted ID.

## Security Consideration
Applications should validate input and use parameterized SQL queries (prepared statements) to prevent SQL Injection.

## Status
Initial test completed. Further testing and evidence collection are pending.

## Scope
All testing is restricted to the local DVWA laboratory.
## SQL Injection — Low Security Level

### Test Objective
Determine whether user input can alter the SQL query logic in DVWA.

### Test Environment
- Application: DVWA
- Security Level: Low
- Module: SQL Injection
- Target: Local instance (`127.0.0.1`)

### Test Input
`1' OR '1'='1' #`

### Expected Observation
If the input changes the query logic successfully, the application may return multiple user records instead of only the record associated with ID `1`.

### Actual Result
_To be completed after observing the application response._

### Security Impact
SQL Injection can allow unauthorized access to database records when user input is incorporated into SQL queries unsafely.

### Recommended Mitigation
- Use parameterized queries (prepared statements).
- Validate input according to the expected data type.
- Apply least-privilege permissions to database accounts.
- Avoid displaying detailed database errors to users.

### Evidence
`screenshots/sqli-low-test.png`

### Scope
Testing is restricted to the local DVWA laboratory.
