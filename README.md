# Web VAPT – OWASP Juice Shop

A hands-on Web Application and API Vulnerability Assessment and Penetration Testing (VAPT) project performed against a locally hosted OWASP Juice Shop instance.

The project documents reconnaissance, API enumeration, authenticated testing, evidence collection, vulnerability analysis, remediation recommendations, and professional security reporting.

> **Disclaimer:** This project was performed against a locally controlled OWASP Juice Shop test environment for authorized security testing and educational purposes.

---

## 🎯 Project Objectives

The objectives of this assessment were to:

- Perform web application reconnaissance
- Identify exposed application and API endpoints
- Analyze HTTP responses and security headers
- Perform authenticated API enumeration
- Test object-level authorization controls
- Identify potential BOLA/IDOR vulnerabilities
- Collect reproducible security evidence
- Document findings using a professional VAPT format
- Provide remediation recommendations
- Maintain a clean and security-conscious GitHub portfolio

---

## 🧪 Target Application

| Item | Details |
|---|---|
| Application | OWASP Juice Shop |
| Target | `127.0.0.1:3000` |
| Environment | Local lab |
| Testing Type | Web Application / API VAPT |
| Authentication | JWT / Cookie-based authentication |
| Primary Finding | BOLA / IDOR |
| Testing Platform | Kali Linux |

---

## 🔍 Assessment Methodology

The assessment followed a structured penetration-testing workflow:

```text
Reconnaissance
      ↓
Service Identification
      ↓
Web Enumeration
      ↓
API Enumeration
      ↓
Authentication Analysis
      ↓
Authorization Testing
      ↓
Vulnerability Validation
      ↓
Evidence Collection
      ↓
Risk Documentation
      ↓
Remediation
      ↓
Professional Reporting
