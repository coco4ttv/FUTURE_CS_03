# API Security Assessment Notes

## Assessment details

- **Target:** JSONPlaceholder
- **Base URL:** https://jsonplaceholder.typicode.com
- **Tool:** Postman
- **Assessment type:** API security risk analysis using read-only requests and a limited simulated POST request
- **Assessment date:** 20 November 2026

## Scope and limitations

The assessment used JSONPlaceholder, a public demonstration API containing synthetic data. The results describe the behaviour of this test service and should not be treated as proof of vulnerabilities in a production application.

No destructive requests or load/stress tests were performed. JSONPlaceholder simulates write operations, so a successful write response does not demonstrate persistent storage. The available endpoints do not provide a genuine authentication and token lifecycle for assessing token security.

## Test results

### 01 – API discovery: list posts

- **Request:** GET /posts
- **HTTP response:** 200 OK
- **Observation:** The endpoint returned a JSON collection of post records, including userId, id, title and body fields.
- **Evidence:** `evidence/01-api-discovery/`

### 02 – API discovery: individual post

- **Request:** GET /posts/1
- **HTTP response:** 200 OK
- **Observation:** The endpoint returned an individual post record containing userId, id, title and body.
- **Evidence:** `evidence/01-api-discovery/`

### 03 – Unauthenticated user access

- **Request:** GET /users/4
- **Authentication:** No Auth
- **HTTP response:** 200 OK
- **Observation:** The endpoint returned a user record without credentials. The response included name, username, email, address, geographic coordinates, phone, website and company details.
- **Interpretation:** This confirms that the demonstration endpoint is publicly accessible. The records are synthetic, so this observation alone does not establish an authentication vulnerability in a production system.
- **Evidence:** `evidence/02-authentication/`

### 04 – User data exposure

- **Request:** GET /users/1
- **HTTP response:** 200 OK
- **Observation:** The response contained a detailed synthetic user profile, including name, username, email, address, geographic coordinates, phone number, website and company information.
- **Interpretation:** The endpoint returns a broad set of profile fields. Whether this would represent excessive data exposure depends on the requirements and sensitivity of a real application; the JSONPlaceholder data is synthetic.
- **Evidence:** `evidence/03-data-exposure/`

### 05 – Object access comparison: post 1

- **Request:** GET /posts/1
- **HTTP response:** 200 OK
- **Observation:** The endpoint returned the post with ID 1.
- **Evidence:** `evidence/04-access-control/`

### 06 – Object access comparison: post 2

- **Request:** GET /posts/2
- **HTTP response:** 200 OK
- **Observation:** The endpoint returned a different post record with ID 2. Both records were accessible through direct object IDs.
- **Interpretation:** This comparison demonstrates access to public demonstration records. It does not establish whether a production application correctly enforces user-specific authorisation.
- **Evidence:** `evidence/04-access-control/`

### 07 – Security response headers

- **Request:** GET /posts/1
- **HTTP response:** 200 OK
- **Headers observed in the response:** `Content-Type: application/json; charset=utf-8`, `Access-Control-Allow-Credentials: true`, `Cache-Control: max-age=43200`, `Expires: -1`, `Pragma: no-cache`, `Vary: Origin, Accept-Encoding`, `X-Content-Type-Options: nosniff`, `X-Powered-By: Express`, `X-RateLimit-Limit: 3600000`, `X-RateLimit-Remaining: 3599999`, `X-RateLimit-Reset: 1790812826`, and `Server: cloudflare`.
- **Observation:** The response included the listed headers. The presence of `X-Powered-By: Express` discloses the server framework. The rate-limit headers expose the configured limit, remaining count and reset value; the screenshot shows a very high limit.
- **Interpretation:** Header presence alone does not establish the overall security of the API. The captured response did not provide enough evidence to assess every recommended security header or the practical effectiveness of the rate limit.
- **Evidence:** `evidence/05-security-headers/`

### 08 – Valid query input

- **Request:** GET /posts?userId=1
- **HTTP response:** 200 OK
- **Observation:** The endpoint returned a collection of posts associated with userId 1.
- **Evidence:** `evidence/06-input-validation/`

### 09 – Unexpected query input

- **Request:** GET /posts?userId=cc
- **HTTP response:** 200 OK
- **Observation:** The response body was an empty JSON array (`[]`). The service did not return a client error for this non-numeric query value.
- **Interpretation:** The observed behaviour may indicate permissive handling of an unexpected query value. It does not, by itself, demonstrate an exploitable input-validation vulnerability.
- **Evidence:** `evidence/06-input-validation/`

### 10 – Query for a non-matching user ID

- **Request:** GET /posts?userId=100000
- **HTTP response:** 200 OK
- **Observation:** The response body was an empty JSON array (`[]`) because no posts matched the supplied userId.
- **Interpretation:** The API returned an empty result rather than an error for a query that matched no records.
- **Evidence:** `evidence/06-input-validation/`

### 11 – Rate-limit header review

- **Request:** GET /posts (the captured response header view)
- **HTTP response:** 200 OK
- **Headers observed:** `X-RateLimit-Limit: 3600000`, `X-RateLimit-Remaining: 3599999`, and `X-RateLimit-Reset: 1790812826`.
- **Observation:** The response exposed rate-limit metadata. The captured values indicate a configured limit of 3,600,000 and 3,599,999 remaining at the time of the response.
- **Limitation:** This was a header inspection only. No repeated-request, load or stress testing was performed, so enforcement and resistance to abuse were not verified.
- **Evidence:** `evidence/07-rate-limiting/`

### 12 – Simulated resource creation

- **Request:** POST /posts
- **HTTP response:** 201 Created
- **Observation:** The API returned a created-resource response containing an ID. One captured response returned ID 101.
- **Interpretation:** JSONPlaceholder simulates write operations. The 201 response demonstrates the endpoint's simulated creation response, not permanent storage.
- **Evidence:** `evidence/08-request-methods/`

### 13 – Persistence verification

- **Status:** Not verified in the evidence reviewed.
- **Observation:** JSONPlaceholder documents that write operations are simulated. A follow-up GET would need to be recorded separately to describe the actual observed persistence-check response.
- **Evidence:** No confirmed persistence-verification response captured in the evidence reviewed.

### 14 – HTTP method inspection

- **Status:** Not verified in the evidence reviewed.
- **Observation:** No OPTIONS response or confirmed Allow header is present in the screenshots reviewed.
- **Evidence:** No confirmed OPTIONS response captured in the evidence reviewed.

### 15 – HTTPS and CORS review

- **Status:** Partially observed.
- **HTTPS:** Requests used the HTTPS JSONPlaceholder base URL.
- **CORS header observed:** `Access-Control-Allow-Credentials: true`.
- **Limitation:** The reviewed screenshot does not show an `Access-Control-Allow-Origin` value, so the allowed origin configuration cannot be concluded from this evidence.
- **Evidence:** `evidence/05-security-headers/`

### 16 – API documentation review

- **Status:** Not recorded in the evidence reviewed.
- **Documentation:** https://jsonplaceholder.typicode.com/guide/
- **Observation:** No specific documentation findings have been recorded here.

### 17 – Authentication token limitations

- **Status:** Not verifiable with this target.
- **Observation:** The tested JSONPlaceholder endpoints do not provide a genuine authentication and token lifecycle for this assessment.
- **Not assessed:** Token strength, expiry, refresh, revocation and JWT validation.

## Findings register

The observations below are limited to the demonstration API. They are not classified as confirmed production vulnerabilities.

| ID | Observation | Assessment |
|---|---|---|
| OBS-01 | User profile endpoints return synthetic profile information without authentication. | Public demo behaviour observed; production authentication risk not established. |
| OBS-02 | User profile responses include multiple contact and profile fields. | Data fields observed; sensitivity in a real application would depend on its data requirements. |
| OBS-03 | Direct requests to two post IDs returned their respective records. | Basic object access comparison completed; authorisation controls in a real application remain unverified. |
| OBS-04 | The response disclosed `X-Powered-By: Express`. | Technology disclosure observed; no exploitability demonstrated. |
| OBS-05 | An unexpected non-numeric userId query returned HTTP 200 with an empty array. | Permissive input handling observed; no exploitability demonstrated. |
| OBS-06 | Rate-limit headers exposed a very high configured limit and remaining count. | Header metadata observed; actual enforcement not tested. |

## Overall assessment

The assessment confirmed that the tested JSONPlaceholder endpoints returned post and user records over HTTPS, including user profile fields, without requiring authentication on the tested user endpoint. Direct post-ID requests returned the requested demonstration records. Query tests returned HTTP 200 with an empty array for both a non-numeric userId and a userId with no matching posts.

The response-header review identified technology disclosure through `X-Powered-By: Express`, along with rate-limit metadata and other response headers. A simulated POST request returned HTTP 201 and an ID, but this does not demonstrate persistent storage.

The work did not verify production authentication or authorisation controls, token security, actual rate-limit enforcement, persistence of created resources, or the OPTIONS method response. These limitations are important because JSONPlaceholder is a demonstration service using synthetic data and simulated write operations. No confirmed production vulnerability is claimed on the basis of these tests.
