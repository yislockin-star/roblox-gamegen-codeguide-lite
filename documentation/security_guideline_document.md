# Security Guidelines for roblox-gamegen-codeguide-lite

This document outlines security best practices and controls tailored to the **AI-Powered Roblox Game & Asset Generator** boilerplate. It ensures the application is robust, resilient, and compliant with industry standards.

---

## 1. Authentication & Access Control

• **Clerk Integration & Protected Routes**
  - Enforce Clerk’s `authMiddleware` on all `/api/*` and generator dashboard routes.
  - Deny unauthenticated access; redirect to login for protected pages.

• **Strong Authentication Policies**
  - Require complex passwords (minimum 12 characters, uppercase, lowercase, numbers, symbols).
  - Store user credentials exclusively via Clerk’s secure hashing (bcrypt or Argon2).
  - Implement multi-factor authentication (MFA) for elevated privileges or billing changes.

• **Session Management**
  - Use HttpOnly, Secure cookies with `SameSite=Strict` for session tokens.
  - Enforce idle and absolute session timeouts (e.g., 30 minutes idle, 24 hours max).
  - Provide explicit logout endpoints that revoke session tokens.

• **Role-Based Access Control (RBAC)**
  - Define roles (e.g., `developer`, `admin`) in Clerk metadata.
  - Perform server-side role checks before sensitive operations (e.g., billing, credits adjustments).

---

## 2. Input Handling & Processing

• **Prompt & Payload Validation**
  - Treat all developer prompts as untrusted; sanitize control characters.
  - Enforce length limits and block suspicious keywords (e.g., OS commands).

• **SQL & Injection Protection**
  - Always use Supabase’s parameterized queries or client libraries—no string concatenation.
  - Enable PostgreSQL Row-Level Security (RLS) policies to isolate each user’s data.

• **Output Encoding**
  - Encode any user-supplied text before rendering in the UI to prevent XSS.
  - Use a trusted syntax-highlighter for displaying Lua scripts rather than `dangerouslySetInnerHTML`.

---

## 3. Data Protection & Privacy

• **Encryption in Transit & At Rest**
  - Enforce TLS 1.2+ (HTTPS) for all network traffic (Vercel handles TLS by default).
  - Verify Supabase database uses at-rest encryption.

• **Secrets Management**
  - Store LLM API keys, Clerk, and Supabase credentials in Vercel environment variables.
  - Avoid committing any secrets or `.env.local` files to source control.
  - Consider a dedicated secrets manager (e.g., AWS Secrets Manager) for high-sensitivity keys.

• **Data Minimization & Retention**
  - Persist only necessary PII (e.g., user ID, email) and generated assets.
  - Implement a data-deletion policy: allow users to fully delete their account and associated data.

• **Logging & Information Leakage**
  - Sanitize logs—never include full prompts or API keys.
  - Mask or truncate personally identifiable data before logging.
  - In production, disable stack-trace output in error responses.

---

## 4. API & Service Security

• **Endpoint Protection**
  - All `/api/chat`, `/api/projects`, `/api/assets` routes require Clerk authentication.
  - Validate HTTP methods: POST for generation and creation, GET for retrieval, DELETE for removal.

• **Rate Limiting & Throttling**
  - Implement rate limiting per IP and per user for `/api/chat` to guard against abuse and DoS.
  - Leverage Vercel’s Edge Functions or an API gateway for token-based quotas.

• **CORS Configuration**
  - Restrict CORS to trusted origins (your frontend domain).
  - Avoid wildcard (`*`) origins on production.

• **Minimal Response Data**
  - Return only required fields (e.g., `scriptId`, `codeSnippet`), never full user profiles or secrets.

---

## 5. Web Application Security Hygiene

• **Security Headers**
  - Content-Security-Policy (CSP): allow scripts/styles only from self and vetted CDNs.
  - Strict-Transport-Security (HSTS): `max-age=31536000; includeSubDomains; preload`.
  - X-Content-Type-Options: `nosniff`.
  - X-Frame-Options: `DENY`.
  - Referrer-Policy: `no-referrer-when-downgrade`.

• **CSRF Protection**
  - Use anti-CSRF tokens (e.g., Next.js built-in CSRF protection for form submissions).

• **Secure Cookies**
  - Set `HttpOnly`, `Secure`, `SameSite=Strict` on session and CSRF cookies.

• **Subresource Integrity (SRI)**
  - When loading third-party scripts/styles (e.g., shadcn/ui CDN), include integrity hashes.

---

## 6. Infrastructure & Configuration Management

• **Environment Hardening**
  - Disable debug/log-level=verbose in production builds.
  - Remove unused environment variables and credentials.

• **Service & Port Exposure**
  - On Vercel, rely on platform-managed firewalls; do not expose admin panels to the public.

• **Dependency Updates & Patching**
  - Enable Dependabot or Snyk to scan for known vulnerabilities in NPM dependencies.
  - Review and merge dependency updates regularly; rebuild lockfiles (`pnpm lockfile` or similar).

---

## 7. Dependency Management

• **Trusted Libraries Only**
  - Vet all major dependencies: Next.js, Clerk, Supabase, Tailwind, shadcn/ui, Vercel AI SDK.
  - Limit direct dependencies; remove unused packages to shrink attack surface.

• **Lockfiles & Deterministic Builds**
  - Commit `pnpm-lock.yaml` (or `package-lock.json`) to ensure reproducible installations.

• **Vulnerability Scanning**
  - Integrate static analysis (e.g., npm audit, GitHub Code Scanning) in your CI pipeline.

---

## 8. AI Generation & Prompt Security

• **Prompt Engineering & Filtering**
  - Prepend system prompts that restrict LLM from executing malicious code or network calls.
  - Sanitize user prompts to prevent injection of unauthorized instructions.

• **Output Validation & Linting**
  - Run a server-side Lua linter or basic syntax checker on generated scripts before saving.
  - Reject or flag outputs containing dangerous APIs (e.g., `os.execute`, file writes).

• **Usage Monitoring & Quotas**
  - Track tokens or generation counts per user; enforce credit-based throttles.
  - Alert on anomalous usage spikes that may indicate abuse.

---

## 9. Monitoring, Error Handling & Incident Response

• **Centralized Logging & Alerts**
  - Forward application logs to a SIEM (e.g., Datadog, Logflare).
  - Alert on repeated failures in `/api/chat` or database access errors.

• **Fail Securely**
  - On upstream LLM or Supabase failure, surface a generic error message without revealing internals.
  - Ensure partial failures do not leak or corrupt user data.

• **Incident Preparedness**
  - Document recovery procedures for compromised API keys or database credentials.
  - Rotate credentials immediately upon suspected leakage.

---

## Conclusion
By adhering to these guidelines—rooted in Security by Design, Least Privilege, and Defense in Depth—you will build a secure, scalable, and trustworthy AI-powered Roblox generator platform. Regularly review and update controls as dependencies, threat models, and regulations evolve.
