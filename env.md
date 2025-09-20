env:                                   # The environment, mandatory
  contexts :                           # List of 1 or more contexts, mandatory
    - name: context 1                  # Name to refer to this context, mandatory
      urls:                            # Mandatory list of top-level URLs
        - https://example.com
        - https://example.org/api
      includePaths:                    # Optional regex list to include specific paths
        - "^/api/.*"
      excludePaths:                    # Optional regex list to exclude paths
        - "^/admin/.*"
      authentication:
        method: form                   # One of 'manual', 'http', 'form', 'json', 'script', 'autodetect', 'browser', or 'client'
        parameters:                    # Depends on method
          hostname: auth.example.com   # For 'http' authentication
          port: 8080                   # Int, only for 'http' authentication
          realm: "Example Realm"       # Realm for HTTP Auth
          loginPageUrl: https://example.com/login    # Only for 'form' or 'json' authentication
          loginRequestUrl: https://example.com/api/login # Only for 'form' or 'json' authentication
          loginRequestBody: "username=${username}&password=${password}"  # POST body for login
          script: "./scripts/auth.js"  # Path to custom script, only for 'script' auth
          scriptInline: |              # Inline script (JS, etc.) — use instead of 'script'
            function authenticate(context) {
              // Custom auth logic
            }
          scriptEngine: "ECMAScript"   # Script engine name, like ECMAScript, Groovy, etc.
        verification:
          method: response             # One of 'response', 'request', 'both', 'poll'
          loggedInRegex: "Welcome.*"   # Regex to detect logged-in state
          loggedOutRegex: "Login Page" # Regex to detect logged-out state
          pollFrequency: 5             # Only for 'poll' method
          pollUnits: seconds           # 'requests' or 'seconds'
          pollUrl: https://example.com/status # Polling URL
          pollPostData: "statusCheck=true"    # Optional POST body for polling
          pollAdditionalHeaders:       # Headers for poll request (optional)
            - header: Authorization
              value: Bearer ${authToken}
      sessionManagement:
        method: cookie                 # One of 'cookie', 'http', 'script'
        parameters:
          script: "./scripts/session.js"     # Path to session management script (optional)
          scriptEngine: "ECMAScript"         # Name of script engine
      technology:
        exclude:                       # Tech to exclude, per https://www.zaproxy.org/techtags/
          - WordPress
          - ASP
        include:                       # Optional: Tech to explicitly include (exclude takes precedence)
          - Apache
          - Tomcat
      structure:
        structuralParameters:          # Optional: Parameters considered structural
          - sessionId
          - page
        dataDrivenNodes:              # Optional: Data-driven nodes to detect repeating structures
          - name: userIdParam
            regex: "/users/(\\d+)/orders/(\\d+)"   # Must have 2 or 3 capture groups
      users:                           # List of users for auth testing
        - name: testUser1
          credentials:
            username: user1@example.com   # Username
            password: ${user1Password}    # Password (can reference vars)
            totp:                         # Optional TOTP for MFA
              secret: JBSWY3DPEHPK3PXP
              period: 30
              digits: 6
              algorithm: SHA1
  vars:                                # Custom variables usable throughout config
    user1Password: Pa$$w0rd123
    authToken: abc123xyzToken
    myVarOne: CustomConfigVarOne
    myVarTwo: ${myVarOne}.VarTwo
  parameters:
    failOnError: true                  # If true, exit on any error
    failOnWarning: false               # If true, exit on any warning
    continueOnFailure: false          # If true, keeps running even if jobs fail
    progressToStdout: true            # Logs job progress to stdout
  proxy:                               # Optional: use upstream proxy
    hostname: proxy.example.com        # Proxy host
    port: 3128                         # Proxy port
    realm: proxyRealm                  # Proxy realm if using HTTP auth
    username: proxyUser                # Proxy username
    password: proxyPass                # Proxy password
