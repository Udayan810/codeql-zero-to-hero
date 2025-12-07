# Security Vulnerabilities Report

This report documents security vulnerabilities identified in the Pizza Bubonja's Flask application through manual code analysis.

## Summary

The application contains several security vulnerabilities that should be addressed before production deployment.

## Identified Vulnerabilities

### 1. Hardcoded Credentials and Plain Text Password Storage
- **Location**: `app.py` lines 22-27 (FIXED)
- **Issue**: Gmail username and password are stored in plain text files (`username.txt`, `password.txt`) and read directly into variables
- **Risk**: If these files are compromised, attackers gain access to the Gmail account
- **Severity**: High
- **Status**: ✅ **FIXED** - Credentials now loaded from environment variables (`GMAIL_USERNAME`, `GMAIL_PASSWORD`). Plain text files no longer used.

### 2. No Input Validation or Sanitization
- **Location**: `send_email` route in `app.py` (FIXED)
- **Issue**: Form data (`name`, `email`, `subject`, `message`) is used directly without validation
- **Risk**: Potential injection attacks, malformed emails, or XSS if displayed
- **Severity**: Medium
- **Status**: ✅ **FIXED** - Implemented WTForms validation with length limits, email format validation, and required field checks.

### 3. Debug Mode Enabled in Production
- **Location**: `app.py` line 38 (FIXED)
- **Issue**: `app.run(debug=True)` exposes detailed error information
- **Risk**: Information disclosure, potential code execution in debug console
- **Severity**: High (in production environment)
- **Status**: ✅ **FIXED** - Debug mode now controlled by `FLASK_DEBUG` environment variable, defaulting to False.

### 4. No CSRF Protection
- **Location**: All routes in `app.py` (FIXED)
- **Issue**: Flask-WTF or similar CSRF protection not implemented
- **Risk**: Cross-Site Request Forgery attacks on forms
- **Severity**: Medium
- **Status**: ✅ **FIXED** - Implemented Flask-WTF CSRF protection. Note: Templates need to be updated to include `{{ csrf_token() }}` in forms.

### 5. Broad Exception Handling
- **Location**: `send_email` route in `app.py` line 35 (FIXED)
- **Issue**: Generic `except:` clause catches all exceptions, including potential security-related ones
- **Risk**: Security exceptions might be silently ignored
- **Severity**: Low
- **Status**: ✅ **FIXED** - Replaced with specific exception handling for SMTP errors, with proper logging for unexpected errors.

### 6. No Rate Limiting
- **Location**: `send_email` route (FIXED)
- **Issue**: No protection against email spam or abuse
- **Risk**: Email service can be abused for spam
- **Severity**: Medium
- **Status**: ✅ **FIXED** - Implemented Flask-Limiter with 5 emails per hour limit on send_email route, plus global limits.

### 7. Database Schema Present but Not Used
- **Location**: `schema.sql`
- **Issue**: Database schema exists but app doesn't connect to database
- **Risk**: If database is later implemented without proper security, SQL injection vulnerabilities
- **Severity**: Low (currently not exploitable)
- **Status**: ⚠️ **NOT ADDRESSED** - Database integration not implemented yet. When implemented, use SQLAlchemy with parameterized queries.

### 8. Missing HTTPS Enforcement
- **Location**: Application configuration
- **Issue**: No HTTPS redirection or secure headers
- **Risk**: Man-in-the-middle attacks on email credentials
- **Severity**: High
- **Status**: ⚠️ **PARTIALLY ADDRESSED** - HTTPS enforcement requires production deployment configuration (e.g., reverse proxy). Added recommendation for production setup.

## Recommendations

1. **Use environment variables or secure credential management** instead of plain text files
2. **Implement input validation** using WTForms or similar
3. **Disable debug mode** in production
4. **Add CSRF protection** with Flask-WTF
5. **Implement proper error handling** and logging
6. **Add rate limiting** to prevent abuse
7. **Enforce HTTPS** and use secure email protocols
8. **Consider implementing the database** with proper ORM (SQLAlchemy) and parameterized queries

## Analysis Method

This report was generated through manual code review. CodeQL analysis was attempted but encountered issues with query pack availability. The analysis focused on common Flask application security patterns and best practices.

## Next Steps

- Address high-severity vulnerabilities before production deployment
- Implement security testing in CI/CD pipeline
- Consider security code review for future changes
- Implement proper logging and monitoring
