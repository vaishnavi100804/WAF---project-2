# WAF_Rule_Development_Lab_Env

# WAF Rule Development Lab

A simple lab setup to learn Web Application Firewall (WAF) rule development using ModSecurity and block basic SQL Injection attacks.

## Overview

This lab helps you understand how WAF rules work by setting up ModSecurity and creating custom rules to detect and block SQL injection attempts.

## Prerequisites

- Ubuntu/Debian system
- Apache web server
- Basic command line knowledge

## Installation

### 1. Install Apache and ModSecurity

```bash
# Update system
sudo apt update

# Install Apache
sudo apt install apache2

# Install ModSecurity
sudo apt install libapache2-mod-security2

# Enable modules
sudo a2enmod security2
sudo systemctl restart apache2
```

### 2. Configure ModSecurity

```bash
# Copy default configuration
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

# Edit configuration
sudo nano /etc/modsecurity/modsecurity.conf
```

Change the following line:
```
SecRuleEngine DetectionOnly
```
to:
```
SecRuleEngine On
```

### 3. Create a Test Web Application

Create a simple PHP login page to test against:

```bash
sudo nano /var/www/html/login.php
```

```php
<?php
if ($_POST) {
    $username = $_POST['username'];
    echo "Login attempt for: " . htmlspecialchars($username);
}
?>
<form method="post">
    Username: <input type="text" name="username"><br>
    Password: <input type="password" name="password"><br>
    <input type="submit" value="Login">
</form>
```

## Basic SQL Injection Rule

### Create Custom Rule

```bash
sudo nano /etc/modsecurity/rules/01-sql-injection.conf
```

Add this basic SQL injection rule:

```
# Basic SQL Injection Detection
SecRule ARGS "@detectSQLi" \
    "id:1001,\
    phase:2,\
    deny,\
    status:403,\
    msg:'SQL Injection Attack Detected',\
    logdata:'Matched Data: %{MATCHED_VAR} found within %{MATCHED_VAR_NAME}'"
```

### Test the Rule

1. Restart Apache:
```bash
sudo systemctl restart apache2
```

2. Test SQL injection attempt:
```bash
# This should be blocked
curl -X POST http://localhost/login.php -d "username=admin' OR '1'='1"
```

## Monitoring

Check ModSecurity logs:
```bash
sudo tail -f /var/log/apache2/modsec_audit.log
```

## Common SQL Injection Patterns to Test

- `admin' OR '1'='1`
- `admin' UNION SELECT * FROM users`
- `admin'; DROP TABLE users--`
- `1' OR '1'='1' --`

## Next Steps

- Experiment with more complex rules
- Test different attack vectors
- Learn about false positives and tuning
- Explore the OWASP ModSecurity Core Rule Set

## Resources

- [ModSecurity Documentation](https://github.com/SpiderLabs/ModSecurity)
- [OWASP ModSecurity Core Rule Set](https://coreruleset.org/)
- [Web Application Firewall Evasion Techniques](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF)

## Disclaimer

This lab is for educational purposes only. Use only on systems you own or have permission to test.
