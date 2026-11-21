# API Security Risk Analysis Report

## 1. Executive Summary

This assessment reviewed the security behaviour of JSONPlaceholder, a public demonstration REST API, using Postman. The work covered API discovery, unauthenticated access, user profile data, object access, response headers, query input, rate-limit metadata, simulated resource creation and HTTP method inspection.

The tests produced several useful observations:

- The tested user endpoint returned a detailed synthetic user profile without authentication.
- Requests for two different post IDs returned the corresponding public demonstration records. This did not establish broken access control.
- The response exposed the technology header `X-Powered-By: Express`.
- A non-numeric `userId` query returned HTTP 200 with an empty JSON array rather than a client error.
- Rate-limit headers were present and advertised a very high limit, but enforcement was not tested.
- A simulated POST request returned HTTP 201 and ID 101. A subsequent GET request for that ID returned HTTP 404 with an empty JSON object.
- An OPTIONS request returned HTTP 204 and included an `Access-Control-Allow-Methods` header advertising several methods.

JSONPlaceholder uses synthetic data and simulates write operations. It does not provide a real user authentication and token lifecycle for this assessment. Therefore, the observations describe this demonstration service and must not be represented as confirmed vulnerabilities in a production system.

The main outcome is a documented set of API security observations, limitations and recommendations that could be applied when assessing a real business API.

## 2. Introduction and Business Context

An Application Programming Interface (API) allows different software systems to communicate. For example, a shopping website may use an API to retrieve product details, create customer orders and check delivery information. A mobile banking application may use APIs to retrieve account balances or submit payments.

APIs are important to businesses because they connect customer-facing applications, internal systems and third-party services. If an API is poorly secured, attackers may be able to access information, misuse business functions, disrupt services or manipulate records.

This assessment used JSONPlaceholder as a controlled learning target to practise identifying API behaviour and interpreting security-relevant responses. The results are considered in a business context, with recommendations informed by the OWASP API Security Top 10 (2023) and OWASP security guidance.

## 3. Scope and Rules of Engagement

### 3.1 Target

- **Service:** JSONPlaceholder
- **Base URL:** https://jsonplaceholder.typicode.com
- **Assessment tool:** Postman
- **Assessment period represented in the captured evidence:** 20–21 November 2026

### 3.2 Areas reviewed

- API endpoint discovery
- Public access to user endpoints
- User profile response fields
- Access to individual post records
- HTTP response headers and CORS-related headers
- Query parameter handling
- Rate-limit response headers
- Simulated POST behaviour and follow-up retrieval
- OPTIONS method response

### 3.3 Testing boundaries

The work used controlled requests to public demonstration endpoints. No destructive PUT, PATCH or DELETE requests were sent. No high-volume or stress testing was performed. No attempt was made to bypass authentication or access controls.

The service does not provide a genuine authentication and token lifecycle for this assessment. Token expiry, refresh, revocation and JWT validation were therefore outside the practical scope.

### 3.4 Important limitations

JSONPlaceholder is a demonstration API containing synthetic data. Its write operations are simulated rather than persistently stored. This affects the interpretation of the POST and persistence tests. Results cannot be assumed to represent the security controls of a commercial API.

## 4. Tools and Methodology

Postman was used to send HTTP requests and inspect status codes, response bodies and response headers. The assessment followed a manual testing approach:

1. Identify available API resources through collection and individual resource requests.
2. Observe whether selected endpoints returned data without authentication.
3. Compare responses from different object identifiers.
4. Inspect returned profile fields and HTTP response headers.
5. Submit valid, unexpected and non-matching query values.
6. Review rate-limit metadata without generating excessive traffic.
7. Send a limited POST request and attempt to retrieve the returned resource.
8. Send an OPTIONS request and inspect the response headers.

The assessment recorded observed behaviour rather than assuming that a response code alone proves a vulnerability.

## 5. Assessment Results

### 5.1 API Discovery

**Requests**
- `GET /posts`
- `GET /posts/1`

**Observed result**

Both requests returned HTTP 200 OK. The collection request returned a JSON list of posts. The individual request returned a post containing fields such as `userId`, `id`, `title` and `body`.

**What this means**

The endpoints provide access to a collection of post records and to an individual record selected by its ID.

**Business relevance**

A business API commonly exposes similar endpoints for products, orders, bookings or customer records. Knowing which endpoints exist is an important first step in understanding what information and functions an API makes available.

**Recommendation**

Maintain an accurate API inventory and document each endpoint, its purpose, accepted parameters, data returned and required permissions. Restrict access to endpoints that are not intended for public use.

**Evidence**

[E01–E02](evidence/01-api-discovery) in `evidence/01-api-discovery/`.

### 5.2 Authentication – Public User Endpoint

**Request**

`GET /users/4` with Postman authentication set to **No Auth**.

**Observed result**

The request returned HTTP 200 OK and a user profile. The response included a name, username, email address, postal address, geographic coordinates, phone number, website and company information.

**What this means**

The tested endpoint returned the profile without the request supplying authentication credentials. This is the expected public behaviour of this JSONPlaceholder demonstration endpoint. It does not establish that a private business API has broken authentication.

**Business relevance**

If a business API containing private customer or employee information allowed unauthenticated access, people outside the organisation could potentially retrieve information they should not see. Depending on the data involved, this could lead to privacy complaints, regulatory consequences, financial costs and loss of customer trust.

**Recommended remediation for a real API**

- Require authentication for endpoints that expose private or restricted information.
- Validate credentials or access tokens on the server for every protected request.
- Apply appropriate token expiry, revocation and secure storage controls.
- Separate authentication (confirming who a user is) from authorisation (checking what that user is allowed to do).
- Review public endpoints to ensure that public access is intentional and that only appropriate data is returned.

**OWASP mapping**

This test relates to the risk area described by **API2:2023 – Broken Authentication**, but no authentication vulnerability was confirmed in JSONPlaceholder because the endpoint is intentionally public.

**Evidence**

E03 in `evidence/02-authentication/`.

### 5.3 User Data Exposure

**Request**

`GET /users/1`

**Observed result**

The request returned HTTP 200 OK. The profile included a name, username, email, address, geographic coordinates, phone number, website and company information.

**What this means**

The endpoint returns a broad set of profile fields in one response. The information is synthetic demonstration data. Whether the same fields would be excessive in a real application depends on the business purpose, the user's permissions and the data that the application needs to disclose.

**Business relevance**

A business that returns unnecessary personal information increases the amount of data exposed if an account, API key or endpoint is compromised. Exposure of contact details and location information can create privacy and safety concerns, increase incident response costs and damage the organisation's reputation.

**Recommended remediation for a real API**

- Return only the fields required for the task being performed.
- Apply field-level authorisation where different users should see different information.
- Avoid returning sensitive profile fields in general-purpose responses.
- Review API responses against data protection and privacy requirements.
- Test responses using accounts with different roles and permissions.

**OWASP mapping**

This area is related to **API3:2023 – Broken Object Property Level Authorization**. The category covers failures to apply appropriate authorisation to individual object properties. The JSONPlaceholder response alone does not prove that such a failure exists in a production context.

**Evidence**

E04 in `evidence/03-data-exposure/`.

### 5.4 Access Control – Comparison of Post IDs

**Requests**

- `GET /posts/1`
- `GET /posts/2`

**Observed result**

Both requests returned HTTP 200 OK. Each response contained the corresponding public post record.

**What this means**

The comparison confirms that the demonstration API returns records when their IDs are requested. It does not prove that one authenticated user can access another user's private information. The test did not use separate user accounts or records protected by ownership rules.

**Business relevance**

In a real service, broken object-level authorisation could allow one customer to view another customer's orders, invoices or personal records by changing an ID in a request. This could cause serious confidentiality and privacy issues.

**Recommended remediation for a real API**

- Check the requesting user's permission to access the specific record on every request.
- Enforce ownership and role checks on the server, not only in the user interface.
- Test with separate accounts that have different record ownership and permissions.
- Deny access when the user does not have permission to the requested object.
- Include object-level authorisation tests in the development and release process.

**OWASP mapping**

This test relates to **API1:2023 – Broken Object Level Authorization (BOLA)**. BOLA was not confirmed because JSONPlaceholder does not provide the separate authenticated accounts and private record ownership needed to demonstrate it.

**Evidence**

E05–E06 in `evidence/04-access-control/`.

### 5.5 HTTP Response Headers and Security Configuration

**Request**

`GET /posts/1`

**Observed result**

The captured response included the following headers:

| Header | Observed value |
|---|---|
| `Content-Type` | `application/json; charset=utf-8` |
| `Access-Control-Allow-Credentials` | `true` |
| `Cache-Control` | `max-age=43200` |
| `Expires` | `-1` |
| `Pragma` | `no-cache` |
| `Vary` | `Origin, Accept-Encoding` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Powered-By` | `Express` |
| `X-RateLimit-Limit` | `3600000` |
| `X-RateLimit-Remaining` | `3599999` |
| `X-RateLimit-Reset` | `1790812826` |
| `Server` | `cloudflare` |

**What this means**

The `X-Powered-By` header reveals that Express is used by the application stack. This is technology disclosure: it gives a remote observer information about part of the server technology. It does not, by itself, show that the server is vulnerable.

The response also includes CORS-related and caching headers. The captured evidence does not show enough information to determine the complete CORS origin policy or to assess every recommended security header.

**Business relevance**

Unnecessary technology details can help an attacker understand the technologies used by a service and select more targeted research. Incorrect CORS or caching configuration can also expose information to unintended websites or users if sensitive responses are involved.

**Recommended remediation for a real API**

- Remove unnecessary technology-identification headers where practical.
- Review security headers against the application's architecture and threat model.
- Configure CORS with an explicit allowlist of trusted origins.
- Allow credentials only for trusted origins that genuinely require them.
- Apply appropriate cache controls to sensitive or user-specific responses.
- Review headers periodically and test the deployed configuration.

**OWASP mapping**

These observations relate to **API8:2023 – Security Misconfiguration**. The evidence identifies configuration details worth reviewing; it does not prove that a security misconfiguration is exploitable.

**Evidence**

E07 in `evidence/05-security-headers/`.

### 5.6 Input Validation

**Requests and results**

| Request | HTTP response | Response body / result |
|---|---|---|
| `GET /posts?userId=1` | 200 OK | Matching post records returned |
| `GET /posts?userId=cc` | 200 OK | Empty JSON array: `[]` |
| `GET /posts?userId=100000` | 200 OK | Empty JSON array: `[]` |

**What this means**

The valid numeric value returned matching posts. The non-numeric value `cc` returned an empty array rather than a client error. The numeric value `100000` also returned an empty array because no posts matched that user ID.

An empty array is a reasonable response when a valid query has no matching records. The non-numeric input is the more relevant validation observation. However, the result alone does not demonstrate injection or another exploitable vulnerability.

**Business relevance**

If a business application does not validate input properly, it may process unexpected values, produce incorrect results or pass unsafe data to other components. Depending on how the application uses that input, poor validation can contribute to security issues, data quality problems and service disruption.

**Recommended remediation for a real API**

- Validate that `userId` is an integer before processing it.
- Check values against documented ranges and business rules.
- Return a clear client error, such as HTTP 400 Bad Request, for invalid input where that matches the API contract.
- Use parameterised database queries rather than constructing database commands from raw input.
- Log repeated invalid requests and investigate suspicious patterns.
- Test boundary values, unexpected types, missing parameters and malformed input.

**OWASP mapping**

The OWASP REST Security Cheat Sheet recommends validating input type, format, length and range, and rejecting unexpected content. This is supporting OWASP guidance rather than a standalone category in the API Security Top 10 (2023).

**Evidence**

E08–E10 in `evidence/06-input-validation/`.

### 5.7 Rate-Limit Header Review

**Request**

`GET /posts`

**Observed result**

The captured response included:

- `X-RateLimit-Limit: 3600000`
- `X-RateLimit-Remaining: 3599999`
- `X-RateLimit-Reset: 1790812826`

**What this means**

The server exposes rate-limit information. The captured limit is very high. The assessment did not send repeated high-volume requests, so it did not establish whether the service blocks excessive traffic or whether the limit is effectively enforced.

**Business relevance**

Without effective request limits, an attacker or malfunctioning integration could send large numbers of requests. This could slow down a service, make it unavailable to customers or increase infrastructure and bandwidth costs.

**Recommended remediation for a real API**

- Apply appropriate request limits based on the endpoint and business risk.
- Consider limits per authenticated account, API key and source IP where appropriate.
- Return HTTP 429 Too Many Requests when a client exceeds its limit.
- Use stricter limits for expensive operations and sensitive workflows.
- Monitor traffic patterns and alert on unusual request volumes.
- Verify enforcement in a controlled test environment.

**OWASP mapping**

This area relates to **API4:2023 – Unrestricted Resource Consumption**. The assessment observed rate-limit metadata but did not verify whether enforcement was effective.

**Evidence**

E11 in `evidence/07-rate-limiting/`.

### 5.8 Simulated POST and Persistence Verification

**Requests**

- `POST /posts`
- `GET /posts/101`

**Observed result**

The POST request returned HTTP 201 Created and a response containing ID 101. The follow-up GET request to `/posts/101` returned HTTP 404 Not Found with an empty JSON object (`{}`).

**What this means**

The POST response looked like a successful creation response, but the resource could not subsequently be retrieved using the returned ID. This is consistent with JSONPlaceholder's documented behaviour: write operations are simulated and changes are not permanently stored.

**Business relevance**

In a real business application, a similar mismatch could be serious. A customer might believe an order, booking or support request had been saved when the system had not actually stored it. This could result in lost sales, missed bookings, duplicated work, disputes and reduced customer confidence.

**Recommended remediation for a real API**

- Confirm that a database write has been committed before returning a successful creation response.
- Use database transactions for operations that must be completed reliably.
- Return accurate status codes and resource identifiers.
- Monitor failed writes and investigate database or service errors.
- Use reconciliation processes for important transactions so that missing records can be identified.

**Assessment conclusion**

This is an observed limitation of the JSONPlaceholder test environment, not a confirmed production vulnerability. The POST response and subsequent 404 are both recorded as evidence of the simulated-write behaviour.

**Evidence**

E12–E13 in `evidence/08-request-methods/`.

### 5.9 HTTP Method Inspection

**Request**

`OPTIONS /posts`

**Observed result**

The server returned HTTP 204 No Content. The response included an `Access-Control-Allow-Methods` header. The screenshot visibly shows GET, HEAD, PUT, PATCH, POST and DELETE among the listed methods; the full header value is truncated in the captured view. The response also included `Access-Control-Allow-Credentials: true`.

**What this means**

The OPTIONS response advertises methods in the server's CORS response. It does not prove that every listed method is available for every resource, that the methods would succeed, or that they are correctly protected by authorisation controls.

No PUT, PATCH or DELETE requests were sent.

**Business relevance**

If a real API allows users to modify or delete business records without checking their permissions, information could be changed or removed without approval. This could affect orders, customer records, financial data or business operations.

**Recommended remediation for a real API**

- Allow only the HTTP methods required by each endpoint.
- Enforce authentication and authorisation for every sensitive operation.
- Check that the requesting user has permission to perform the action on the specific record.
- Return HTTP 405 Method Not Allowed for methods that are not supported.
- Configure CORS origins, methods and credentials narrowly rather than allowing unnecessary cross-origin access.

**OWASP mapping**

The method restriction and authorisation recommendations are covered by the OWASP REST Security Cheat Sheet. The OPTIONS response alone does not establish an API Security Top 10 vulnerability.

**Evidence**

E14 in `evidence/08-request-methods/`.

## 6. Findings Register and Assessment Status

The following register separates observed behaviour from vulnerabilities that could not be verified.

| ID | Area | Assessment outcome |
|---|---|---|
| OBS-01 | Public user endpoint | Unauthenticated access to synthetic profile data observed; production authentication weakness not established |
| OBS-02 | User profile fields | Multiple profile fields returned; excessive exposure in a real business context not established |
| OBS-03 | Object access | Two public post IDs were accessible; BOLA not verified |
| OBS-04 | Technology disclosure | `X-Powered-By: Express` observed; exploitability not demonstrated |
| OBS-05 | Input handling | Non-numeric query returned HTTP 200 and an empty array; exploitable validation weakness not demonstrated |
| OBS-06 | Rate limiting | Rate-limit headers observed; enforcement not tested |
| OBS-07 | Persistence | POST returned 201; subsequent GET returned 404, consistent with simulated writes |
| OBS-08 | HTTP methods | OPTIONS returned 204 and advertised methods; method-level authorisation not tested |

No confirmed production vulnerability is claimed by this assessment.

## 7. Business Risk and Prioritised Recommendations

Although the demonstration service did not allow the assessment to confirm production vulnerabilities, a business operating a similar API should prioritise the following controls.

### Priority 1 – Protect private data and records

- Require authentication for private endpoints.
- Enforce object-level and property-level authorisation on the server.
- Return only the information required by the user's task.
- Test access using separate accounts and different permission levels.

**Business benefit:** Reduces the likelihood of customer data exposure, unauthorised record access and privacy incidents.

### Priority 2 – Protect service availability

- Apply effective rate limits.
- Monitor unusual request patterns.
- Use HTTP 429 responses when limits are exceeded.
- Set limits appropriate to the cost and sensitivity of each operation.

**Business benefit:** Helps maintain service availability and control infrastructure costs.

### Priority 3 – Validate requests and protect data integrity

- Validate input type, format and range.
- Use parameterised database queries.
- Confirm successful data writes before acknowledging completion.
- Monitor and reconcile failed transactions.

**Business benefit:** Reduces incorrect data, failed transactions and avoidable customer disputes.

### Priority 4 – Review API configuration

- Remove unnecessary technology disclosure.
- Restrict HTTP methods to those required.
- Configure CORS using trusted origins and necessary methods only.
- Review response headers and caching rules.

**Business benefit:** Reduces unnecessary information disclosure and limits opportunities for configuration-related weaknesses.

## 8. OWASP Reference Mapping

The OWASP API Security Top 10 (2023) provides a recognised framework for discussing API risks. The mapping below identifies relevant categories for the assessment areas; it does not mean that each category was confirmed as a vulnerability.

| OWASP category | Relevance to this assessment |
|---|---|
| API1:2023 – Broken Object Level Authorization | Relevant to the post-ID comparison; not verified |
| API2:2023 – Broken Authentication | Relevant to unauthenticated user access; not verified as a vulnerability because the endpoint is intentionally public |
| API3:2023 – Broken Object Property Level Authorization | Relevant to the profile fields returned; not verified |
| API4:2023 – Unrestricted Resource Consumption | Relevant to rate-limit review; enforcement not tested |
| API8:2023 – Security Misconfiguration | Relevant to response-header and CORS configuration review; exploitability not demonstrated |

Input validation and HTTP method restrictions are additionally discussed using OWASP REST Security Cheat Sheet guidance. The simulated POST persistence behaviour is treated as a test-environment limitation rather than being forced into an unrelated OWASP category.

## 9. Limitations of the Assessment

The following limitations affected the depth of testing:

- JSONPlaceholder is a public demonstration service using synthetic data.
- The tested user endpoint does not provide a genuine authentication and token lifecycle.
- Separate authenticated accounts and private record ownership were not available to demonstrate BOLA.
- Rate-limit enforcement was not tested through high-volume requests.
- JSONPlaceholder simulates write operations, so the POST test did not demonstrate persistent storage.
- PUT, PATCH and DELETE requests were not sent.
- The OPTIONS response showed an advertised methods header, but did not establish the authorisation or implementation of each method.
- The captured response headers did not provide enough evidence to assess the complete CORS policy or every security header.

These limitations mean that the results should be interpreted as a practical learning assessment of a demonstration API, not a penetration test or security certification of a production service.

## 10. Conclusion

The assessment provided practical experience in discovering API endpoints, sending HTTP requests, interpreting response codes, reviewing JSON data and inspecting response headers using Postman.

The tests demonstrated that JSONPlaceholder returns public post and user records, accepts the tested query requests, exposes rate-limit and technology information in response headers, and simulates resource creation. The follow-up persistence request returned 404, while the OPTIONS request returned 204 and advertised several HTTP methods.

The most important outcome is the distinction between an observation and a confirmed vulnerability. The assessment did not establish broken authentication, broken object-level authorisation or ineffective rate limiting in a production environment. It did identify security controls that a business should verify when assessing its own APIs.

For a real organisation, the recommended priorities are to protect private data with authentication and authorisation, validate all input, enforce rate limits, restrict HTTP methods and ensure that successful write responses reflect data that has actually been stored.

## 11. References

1. OWASP Foundation. **OWASP API Security Top 10 – 2023.** https://owasp.org/www-project-api-security/
2. OWASP. **API Security Top 10 – 2023 Documentation.** https://api-security.owasp.org/editions/2023/en/0x00-header/
3. OWASP Cheat Sheet Series. **REST Security Cheat Sheet.** https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
4. JSONPlaceholder. **JSONPlaceholder Guide.** https://jsonplaceholder.typicode.com/guide/
5. JSONPlaceholder. **JSONPlaceholder API.** https://jsonplaceholder.typicode.com/

## Appendix A – Evidence Index

| Evidence | Assessment area | Repository location |
|---|---|---|
| E01 | Posts collection | `evidence/01-api-discovery/E01-posts-list.png` |
| E02 | Individual post | `evidence/01-api-discovery/E02-single-post.png` |
| E03 | Unauthenticated user request | `evidence/02-authentication/E03-user4-unauthenticated-request.png` |
| E04 | User profile response | `evidence/03-data-exposure/E04-user-data-response.png` |
| E05 | Post ID 1 response | `evidence/04-access-control/E05-post-1.png` |
| E06 | Post ID 2 response | `evidence/04-access-control/E06-post-2.png` |
| E07 | Security response headers | `evidence/05-security-headers/E07-response-headers.png` |
| E08 | Valid userId query | `evidence/06-input-validation/E08-valid-query.png` |
| E09 | Non-numeric userId query | `evidence/06-input-validation/E09-invalid-query.png` |
| E10 | Non-matching userId query | `evidence/06-input-validation/E10-nonexistent-resource.png` |
| E11 | Rate-limit response headers | `evidence/07-rate-limiting/E11-rate-limit-headers.png` |
| E12 | Simulated POST response | `evidence/08-request-methods/E12-create-post.png` |
| E13 | Persistence check (404 response) | `evidence/08-request-methods/404-for-the uploaded-101.png` |
| E14 | OPTIONS method inspection | `evidence/08-request-methods/HTTP-Methods-Inspection.png` |
