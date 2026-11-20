# API Security Assessment Notes

## Assessment details

- **Target:** JSONPlaceholder
- **Base URL:** https://jsonplaceholder.typicode.com
- **Tool:** Postman
- **Supporting tool:** Browser Developer Tools
- **Assessment type:** Read-only API security risk analysis, with a limited simulated POST request
- **Assessment date:** 11 november 2026

## Recording guidance

Record the actual request, response status, relevant fields and interpretation for each test. Keep observations separate from conclusions. Do not describe an item as a vulnerability unless the evidence supports that conclusion.

Use these statuses: Completed, Not tested, Not verifiable, or Not applicable.

## Test record

### 01 – API discovery: list posts
- Request: GET /posts
- Status: Completed – verify screenshot and record exact response
- HTTP response: [Record]
- Observation: [Record]
- Evidence: evidence/01-api-discovery/

### 02 – API discovery: individual post
- Request: GET /posts/1
- Status: Completed – verify screenshot and record exact response
- HTTP response: [Record]
- Observation: [Record]
- Evidence: evidence/01-api-discovery/

### 03 – Unauthenticated user access
- Request: GET /users/4
- Authentication: No Auth
- Observed response: HTTP 200
- Observation: The endpoint returned a user record without credentials. The response included profile fields such as name, username, email, address, phone, website and company details.
- Interpretation: This confirms public access to a demonstration record. The data is synthetic and this alone does not establish an authentication vulnerability.
- Evidence: evidence/02-authentication/

### 04 – User data exposure
- Request: GET /users/1
- Status: [Record]
- Fields observed: [Record]
- Observation: [Record]
- Interpretation: [Record]
- Evidence: evidence/03-data-exposure/

### 05 – Object access comparison
- Request: GET /posts/1
- Status: [Record]
- Observation: [Record]
- Evidence: evidence/04-access-control/

### 06 – Object access comparison
- Request: GET /posts/2
- Status: [Record]
- Observation: [Record]
- Limitation: Public demonstration records do not establish whether a real application enforces per-user authorisation.
- Evidence: evidence/04-access-control/

### 07 – Security response headers
- Request: [Record endpoint]
- Status: [Record]
- Headers reviewed: [Record]
- Observation: [Record]
- Evidence: evidence/05-security-headers/

### 08 – Valid input
- Request: GET /posts?userId=1
- Status: [Record]
- Observation: [Record]
- Evidence: evidence/06-input-validation/

### 09 – Unexpected input
- Request: GET /posts?userId=abc
- Status: [Record]
- Observation: [Record]
- Evidence: evidence/06-input-validation/

### 10 – Non-existent resource
- Request: GET /posts/999999
- Status: [Record]
- Observation: [Record]
- Evidence: evidence/06-input-validation/

### 11 – Rate limiting review
- Request: [Record endpoint]
- Status: [Record]
- Rate-limit headers: [Record]
- Observation: [Record]
- Limitation: No load or stress testing was performed.
- Evidence: evidence/07-rate-limiting/

### 12 – Simulated resource creation
- Request: POST /posts
- Status: [Record]
- Request body: [Summarise]
- Response: [Record]
- Observation: [Record]
- Evidence: evidence/08-request-methods/

### 13 – Persistence verification
- Request: GET /posts/101 (or the ID returned by the POST response)
- Status: [Record]
- Observation: JSONPlaceholder simulates write operations; a successful POST response does not mean the new record was persistently stored. Record the actual verification response.
- Evidence: evidence/08-request-methods/

### 14 – HTTP method inspection
- Request: OPTIONS /posts
- Status: [Record]
- Allow header: [Record if present]
- Observation: [Record]
- Evidence: evidence/08-request-methods/

### 15 – HTTPS and CORS review
- Request: [Record endpoint]
- HTTPS: [Record]
- Access-Control-Allow-Origin: [Record]
- Access-Control-Allow-Credentials: [Record]
- Observation: [Record]
- Evidence: evidence/05-security-headers/

### 16 – API documentation review
- Documentation: https://jsonplaceholder.typicode.com/guide/
- Resources and methods reviewed: [Record]
- Documented limitations: [Record]

### 17 – Authentication token limitations
- Status: Not verifiable with this target.
- Observation: JSONPlaceholder does not provide a genuine authentication and token lifecycle for these demo endpoints.
- Not assessed: Token strength, expiry, refresh, revocation and JWT validation.

## Findings register

Do not add a confirmed finding until the supporting evidence has been reviewed.

| ID | Observation | OWASP category | Severity | Evidence |
|---|---|---|---|---|
| F-01 | [To be determined] | [To be determined] | [To be determined] | [Reference] |

## Overall assessment

[Complete after reviewing all evidence. Summarise what was tested, what was observed, what could not be verified and the main recommendations.]
