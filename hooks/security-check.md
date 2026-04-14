# Security Check Hook

**Type:** pre
**When:** Before agent execution

Instructs agents to evaluate security concerns during their work. This hook is
prepended to the agent prompt so the agent self-applies these checks.

## Security Requirements

Before producing any code or design artifact, verify the following:

### No Hardcoded Secrets
- No API keys, passwords, tokens, or credentials in source code
- No connection strings with embedded passwords
- All secrets must come from environment variables or secure vaults
- Check for accidental commits of .env files or credential files

### SQL Injection Prevention
- All database queries use parameterized statements or prepared statements
- No string concatenation for SQL query construction
- ORM queries use parameter binding, not string interpolation
- Dynamic table/column names are validated against allowlists

### Cross-Site Scripting (XSS) Prevention
- All user input is sanitized before rendering in HTML
- Use framework-provided escaping mechanisms (not manual regex)
- Content-Security-Policy headers are configured
- Avoid `innerHTML`, `dangerouslySetInnerHTML`, or equivalent unless explicitly sanitized

### OWASP Top 10 Awareness
- Authentication: verify endpoints that need auth are protected
- Authorization: verify role/permission checks on sensitive operations
- Input validation: all external input validated for type, length, format
- Error handling: no stack traces or internal details in error responses
- Logging: sensitive data (passwords, tokens) never logged
- Dependencies: flag known-vulnerable dependency versions if detected

## Application

When this hook is active, the agent must note any security concerns found during
its work and address them proactively. Security violations are blocking issues.
