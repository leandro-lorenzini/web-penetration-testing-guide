# Web Application Penetration Testing Guide

This document provides a reference guide for manual penetration testing on a web application. It is not an exhaustive pentesting checklist but rather a starting point to help guide your testing efforts. Each test outlined here includes sample commands and techniques you can use to evaluate common vulnerabilities. However, the specific tests you conduct should be tailored to the architecture and functionality of the application under review.

Use these examples as a framework rather than a step-by-step checklist; they are intended to save time and provide practical approaches, so you don’t have to start from scratch. As always, adapt your tests based on the unique aspects of the application, and remember that this list only scratches the surface of a comprehensive security assessment.


### Authentication Testing
- **Test**: Brute-force login attempts.
- **Sample**: Use a tool like `hydra` to attempt multiple username-password combinations.
  ```bash
  hydra -L users.txt -P passwords.txt -s 443 -f api.example.com https-post-form "/api/login:username=^USER^&password=^PASS^:Invalid username or password"
  ```
- **Goal**: Verify if the app locks out accounts after multiple failed attempts or implements CAPTCHA.

### Authorization Testing (Privilege Escalation)
- **Test**: Access restricted resources as a lower-privilege user.
- **Sample**: Use a tool like `curl` to change the `user_id` in API calls and check responses.
  ```bash
  curl -X GET "https://api.example.com/user/12345/orders" -H "Authorization: Bearer <low-privilege-token>"
  ```
- **Goal**: Check if accessing other users’ data is restricted by proper access control.

### Session Management
- **Test**: Session token reuse and logout behavior.
- **Sample**: Log in, copy the session token, log out, and attempt to reuse the token.
  ```bash
  curl -X GET "https://api.example.com/user/profile" -H "Authorization: Bearer <old-session-token>"
  ```
- **Goal**: Confirm that old tokens are invalidated after logout and that session tokens have expiration.

### SQL Injection
- **Test**: SQL injection on user-input fields.
- **Sample**: Manually input common SQL payloads into search or login fields.
  ```plaintext
  ' OR '1'='1
  ```
- **Goal**: See if the application returns unexpected data, errors, or changes in behavior that indicate vulnerability.

### Cross-Site Scripting (XSS)
- **Test**: Inject JavaScript payloads into fields to check for reflected or stored XSS.
- **Sample**: Enter the following payload in form fields or URL parameters.
  ```html
  <script>alert("XSS")</script>
  ```
- **Goal**: Observe if the payload is executed, indicating XSS vulnerability.

### Insecure Direct Object References (IDOR)
- **Test**: Modify object identifiers in API requests to access unauthorized data.
- **Sample**: Change resource identifiers in an API request to test access.
  ```bash
  curl -X GET "https://api.example.com/user/100/orders" -H "Authorization: Bearer <token>"
  ```
- **Goal**: Check if the app enforces access control on user-specific data.

### Rate Limiting
- **Test**: Send rapid requests to critical endpoints (e.g., login, signup) and measure response times.
- **Sample**: Use `seq` with `curl` to quickly submit repeated requests.
  ```bash
  for i in `seq 1 1000`; do curl -X POST "https://api.example.com/login" -d "username=user&password=pass"; done
  ```
- **Goal**: Determine if the application has rate limits or throttling on sensitive endpoints.

### File Upload
- **Test**: Attempt to upload executable files or large files if there is an upload feature.
- **Sample**: Upload a PHP shell disguised as an image (e.g., `.jpg`) to test file-type validation.
  ```plaintext
  image.php.jpg with content: <?php system($_GET['cmd']); ?>
  ```
- **Goal**: Confirm file validation measures and prevent arbitrary file uploads.

### API Error Handling
- **Test**: Send malformed JSON or intentionally erroneous API requests.
- **Sample**: Send requests with invalid JSON or missing required fields.
  ```bash
  curl -X POST "https://api.example.com/endpoint" -d '{"incomplete_json":}'
  ```
- **Goal**: Check for error messages that reveal stack traces or sensitive information.

### Cross-Origin Resource Sharing (CORS) Misconfiguration
- **Test**: Check if the API allows access from unauthorized domains.
- **Sample**: Use a custom JavaScript script in a separate domain to test if it can access the API.
  ```javascript
  fetch('https://api.example.com/data', {
    method: 'GET',
    credentials: 'include'
  }).then(response => response.text()).then(data => console.log(data));
  ```
- **Goal**: Ensure sensitive endpoints are not accessible from arbitrary origins.
