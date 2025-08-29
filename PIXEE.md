# PIXEE.md - Security Remediation Preferences for OWASP Juice Shop

This file defines security remediation preferences for the OWASP Juice Shop application. When vulnerability scanners or security tools identify issues with ambiguous remediation approaches, follow these specific guidelines.

## SQL Injection Vulnerabilities

**Always use parameterized queries or prepared statements.** Never attempt to fix SQL injection through input sanitization, escaping, or blacklisting approaches. 

For this Node.js/Sequelize application:
- Replace raw SQL queries with Sequelize ORM methods
- If raw SQL is absolutely necessary, use Sequelize's `replacements` parameter with bound parameters
- **Never** use string concatenation or template literals to build SQL queries
- Remove any attempts to escape single quotes or filter "dangerous" SQL keywords

**Example**: Replace `SELECT * FROM Users WHERE email = '${email}'` with `SELECT * FROM Users WHERE email = :email` using proper parameter binding.

## Cross-Site Scripting (XSS)

**Implement context-aware output encoding at the point of output, not input.** Do not rely on input sanitization alone.

For this Angular/Express application:
- Use Angular's built-in XSS protections (avoid `innerHTML` and `bypassSecurityTrust` methods)
- Server-side: encode all user data before sending to templates using a library like `html-entities`
- **Never** strip or filter HTML tags from input - encode the output instead
- For rich text requirements, use a well-maintained allowlist-based sanitizer like DOMPurify

## Authentication Bypass

**Implement server-side session validation for every protected resource.** Client-side authentication checks are insufficient.

For JWT-based authentication in this application:
- Validate JWT tokens on every API request requiring authentication
- Check token expiration, signature, and claims server-side
- **Never** rely solely on frontend route guards or client-side token validation
- Implement proper session invalidation on logout
- Use secure, httpOnly cookies for token storage when possible

## NoSQL Injection

**Validate input types and use strict object schemas.** MongoDB injection typically occurs through object injection, not string injection.

For applications using MongoDB or similar NoSQL databases:
- Validate that query parameters are the expected primitive types (string, number, boolean)
- **Never** pass user input directly as query objects
- Use schema validation libraries to enforce expected object structures
- Cast inputs to expected types before database operations

## Authorization/Access Control Issues

**Implement resource-level authorization checks on every API endpoint.** Do not rely on obscurity or client-side access controls.

For this multi-user application:
- Verify that the authenticated user has permission to access the specific resource (not just that they're logged in)
- **Never** trust client-provided resource IDs without server-side validation
- Implement consistent authorization middleware across all API routes
- Log authorization failures for security monitoring

## Path Traversal/Directory Listing

**Use allowlists for file access and disable directory browsing.** Never attempt to filter out traversal sequences.

For file serving in this Express application:
- Use `path.resolve()` and validate that the resolved path is within allowed directories
- **Never** try to filter out `../` sequences - they can be encoded or doubled
- Disable Express static middleware directory listings (`index: false`)
- Store uploaded files outside the web root when possible

## Information Disclosure

**Implement generic error responses and comprehensive logging.** Detailed errors should only appear in server logs.

For error handling in this application:
- Return generic error messages to clients (e.g., "Invalid request")
- Log detailed error information server-side for debugging
- **Never** expose stack traces, database errors, or system paths to users
- Remove or restrict access to debug endpoints in production builds

## Insecure Cryptography

**Use bcrypt for password hashing with a minimum cost factor of 12.** Never use fast hashing algorithms for passwords.

For password storage and session management:
- Replace any MD5, SHA-1, or plain SHA-256 password hashing with bcrypt
- Use a minimum cost factor of 12 for bcrypt (higher for sensitive applications)
- **Never** store plaintext passwords or use reversible encryption for passwords
- Generate cryptographically secure random tokens for sessions and CSRF protection

## Rate Limiting

**Implement progressive rate limiting on authentication and sensitive endpoints.** Use both IP-based and user-based limits.

For this Express application:
- Use express-rate-limit middleware with sliding window counters
- Implement stricter limits on login attempts (5 per 15 minutes per IP)
- **Never** rely solely on client-side throttling
- Consider implementing CAPTCHA after multiple failed attempts

## File Upload Vulnerabilities

**Validate file types by content, not just extension.** Store uploads outside the web root.

For file upload functionality:
- Use libraries like `file-type` to validate actual file content
- Restrict file sizes and implement virus scanning if possible
- **Never** trust the Content-Type header or file extension alone
- Store uploaded files in a location that cannot be directly accessed via URL
- Generate unique filenames to prevent conflicts and information disclosure

## Cross-Site Request Forgery (CSRF)

**Implement CSRF tokens for all state-changing operations.** Use the double-submit cookie pattern for SPA applications.

For this Angular/Express application:
- Generate unique CSRF tokens for each session
- Validate tokens on all POST, PUT, DELETE, and PATCH requests
- **Never** rely solely on checking the Referer header
- Use SameSite cookie attributes as additional protection

---

**General Principle**: When in doubt, choose the most restrictive and secure option. This application is designed to be vulnerable for educational purposes, but all security fixes should implement production-ready security controls.