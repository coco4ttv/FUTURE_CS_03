# API Security Risk Analysis

A practical API security assessment of JSONPlaceholder, a public demonstration REST API, completed as part of the Future Interns Cyber Security Internship (Task 3).

## Assessment overview

The assessment covered API discovery, unauthenticated access, response data exposure, object access behaviour, HTTP response headers, input handling and resource-consumption controls. A limited POST request and follow-up persistence check were also performed.

The assessment was carried out using controlled requests in Postman. The results are documented with supporting screenshots and a written report. JSONPlaceholder uses synthetic data and simulates write operations, so observations from this exercise should not be treated as confirmed vulnerabilities in a production service.

## Report

- **[API Security Risk Analysis Report (PDF)](report/API_Security_Risk_Analysis_Report.pdf)** – report prepared for submission.
- [Markdown version of the report](report/API-Security-Assessment-Report.md)

## Scope and methodology

- **Target:** https://jsonplaceholder.typicode.com
- **Primary tool:** Postman
- **Supporting tool:** Browser Developer Tools
- **Approach:** Review of the API documentation followed by controlled requests to public demonstration endpoints.
- **Excluded:** Exploitation, bypass attempts, destructive testing, load testing and testing of private or production APIs.

The service does not provide a genuine user authentication and token lifecycle suitable for a full authentication assessment. Its write operations are simulated; a successful POST response does not establish that submitted data was permanently stored. These limitations are reflected in the report.

## Repository contents

| Path | Contents |
|---|---|
| `evidence/` | Postman screenshots grouped by assessment area |
| `notes/` | Supporting assessment observations |
| `postman/` | Exported Postman collection |
| `report/` | PDF and Markdown versions of the assessment report |

## Tools and references

- [Postman](https://www.postman.com/)
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/)
- [JSONPlaceholder Guide](https://jsonplaceholder.typicode.com/guide/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
