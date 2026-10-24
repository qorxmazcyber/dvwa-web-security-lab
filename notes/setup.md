# DVWA Lab Setup

## 1. Objective

This document describes the installation and configuration of Damn Vulnerable Web Application (DVWA) in a local Kali Linux laboratory.

## 2. Lab Environment

- Operating System: Kali Linux
- Web Server: Apache
- Database: MariaDB
- Server-side Language: PHP
- Application: DVWA
- Local URL: `http://127.0.0.1/dvwa/`

## 3. Start Required Services

Start Apache and MariaDB:

```bash
sudo systemctl start apache2
sudo systemctl start mariadb
```

Verify their status:

```bash
systemctl status apache2
systemctl status mariadb
```

## 4. Download DVWA

Clone the official DVWA repository into the local web server directory:

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa
```

If DVWA is already installed, do not clone it again.

## 5. Configure DVWA

Enter the DVWA directory:

```bash
cd /var/www/html/dvwa
```

Create the configuration file from the provided template:

```bash
sudo cp config/config.inc.php.dist config/config.inc.php
```

Configure the database settings in `config/config.inc.php` to match the local MariaDB database and its dedicated user.

Do not upload database passwords or sensitive configuration files to GitHub.

## 6. Configure the Database

Open the MariaDB shell:

```bash
sudo mariadb
```

Create the database and a dedicated user if they do not already exist. Use a strong local lab password and configure the application to use the same credentials.

Example SQL commands for a new local lab:

```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'REPLACE_WITH_A_STRONG_PASSWORD';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

If the database and user already exist, do not recreate them blindly. Verify the existing configuration first.

## 7. Initialize the Database

1. Open `http://127.0.0.1/dvwa/setup.php` in the browser.
2. Review the setup status.
3. Select **Create / Reset Database**.
4. Verify that the application opens successfully.

## 8. Verification

The DVWA application opened successfully in the browser after the database initialization process.

## 9. Security Notes

- Keep DVWA restricted to the local laboratory.
- Do not expose the intentionally vulnerable application to the public internet.
- Never commit `config.inc.php`, passwords, tokens, or other secrets.
- Perform security testing only on systems for which explicit authorization has been granted.

## 10. Next Steps

- Study SQL Injection in the local lab.
- Study Cross-Site Scripting (XSS).
- Document findings, impact, and mitigation methods.
- Create technical reports supported by screenshots.


