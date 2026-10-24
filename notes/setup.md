# DVWA Lab Setup

## 1. Objective

The objective of this lab is to install and configure Damn Vulnerable Web Application (DVWA) in a local Kali Linux environment for educational web security testing.

## 2. Lab Environment

- Operating System: Kali Linux
- Web Server: Apache
- Server-side Language: PHP
- Database: MariaDB
- Target Application: DVWA
- Access URL: `http://127.0.0.1/dvwa/`

## 3. Setup Steps

1. Started the Apache web server.
2. Started the MariaDB database service.
3. Downloaded DVWA into the local web server directory.
4. Created the DVWA configuration file from the provided example.
5. Created the `dvwa` database and configured its database user.
6. Updated the database connection settings in the DVWA configuration file.
7. Opened the DVWA setup page in the browser.
8. Used the **Create / Reset Database** option to initialize the database.
9. Verified that the DVWA application opened successfully.

## 4. Verification

**Status:** Completed

DVWA opened successfully in the browser after the database initialization process.

## 5. Security Notes

- DVWA is intentionally vulnerable and must be used only in an isolated local lab.
- Do not expose this application to the public internet.
- Do not upload database passwords or sensitive configuration files to GitHub.
- Perform testing only against systems for which explicit authorization has been granted.

## 6. Next Steps

- Capture screenshots of the working DVWA application.
- Study SQL Injection in the local lab.
- Study Cross-Site Scripting (XSS).
- Document findings, possible impact, and mitigation methods.

