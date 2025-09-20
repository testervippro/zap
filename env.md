

# ✅ ZAP Automation Framework: `env:` Section (Complete Guide)

The `env:` section defines the **environment configuration** that the rest of the automation framework will operate on. This includes:

* What URLs ZAP should scan
* What authentication to use (if any)
* Which technologies to include/exclude
* How to manage sessions
* Which users can log in
* What variables to use across the YAML
* Whether to fail on errors
* Optional proxy settings

It is the **foundation** of any ZAP Automation Framework scan.

---

## 🔷 Structure of the `env:` Section

```yaml
env:
  contexts:
    - name:           # Required – context name
      urls:           # Required – list of root URLs
      includePaths:   # Optional – regex patterns to include
      excludePaths:   # Optional – regex patterns to exclude
      authentication: # Optional – authentication method & parameters
      sessionManagement:  # Optional – how ZAP manages the session
      technology:     # Optional – target tech to include/exclude
      structure:      # Optional – structural and data-driven path config
      users:          # Optional – list of users for authentication
  vars:               # Optional – global variables used in the YAML
  parameters:         # Optional – controls fail behavior and logging
  proxy:              # Optional – upstream proxy configuration
```

---

## 🔹 `contexts:` (Required)

Defines **one or more "contexts"**, each representing a logical web application scope. Other jobs (like spider and scanner) act on these contexts.

### ➤ Example:

```yaml
contexts:
  - name: "MainApp"
    urls:
      - "http://example.com"
```

### 🔸 `urls:` (Required)

* List of base URLs for this context.
* Avoid using `:80` for HTTP or `:443` for HTTPS — ZAP treats those as default and may break matching if included.

### 🔸 `includePaths:` / `excludePaths:` (Optional)

* Regex patterns for **what to include or exclude** from scanning.
* Helps narrow the scan scope.

```yaml
includePaths:
  - "http://example.com/dashboard.*"

excludePaths:
  - "http://example.com/logout.*"
```

---

## 🔹 `authentication:` (Optional, but Required for Authenticated Scans)

Tells ZAP how to log in to the app.

### ✅ Supported Methods:

| Method    | Use Case                    |
| --------- | --------------------------- |
| `manual`  | Use existing session/cookie |
| `form`    | Login with POST form        |
| `http`    | Basic/Digest HTTP Auth      |
| `json`    | API login with JSON body    |
| `script`  | Custom script-based login   |
| `browser` | Headless browser login      |
| `client`  | Client certificate auth     |

### 🔸 `parameters:` (Varies by method)

#### 🔹 For `form` login:

```yaml
authentication:
  method: form
  parameters:
    loginPageUrl: "http://example.com/login"
    loginRequestUrl: "http://example.com/login"
    loginRequestBody: "username={%username%}&password={%password%}"
  verification:
    method: response
    loggedInRegex: "Logout"
    loggedOutRegex: "Login"
```

* **`{%username%}` and `{%password%}`** get replaced by the user credentials in the `users:` section.
* `verification.method` can be:

  * `response` → look at response content
  * `request` → check a specific follow-up request
  * `both` or `poll` → advanced checks (like polling a URL for status)

---

## 🔹 `sessionManagement:` (Optional)

Defines how ZAP **tracks session state**.

### ➤ Common Option:

```yaml
sessionManagement:
  method: cookie
```

### ➤ Scripted session management:

```yaml
sessionManagement:
  method: script
  parameters:
    script: /zap/wrk/scripts/sessionHandler.js
    scriptEngine: ECMAScript
```

---

## 🔹 `technology:` (Optional)

Used to tell ZAP what technologies are present in the target app.

```yaml
technology:
  exclude:
    - Apache
    - ColdFusion
```

> 🔗 Full list of technologies: [https://www.zaproxy.org/techtags/](https://www.zaproxy.org/techtags/)

---

## 🔹 `structure:` (Optional)

Useful for APIs or applications with data-driven URLs.

```yaml
structure:
  structuralParameters:
    - sessionid
  dataDrivenNodes:
    - name: ProductIDs
      regex: "/product/(\\d+)"
```

* Helps ZAP recognize **dynamic paths** (e.g., `/product/123`)
* Regex must have 2 or 3 capturing groups.

---

## 🔹 `users:` (Optional)

Defines one or more users ZAP can log in as (used with `authentication`).

```yaml
users:
  - name: "adminUser"
    credentials:
      username: "admin"
      password: "adminpass"
```

### ➤ Advanced: TOTP support (for MFA apps)

```yaml
totp:
  secret: "BASE32ENCODEDSECRET"
  period: 30
  digits: 6
  algorithm: SHA1
```

---

## 🔹 `vars:` (Optional)

Define **reusable variables** anywhere in the YAML.

```yaml
vars:
  baseUrl: "http://example.com"
  loginUser: "admin"
  loginPass: "admin123"
```

Then use like this:

```yaml
loginRequestUrl: "${baseUrl}/login"
username: "${loginUser}"
password: "${loginPass}"
```

---

## 🔹 `parameters:` (Optional)

Controls framework-level behavior.

```yaml
parameters:
  failOnError: true         # Stop pipeline if an error occurs
  failOnWarning: false      # Allow warnings
  continueOnFailure: false  # Stop if a job fails
  progressToStdout: true    # Print progress in real time
```

---

## 🔹 `proxy:` (Optional)

If ZAP should connect through an upstream proxy (e.g., for internet access or security scanning).

```yaml
proxy:
  hostname: "proxy.internal"
  port: 8080
  realm: "corp"
  username: "proxyuser"
  password: "proxypass"
```

---

# 🧪 Full Example: Authenticated Scan with Context

```yaml
env:
  contexts:
    - name: "MainApp"
      urls:
        - "http://example.com"
      includePaths:
        - "http://example.com/secure/.*"
      excludePaths:
        - "http://example.com/logout.*"
      authentication:
        method: form
        parameters:
          loginPageUrl: "${baseUrl}/login"
          loginRequestUrl: "${baseUrl}/login"
          loginRequestBody: "username={%username%}&password={%password%}"
        verification:
          method: response
          loggedInRegex: "Logout"
          loggedOutRegex: "Login"
      sessionManagement:
        method: cookie
      technology:
        exclude:
          - ColdFusion
          - ASP
      structure:
        dataDrivenNodes:
          - name: "Product Pages"
            regex: "/product/(\\d+)"
      users:
        - name: "adminUser"
          credentials:
            username: "${loginUser}"
            password: "${loginPass}"
  vars:
    baseUrl: "http://example.com"
    loginUser: "admin"
    loginPass: "admin123"
  parameters:
    failOnError: true
    failOnWarning: false
    continueOnFailure: false
    progressToStdout: true
  proxy:
    hostname: "proxy.corp.local"
    port: 8080
    realm: "corp"
    username: "proxyuser"
    password: "proxypass"
```

---

## ✅ Recap: Key Things to Remember

| Section              | Purpose                                                         |
| -------------------- | --------------------------------------------------------------- |
| `contexts:`          | Define the web app and how to scan it                           |
| `authentication:`    | Configure how ZAP logs in                                       |
| `sessionManagement:` | How ZAP maintains sessions (cookies, script, etc.)              |
| `users:`             | Define credentials for authenticated users                      |
| `vars:`              | Reuse variables for cleaner configuration                       |
| `parameters:`        | Control what causes ZAP to fail or continue                     |
| `proxy:`             | Use if you're behind a corporate proxy or want to route traffic |
| `technology:`        | Improve scan quality by excluding irrelevant tech               |


