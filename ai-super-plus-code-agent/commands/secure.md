---
name: /secure
description: >
  Security audit. OWASP Top 10 scan, dependency vulnerability check, secrets detection,
  security header verification, auth flow analysis. Produces security audit report with
  findings and hardening recommendations.
---

# Secure Command

## Purpose
Perform comprehensive security audit across application, dependencies, secrets, and infrastructure.

## When to Use
- Pre-deployment security verification
- Vulnerability response when CVE published
- Compliance audit (SOC2, HIPAA, PCI)
- Regular security assessment cycle
- After security incident
- New feature security review

## Execution Steps

### 1. OWASP Top 10 Assessment

**A1: Injection**
- **SQL Injection**: Check for parameterized queries
- **NoSQL Injection**: Verify query parameter handling
- **Command Injection**: Check system() call protection
- **LDAP Injection**: Verify LDAP query safety
- **Path Traversal**: Check path validation
- **Test**: Attempt injection with test payloads

**A2: Broken Authentication**
- **Password Storage**: Verify bcrypt/Argon2 hashing
- **Session Management**: Check cookie flags (HttpOnly, Secure, SameSite)
- **Password Policy**: Minimum length, complexity requirements
- **Password Reset**: Secure token generation and validation
- **Multi-Factor Auth**: MFA implementation present
- **Account Lockout**: Protection against brute force
- **Session Timeout**: Appropriate timeouts configured

**A3: Sensitive Data Exposure**
- **HTTPS**: TLS 1.2+ enforced everywhere
- **Encryption**: Data encrypted at rest (AES-256)
- **Key Management**: Secure key storage and rotation
- **PII Protection**: Sensitive data masked in logs
- **Secure Transport**: No plaintext transmission
- **Backup Encryption**: Backups encrypted
- **Mobile**: Secure mobile app communication

**A4: XML External Entities (XXE)**
- **XML Parsing**: External entities disabled
- **DTD Disabling**: DOCTYPE declarations disabled
- **File Upload**: XML file handling secure
- **Dependencies**: XML libraries patched
- **Content-Type**: Validation of content types
- **Parser Config**: Secure parser configuration

**A5: Broken Access Control**
- **Authorization**: Verify permissions on all endpoints
- **Resource Level**: Check object ownership before access
- **RBAC**: Role-based access control implemented
- **API Scopes**: Token permissions limited to scope
- **Public Endpoints**: Intentional public APIs only
- **Admin Pages**: Explicit admin role required
- **Audit Logging**: Permission decisions logged

**A6: Security Misconfiguration**
- **Default Credentials**: No default passwords
- **Unused Services**: Disable unused services
- **Directory Listing**: Disable directory indexing
- **Error Messages**: Generic error messages in production
- **Security Headers**: All headers configured
- **Framework Updates**: Latest patches applied
- **Admin Interfaces**: Protected/internal only

**A7: Cross-Site Scripting (XSS)**
- **Output Encoding**: HTML encode all user content
- **Content Security Policy**: Strict CSP headers
- **Input Validation**: Server-side validation
- **DOM Manipulation**: No innerHTML with user data
- **Sanitization**: DOMPurify for user HTML
- **Framework Protection**: Use framework XSS protections
- **JavaScript Encoding**: Context-appropriate encoding

**A8: Insecure Deserialization**
- **Object Validation**: Validate deserialized objects
- **Type Checking**: Enforce expected types
- **Safe Serialization**: Use JSON, not Pickle/Marshal
- **Version Control**: Lock library versions
- **Signed Data**: HMAC verify serialized data
- **Whitelisting**: Only deserialize known types

**A9: Using Components with Known Vulnerabilities**
- **Dependency Audit**: npm audit or equivalent
- **Version Check**: Identify outdated packages
- **CVE Scanning**: Check for published CVEs
- **Upgrade Plan**: Plan for security updates
- **Unused Removal**: Remove unused dependencies
- **Monitoring**: Runtime vulnerability detection

**A10: Insufficient Logging & Monitoring**
- **Security Events**: Log auth failures, permission denials
- **Structured Logging**: Consistent log format
- **Centralized Logging**: Logs sent to SIEM
- **Alerting**: Real-time alert for security events
- **Retention**: 90+ day log retention
- **Audit Trail**: Non-repudiation of actions
- **Monitoring**: Uptime and error rate monitoring

### 2. Dependency Vulnerability Scan
- **Run npm audit**: Check for known vulnerabilities
- **Check outdated**: npm outdated for updates available
- **Transitive deps**: Scan indirect dependencies
- **License check**: Identify license issues
- **Package reputation**: Check for suspicious packages
- **Version pinning**: Verify locked versions
- **Security scanning**: Snyk or similar scanning
- **Generate report**: List vulnerabilities by severity

### 3. Secrets Detection
- **Git history**: Scan for hardcoded secrets (truffleHog)
- **Code scanning**: Search for API keys, tokens, passwords
- **Config files**: Check environment configs
- **Logs**: Verify secrets not in logs
- **Comments**: Look for commented credentials
- **Tests**: Test fixtures without real credentials
- **Documentation**: No examples with real keys
- **Revoke**: Revoke any exposed credentials

### 4. Security Headers Verification
- **Strict-Transport-Security**: HSTS header present
- **Content-Security-Policy**: CSP policy configured
- **X-Content-Type-Options**: nosniff header set
- **X-Frame-Options**: Clickjacking protection
- **X-XSS-Protection**: Legacy XSS protection
- **Referrer-Policy**: Control referrer leakage
- **Permissions-Policy**: Feature policy configured
- **CORS Headers**: Origin validation

### 5. Authentication Flow Analysis
- **Login Flow**: Secure authentication process
- **Password Reset**: Secure token generation
- **Session Management**: Secure session handling
- **API Authentication**: Proper endpoint protection
- **Token Management**: Token expiration and refresh
- **Multi-Factor**: MFA implementation
- **Account Lockout**: Brute force protection
- **Audit Trail**: Auth attempts logged

### 6. Authorization & Access Control
- **RBAC Implementation**: Roles properly enforced
- **Permission Checks**: Verified on all endpoints
- **Resource Access**: Ownership verified before access
- **Public APIs**: Intentional public access only
- **Token Scopes**: Scope enforcement
- **Admin Functions**: Admin role required
- **Audit Logging**: Access decisions logged

### 7. Data Protection Review
- **Encryption at Rest**: Data encrypted with AES-256
- **Encryption in Transit**: TLS 1.2+ used
- **Key Management**: Secure key storage
- **Key Rotation**: Regular rotation schedule
- **PII Handling**: Sensitive data protection
- **Data Retention**: Deletion after retention period
- **Backup Security**: Encrypted backups
- **Data Masking**: Logs don't contain PII

### 8. Infrastructure Security
- **Network Segmentation**: VPCs and security groups
- **Database Access**: Limited database access
- **SSH Keys**: SSH key management
- **Firewall Rules**: Restrictive firewall rules
- **DDoS Protection**: WAF and rate limiting
- **Intrusion Detection**: IDS/IPS monitoring
- **Patch Management**: Regular patching schedule
- **Configuration Management**: IaC security

### 9. Generate Security Report
- **Vulnerabilities**: All found vulnerabilities listed
- **Risk Assessment**: Risk severity (Critical/High/Medium/Low)
- **Remediation**: Specific fix recommendations
- **Timeline**: Suggested fix timelines
- **Compliance**: Compliance status check
- **Trend**: Security posture improvement
- **Recommendations**: Prioritized actions

### 10. Create Remediation Plan
- **Critical Fixes**: Immediate action items
- **High Priority**: Fix within 1 week
- **Medium Priority**: Fix within 1 month
- **Low Priority**: Fix when possible
- **Quick Wins**: Easy fixes first
- **Long-term**: Architectural improvements
- **Monitoring**: Ongoing security monitoring

## Quality Criteria

- All OWASP Top 10 areas assessed
- Zero critical vulnerabilities
- No known CVEs in dependencies
- No exposed secrets in code
- All security headers present
- Authentication flow secure
- Authorization properly enforced
- Data protection verified
- Infrastructure hardened
- Audit trail complete

## Output Expectations

```
SECURITY_AUDIT_REPORT.md
├── Executive Summary
├── Overall Risk Score
├── Review Date
├── Critical Vulnerabilities
│   ├── Vulnerability ID
│   ├── Description
│   ├── Location
│   ├── Risk Level
│   └── Remediation
├── High Vulnerabilities
├── Medium Vulnerabilities
├── Low Vulnerabilities
├── OWASP Top 10 Status
│   └── Each category assessed
├── Dependency Vulnerabilities
│   ├── Package Name
│   ├── Vulnerability
│   ├── Severity
│   └── Fix Available
├── Secrets Detected
├── Security Headers
├── Authentication Assessment
├── Authorization Assessment
├── Data Protection
├── Infrastructure Security
├── Compliance Status
├── Risk Trend
├── Remediation Plan
│   └── Prioritized actions
├── Monitoring Recommendations
└── Next Steps
```

## Success Indicators

- All OWASP Top 10 issues identified
- Zero critical vulnerabilities
- All dependencies scanned
- No secrets exposed
- Security headers verified
- Auth flow secure
- Access control working
- Data encrypted
- Infrastructure hardened
- Actionable recommendations
