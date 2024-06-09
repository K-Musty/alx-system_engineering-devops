# 0x19-postmortem
# Postmortem Report

## Overview

During the deployment of the Alx System Engineering & DevOps project 0x19, an isolated Ubuntu 14.04 container running an Apache web server experienced an outage at around 06:07 GMT. The server returned a 500 Internal Server Error in response to GET requests instead of the expected HTML file for a Holberton WordPress site.

## Incident Timeline

- **06:07 GMT**: Outage detected.
- **19:20 GMT**: Sammie began troubleshooting the issue.

## Debugging Steps

1. **Process Check**:
    - Verified running processes using `ps aux`.
    - Confirmed two `apache2` processes were running: one as `root` and another as `www-data`.

2. **Configuration Check**:
    - Inspected `/etc/apache2/sites-available` directory.
    - Confirmed the web server's document root was `/var/www/html/`.

3. **System Call Tracing**:
    - Used `strace` on the root Apache process's PID while making a `curl` request; found no useful information.
    - Repeated `strace` on the `www-data` process's PID, which showed an `ENOENT (No such file or directory)` error for `/var/www/html/wp-includes/class-wp-locale.phpp`.

4. **File Inspection**:
    - Searched manually in `/var/www/html/` for the erroneous `.phpp` extension using Vim pattern matching.
    - Found the typo in `wp-settings.php` on line 137: `require_once( ABSPATH . WPINC . '/class-wp-locale.php' );`.

5. **Error Correction**:
    - Fixed the typo by removing the extra `p`.

6. **Validation**:
    - Made another `curl` request, which returned a 200 status code.

7. **Automation**:
    - Created a Puppet manifest to automate fixing similar errors in the future.

## Root Cause

The issue was due to a typographical error in the WordPress application. The `wp-settings.php` file mistakenly referenced `class-wp-locale.phpp` instead of `class-wp-locale.php`.

## Resolution

The typo in `wp-settings.php` was corrected by removing the extra `p`, resolving the file path error and restoring the server's functionality.

## Prevention

To avoid similar issues in the future, the following measures are recommended:

1. **Thorough Testing**:
    - Perform comprehensive testing before deploying applications to catch errors early.

2. **Monitoring**:
    - Use a monitoring service like [UptimeRobot](https://uptimerobot.com/) to get instant alerts in case of website downtime.

3. **Automation**:
    - Implement automation tools like Puppet to manage and apply fixes efficiently across environments.


## Conclusion

The outage was caused by a typographical error in a WordPress configuration file. The issue was resolved by correcting the typo, and an automation script was implemented to prevent similar issues in the future. Improved testing procedures and monitoring can help mitigate such problems going forward.

---

