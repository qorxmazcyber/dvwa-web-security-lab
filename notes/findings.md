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
