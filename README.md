# VAPT-Project
# Web Application VAPT Assessment — giftacareer.in

## Project Overview

This project documents a limited-scope Vulnerability Assessment and Penetration Testing (VAPT) exercise conducted against `giftacareer.in` using a Kali Linux testing environment.

The assessment focused on reconnaissance, network service discovery, HTTP security configuration, automated vulnerability scanning, manual response verification, lightweight performance observations, and evidence documentation.

The objective was to identify potential security weaknesses, record observable application behaviour, and prepare findings that can support further verification and remediation.

## Objectives

* Perform DNS reconnaissance and identify resolved IP addresses.
* Discover accessible network services and review their responses.
* Identify potential HTTP security-header misconfigurations.
* Run automated security scans using OWASP ZAP and Nikto.
* Perform limited manual verification of HTTP responses.
* Observe basic response-time behaviour under light request concurrency.
* Document evidence, limitations, and recommended remediation areas.

## Tools and Technologies

| Tool             | Purpose                                     |
| ---------------- | ------------------------------------------- |
| Kali Linux       | Security testing environment                |
| Nmap 7.94SVN     | Port scanning and service detection         |
| WhatWeb          | Web server and technology reconnaissance    |
| Nikto 2.6.1      | Web server configuration checks             |
| OWASP ZAP 2.17.0 | Automated web security scanning             |
| ApacheBench 2.3  | Lightweight response-time testing           |
| DNS utilities    | Domain resolution and IP discovery          |
| Chromium         | Browser compatibility and manual inspection |

## Assessment Scope

**Target:** `giftacareer.in`

The assessment included DNS resolution, permitted service discovery, HTTP/HTTPS response inspection, security-header observations, automated scanning, limited manual verification, and lightweight performance testing.

During testing, the domain was observed redirecting to `ccgac.bitrix24.site`, associated with the Bitrix24 Sites platform. A contact form hosted at `share.hsforms.com` was also identified and remained outside the tested scope.

Testing was limited to the authorised target and permitted activities. Brute-force attacks, credential stuffing, destructive testing, persistence, and denial-of-service or stress testing were excluded.

## Methodology

1. **Reconnaissance:** Resolve the target domain and collect initial server information.
2. **Port and Service Discovery:** Use Nmap to identify accessible TCP ports and examine service responses.
3. **Web Technology Detection:** Use WhatWeb to inspect the HTTP endpoint.
4. **Vulnerability Scanning:** Use Nikto and OWASP ZAP to identify potential configuration weaknesses.
5. **Manual Verification:** Record HTTP status codes and relevant response headers.
6. **Performance Observation:** Run a small ApacheBench test with 50 requests and a concurrency level of 5.
7. **Browser Testing:** Inspect the page in Chromium.
8. **Evidence Documentation:** Preserve tool outputs and observations for reporting.

## Key Findings and Observations

### 1. DNS Reconnaissance

The DNS lookup returned two IPv4 addresses for `giftacareer.in`:

* `15.197.225.128`
* `3.33.251.168`

The Nmap scan examined `15.197.225.128`; the second address was not scanned in that run.

### 2. Port and Service Discovery

The Nmap scan reported:

| Port                | State    | Service     |
| ------------------- | -------- | ----------- |
| 80/TCP              | Open     | HTTP        |
| 443/TCP             | Open     | HTTPS       |
| 998 other TCP ports | Filtered | No response |

The detected server banner was `awselb/2.0`. The reverse DNS result also contained an AWS Global Accelerator hostname, indicating AWS-related infrastructure.

The HTTPS certificate identified `giftacareer.in` and was reported valid through 9 March 2027.

### 3. Web Server Reconnaissance

WhatWeb received a `403 Forbidden` response from the HTTP endpoint and identified the `awselb/2.0` server banner.

The response limited the amount of application technology information available during that scan.

### 4. Nikto Scan Observations

Nikto reported the following potentially missing HTTP security headers:

* Content-Security-Policy (CSP)
* Permissions-Policy
* Strict-Transport-Security (HSTS)
* X-Content-Type-Options
* Referrer-Policy

The scan also observed internal AWS hostnames in server-related responses and headers such as `x-request-id` and `wafrule`.

These observations require contextual verification. Header behaviour may differ between HTTP and HTTPS responses, and the presence of a header in one response does not establish that it is configured consistently across the application.

### 5. OWASP ZAP Results

The automated ZAP report contained **9 alerts**.

| Risk level    | Alert count |
| ------------- | ----------: |
| High          |           0 |
| Medium        |           3 |
| Low           |           4 |
| Informational |           2 |
| **Total**     |       **9** |

The alerts included:

* Content Security Policy Header Not Set
* Missing Anti-clickjacking Header
* Sub Resource Integrity Attribute Missing
* Server Leaks Version Information via the Server HTTP Response Header
* Strict-Transport-Security Header Not Set
* Timestamp Disclosure — Unix
* X-Content-Type-Options Header Missing
* Modern Web Application
* Re-examine Cache-control Directives

These are automated scanner results, not nine confirmed vulnerabilities. The report contained no alerts marked as user-confirmed, so manual validation and impact assessment remain important.

### 6. Manual HTTP Response Verification

The manual verification evidence contained four HTTP/2 responses with status `405`.

The responses included:

* `Content-Type: text/plain; charset=utf-8`
* `X-Content-Type-Options: nosniff`
* `Content-Length: 19`

A `405 Method Not Allowed` response indicates that the request method was not permitted for the requested resource. Because the captured evidence does not identify the request method and endpoint, it is insufficient by itself to establish a vulnerability or fully validate the security control.

### 7. Lightweight Performance Test

ApacheBench was used to issue 50 requests with a concurrency level of 5.

| Metric                      | Observed result |
| --------------------------- | --------------: |
| Total requests              |              50 |
| Failed requests             |               0 |
| Non-2xx responses           |              50 |
| Requests per second         |           11.08 |
| Average total response time |          406 ms |
| Median total response time  |          372 ms |
| Longest response time       |          701 ms |
| Test duration               |   4.512 seconds |

Although ApacheBench reported zero failed requests, every response was outside the HTTP 2xx range. Therefore, these results describe response behaviour during a small test; they do not establish successful application functionality, maximum capacity, or production-scale performance.

An internal AWS hostname was also observed in the server-related output.

### 8. Browser Compatibility

The audit log recorded that the page rendered correctly in Chromium and that zero browser-console errors were observed during the check.

This is a limited browser observation and does not establish comprehensive cross-browser compatibility.

## Limitations

* HTTP requests frequently returned `403 Forbidden`.
* Some HTTPS probes returned `400`, `405`, or `422` responses.
* Restricted responses limited the visibility of application functionality.
* The Nmap results cover one of the two resolved IP addresses.
* Automated alerts require manual validation before being treated as confirmed vulnerabilities.
* The performance test was intentionally small and was not a stress or load-capacity assessment.
* The third-party hosting platform and externally hosted contact form were not independently assessed.

## Recommended Remediation Areas

Based on the observations, the following areas merit review by the authorised application or infrastructure owner:

1. Review and configure appropriate security headers, including CSP, HSTS, anti-clickjacking protections, Referrer-Policy, Permissions-Policy, and X-Content-Type-Options.
2. Review server and infrastructure responses for unnecessary disclosure of internal hostnames or implementation details.
3. Validate the applicability and impact of each ZAP alert before prioritising remediation.
4. Review cache-control directives and third-party resource integrity where applicable.
5. Reassess the application through authorised testing that provides sufficient access to validate relevant functionality.
6. Repeat performance testing only when an appropriate test environment, expected response behaviour, and safe testing limits are defined.

## Repository Evidence

The collected evidence includes:

* `VAPT_Evidence/audit_log.txt`
* `VAPT_Evidence/manual_verification.txt`
* `VAPT_Evidence/nikto_output.txt`
* `VAPT_Evidence/perf_test.txt`
* `VAPT_Evidence/recon_dns.txt`
* `VAPT_Evidence/recon_nmap.txt`
* `VAPT_Evidence/recon_whatweb.txt`

The OWASP ZAP scan report was also reviewed as part of the assessment evidence.

## Conclusion

This assessment documented the target's observable network services, HTTP response behaviour, potential security-header configuration issues, automated scanner alerts, and results from a small performance test.

The primary outcome is an evidence-based record of potential weaknesses and areas requiring further verification. The available evidence does not establish that the target is free of vulnerabilities, nor does it confirm that every automated alert represents an exploitable issue.

All observations should be interpreted within the assessment scope and the access limitations encountered during testing.

