# Request Methods

This directory contains evidence from the assessment of API request methods using JSONPlaceholder.

The assessment includes a simulated POST request to create a post, a follow-up GET request to check whether the returned resource is available, and an OPTIONS request to inspect the endpoint's supported HTTP methods.

JSONPlaceholder simulates write operations. A successful POST response therefore does not demonstrate that the submitted resource was permanently stored. The persistence check is documented as an observed limitation of the test environment.

Evidence:
- E12 – Simulated POST request
- E13 – Persistence verification
- E14 – HTTP method inspection
