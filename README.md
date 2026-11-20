# API Security Risk Analysis

A read-only API security assessment of JSONPlaceholder, a public demonstration REST API, completed as part of the Future Interns Cyber Security Internship (Task 3).

## Assessment overview

The assessment reviews API discovery, unauthenticated access, response data exposure, object access behaviour, HTTP response headers, input handling and resource-consumption controls. A limited POST request is also included to observe the demonstration API's simulated resource-creation behaviour.

The work is documented using observed responses and supporting screenshots. JSONPlaceholder is a learning environment containing synthetic data; findings from this assessment should not be treated as evidence of vulnerabilities in a production service.

## Scope and limitations

- **Target:** https://jsonplaceholder.typicode.com
- **Primary tool:** Postman
- **Supporting tool:** Browser Developer Tools
- **Approach:** Documentation review and controlled requests to public demonstration endpoints
- **Excluded:** Exploitation, bypass attempts, destructive testing, load testing and testing private or production APIs

JSONPlaceholder simulates write operations. A successful response to a simulated POST request does not establish that data was persistently stored. The service does not provide a genuine user authentication and token lifecycle suitable for a full authentication assessment.

## Repository contents

| Path | Contents |
|---|---|
| `evidence/` | Screenshots grouped by assessment area |
| `notes/` | Working assessment notes and observations |
| `postman/` | Exported Postman collection and environment guidance |
| `report/` | Final API security assessment report |

## Assessment status

The repository is being populated as the assessment evidence and analysis are finalised. Test results, severity ratings and conclusions will be based on the recorded responses. Areas that cannot be verified with JSONPlaceholder will be identified as limitations rather than reported as confirmed vulnerabilities.

## References

- [JSONPlaceholder](https://jsonplaceholder.typicode.com/)
- [JSONPlaceholder Guide](https://jsonplaceholder.typicode.com/guide/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [OWASP API Security Project](https://github.com/OWASP/API-Security)
