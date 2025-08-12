---
author: muju
title: "Kubernetes Ingress Traefik Middlewares: Complete Reference Guide"
description: "Comprehensive guide to Traefik middlewares in Kubernetes with ready-to-use examples for routing, security, traffic control, and observability."
categories: ["Devops"]
tags: ["Kubernetes", "DevOps", "Traefik", "Ingress", "Middleware", "Security", "Microservices", "Cloud Native"]
image:
  path: /assets/dev/kube/kubernetes.webp
  lqip: data:image/webp;base64,UklGRtQBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSIsAAAABgFNt2/LmycSJBGYwwCACNSQT6uBkZsaik54K6OnI48/4/hIiYgKgGZzOQiBG5owgMMuonsSGFVTZbVItfuAEndwpAcD8JRC/LUBKIGeAFq0DHGln4IP2CbdgoLdqRL1vxPDBiKciQ2NK6NB6gOlIOZsAON/0vbugjP3p+U9Avcxr8VVoN141m1ACAFZQOCAiAQAA8AYAnQEqFAAUAD6RQJpKJaOiIagIALASCWwAnTKEdXem8IZsH33hQHPt6YBvDZKh/GuLZkgF/OkWZ+bNTygMAAD6MfA3/NCO7/ftr7xz71nWzdAYviZtYg+gfdrtg/Tp4hzzy1+pO2kCX7u4nkR6Fk3ZkqICWtz+MTCn9H1ZhMdsmn1btqfkXfNRiyYkHFGKBFBmKYzn8f/9iLH91lHvYiPWd+mLRGD1UhH/jRNx/9jB9qalU9hqU8cN4tj/hMl0TGLIrjwy3tJfq1IYi7OpURv7z9/Qi5lO+41B2kD5EDUE/+zEwCoUer+P3T1kjyZ0JR/1JGnx/zHb+qJb/2Lx9B8P+9UlMJF1/nyvJDtSIRA+78jkMzmAjIZWiCxI2KAAAAA=j
published: true
hidden: true
toc: true
---

Traefik Middleware is a powerful feature for controlling request and response flows in your Kubernetes cluster. This comprehensive reference contains ready-to-use middleware examples categorized by function, from basic routing and security to advanced traffic control and observability. Each example includes detailed explanations and implementation notes to help you quickly integrate these patterns into your infrastructure.

---

## Table of Contents

- [Redirects](#redirects)
- [Authentication & Authorization](#authentication--authorization)
- [Rate Limiting & Traffic Control](#rate-limiting--traffic-control)
- [Headers & Security](#headers--security)
- [Request Modification](#request-modification)
- [Response Modification](#response-modification)
- [Error Handling](#error-handling)
- [Observability & Logging](#observability--logging)
- [Other Useful Middleware](#other-useful-middleware)
- [Middleware Chaining](#middleware-chaining)
- [Implementation Notes](#implementation-notes)

---

## Redirects

### 1. HTTP to HTTPS Redirect

Forces all HTTP traffic to be redirected to HTTPS for enhanced security.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-to-https
spec:
  redirectScheme:
    scheme: https     # Target scheme (https)
    permanent: true   # Returns 301 status code (permanent redirect)
```

**Usage Notes:**
- Set `permanent: false` for a temporary redirect (HTTP 302) instead of permanent (HTTP 301)
- This is commonly applied globally to all ingress routes
- Browsers will cache permanent redirects, improving subsequent request performance

### 2. Redirect to a Different Host

Redirects requests from one domain to another while preserving the path.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-to-host
spec:
  redirectRegex:
    regex: "^http(s?)://old-domain.com/(.*)"
    replacement: "https://new-domain.com/$2"
    permanent: true
```

**Usage Notes:**
- The regex pattern captures the path portion using a capture group `(.*)`
- The replacement uses `$2` to reference the second capture group (the path)
- The first capture group `(s?)` makes the 's' in https optional
- Useful for domain migrations while maintaining URL structure

### 3. Redirect to HTTPS with Custom Port

Redirects traffic to a non-standard HTTPS port.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-to-https-custom-port
spec:
  redirectScheme:
    scheme: https
    port: "8443"    # Custom HTTPS port
    permanent: true
```

**Usage Notes:**
- Useful for environments where HTTPS runs on non-standard ports
- Common in development or testing environments

### 4. Path-Based Redirect

Redirects requests from one path to another on the same host.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-path
spec:
  redirectRegex:
    regex: "^/legacy-path/(.*)"
    replacement: "/new-path/$1"
    permanent: true
```

**Usage Notes:**
- Useful for restructuring your API or website paths
- Preserves path components after the matched pattern
- Can be combined with host redirects for complete URL transformation

### 5. Redirect with Query Parameters

Redirects while preserving query parameters.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-with-query
spec:
  redirectRegex:
    regex: "^/old-search\?(.*)"
    replacement: "/new-search?$1"
    permanent: true
```

**Usage Notes:**
- The regex captures everything after the question mark
- Particularly useful for search pages or API endpoints with query parameters

## Authentication & Authorization

### 1. Basic Authentication

Implements HTTP Basic Authentication to protect routes with username/password credentials.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: basic-auth
spec:
  basicAuth:
    secret: basic-auth-secret  # Reference to a Kubernetes secret
    removeHeader: false        # Optional: keeps the Authorization header for backend services
    realm: "My Secure API"    # Optional: realm name shown in browser auth prompt
```

**Creating the Secret:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
type: Opaque
data:
  users: <base64-encoded-htpasswd-content>
```

**Usage Notes:**
- The secret must contain a key named `users` with htpasswd formatted content
- Generate credentials with: `htpasswd -nb username password | base64`
- Multiple users can be added, one per line in the htpasswd file
- Set `removeHeader: true` if you don't want to pass credentials to backend services
- Basic auth is simple but transmits credentials with minimal encryption (Base64)

### 2. Digest Authentication

Provides more secure authentication than Basic Auth by not transmitting passwords in plaintext.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: digest-auth
spec:
  digestAuth:
    secret: digest-auth-secret  # Reference to a Kubernetes secret
    removeHeader: false         # Optional: keeps auth headers for backend services
    realm: "My Secure API"     # Required: realm for digest authentication
```

**Creating the Secret:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: digest-auth-secret
type: Opaque
data:
  users: <base64-encoded-htdigest-content>
```

**Usage Notes:**
- The secret must contain a key named `users` with htdigest formatted content
- Generate with: `htdigest -c digestfile "realm-name" username`
- More secure than Basic Auth as it uses a challenge-response mechanism
- The realm name in the middleware must match the realm used when generating credentials

### 3. Forward Authentication (External Auth Server)

Delegates authentication to an external service, enabling complex auth flows like OAuth, OIDC, or custom auth systems.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: forward-auth
spec:
  forwardAuth:
    address: "https://auth.example.com/verify"  # External auth service endpoint
    trustForwardHeader: true                    # Trust X-Forwarded-* headers
    authResponseHeaders:                        # Headers to copy from auth response
      - "X-Auth-User"
      - "X-Auth-Email"
      - "X-Auth-Roles"
    authRequestHeaders:                         # Headers to forward to auth service
      - "Authorization"
      - "Cookie"
    tls:                                        # Optional TLS configuration
      caSecret: auth-ca-cert
      insecureSkipVerify: false
```

**Usage Notes:**
- The external auth service must return HTTP 2xx for successful auth, anything else is considered a failure
- `authResponseHeaders` defines which headers from the auth service get forwarded to backend services
- `authRequestHeaders` specifies which headers from the original request are sent to the auth service
- Enables integration with identity providers like Auth0, Okta, Keycloak, etc.
- Can implement complex authorization logic beyond simple username/password

### 4. OAuth2 Authentication

Implements OAuth2 authentication flow using an external provider (requires Traefik Enterprise or ForwardAuth).

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: oauth2-auth
spec:
  forwardAuth:
    address: "https://oauth-proxy.example.com/oauth2/auth"
    trustForwardHeader: true
    authResponseHeaders:
      - "X-Auth-Request-Access-Token"
      - "X-Auth-Request-User"
      - "X-Auth-Request-Email"
      - "Authorization"
```

**Usage Notes:**
- Requires an OAuth2 proxy service like oauth2-proxy, Dex, or a custom implementation
- Enables authentication with providers like Google, GitHub, Microsoft, etc.
- The proxy handles the OAuth flow and token validation
- More complex to set up but provides better security and user experience than Basic Auth

### 5. IP Whitelist

Restricts access to specific IP addresses or CIDR ranges.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: ip-whitelist
spec:
  ipWhiteList:
    sourceRange:
      - "127.0.0.1/32"          # Localhost
      - "10.0.0.0/8"            # Private network range
      - "192.168.1.0/24"        # Local subnet
    ipStrategy:
      depth: 1                  # Optional: for handling proxies
      excludedIPs:              # Optional: IPs to ignore when using depth
        - "127.0.0.1"
```

**Usage Notes:**
- Provides network-level access control
- The `depth` parameter helps with X-Forwarded-For header handling in proxy environments
- Can be combined with other auth methods for defense in depth
- Useful for admin interfaces or internal APIs

## Rate Limiting & Traffic Control

### 1. Basic Rate Limiting

Limits the number of requests a client can make in a given timeframe to prevent abuse and ensure fair resource usage.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: basic-rate-limit
spec:
  rateLimit:
    average: 100      # Average requests per second
    burst: 50         # Maximum initial burst size
```

**Usage Notes:**
- `average`: The maximum rate, measured in requests per second
- `burst`: The maximum number of requests allowed to exceed the rate in a single burst
- Default period is 1 second if not specified
- Useful for protecting APIs from abuse and DoS attacks
- Returns HTTP 429 (Too Many Requests) when limit is exceeded

### 2. Rate Limiting with Period

Provides more control over the rate limiting window by specifying a custom time period.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-with-period
spec:
  rateLimit:
    average: 100      # Average requests per period
    burst: 50         # Maximum initial burst size
    period: 1m        # Time period (1 minute in this example)
    sourceCriterion:  # Optional: define how to identify clients
      ipStrategy:
        depth: 1      # For X-Forwarded-For header handling
```

**Usage Notes:**
- `period`: Time window for rate limiting (valid units: s, m, h)
- `sourceCriterion`: Defines how to identify unique clients (by IP, header, etc.)
- For a 1-minute period with average 100, clients can make 100 requests per minute
- Longer periods are useful for APIs with expected bursts of activity

### 3. Advanced Rate Limiting

Combines rate limiting with client identification strategies for more precise control.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: advanced-rate-limit
spec:
  rateLimit:
    average: 10
    burst: 5
    period: 30s
    sourceCriterion:
      requestHeaderName: "Authorization"  # Rate limit based on auth token
      requestHost: false                  # Don't use hostname for identification
      ipStrategy:
        depth: 2                          # X-Forwarded-For depth
        excludedIPs:                      # IPs to exclude from rate limiting
          - "127.0.0.1"
          - "10.0.0.1"
```

**Usage Notes:**
- `requestHeaderName`: Uses a specific header value to identify clients
- `requestHost`: When true, uses the Host header for client identification
- Useful for rate limiting authenticated users differently than anonymous users
- Can exempt internal services or monitoring tools from rate limits

### 4. Circuit Breaker

Protects backend services by temporarily stopping traffic when error rates exceed thresholds.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.5 || ResponseCodeRatio(500, 600, 0, 600) > 0.5"
    checkPeriod: 10s        # Optional: how often to check the expression
    fallbackDuration: 30s   # Optional: how long to keep circuit open after triggering
```

**Usage Notes:**
- `expression`: Condition that triggers the circuit breaker
- Common metrics in expressions:
  - `NetworkErrorRatio()`: Ratio of network errors
  - `ResponseCodeRatio(min1, max1, min2, max2)`: Ratio of responses with status codes in range [min1:max1] vs [min2:max2]
  - `LatencyAtQuantileMS(quantile)`: Response time in milliseconds at given quantile
  - `ResponseCodeCount(code, period)`: Count of responses with given status code
- When triggered, returns 503 Service Unavailable to clients
- Prevents cascading failures and allows backend services time to recover

### 5. Retry Mechanism

Automatically retries failed requests to improve resilience against transient errors.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: retry-middleware
spec:
  retry:
    attempts: 3                  # Number of retry attempts
    initialInterval: 100ms       # Time between retries (increases exponentially)
    retryOnStatusCodes:          # Status codes that trigger retries
      - 500
      - 502
      - 503
      - 504
```

**Usage Notes:**
- Only retries idempotent methods (GET, HEAD, PUT, DELETE, OPTIONS, TRACE) by default
- `attempts`: Maximum number of attempts (including the initial request)
- `initialInterval`: Starting delay between retries (uses exponential backoff)
- `retryOnStatusCodes`: HTTP status codes that should trigger a retry
- Improves reliability without requiring client-side retry logic
- Be cautious with high attempt values to avoid overwhelming struggling backends

## Headers & Security

### 1. Security Headers

Adds important security-related HTTP headers to protect against common web vulnerabilities.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: security-headers
spec:
  headers:
    # Content Security Policy
    contentSecurityPolicy: "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.example.com; img-src 'self' data: https://*.example.com"
    
    # Cross-site protections
    frameDeny: true                     # X-Frame-Options: DENY
    customFrameOptionsValue: "SAMEORIGIN" # Alternative to frameDeny
    browserXssFilter: true              # X-XSS-Protection: 1; mode=block
    contentTypeNosniff: true            # X-Content-Type-Options: nosniff
    
    # HTTPS enforcement
    forceSTSHeader: true                # Always set STS header
    stsIncludeSubdomains: true          # Include all subdomains in HSTS
    stsPreload: true                    # Allow preloading in browsers
    stsSeconds: 31536000                # 1 year in seconds
    
    # Referrer Policy
    referrerPolicy: "strict-origin-when-cross-origin"
    
    # Permissions Policy (formerly Feature-Policy)
    permissionsPolicy: "camera=(), microphone=(), geolocation=(self), payment=()"
```

**Usage Notes:**
- `contentSecurityPolicy`: Restricts which resources can be loaded
- `frameDeny`/`customFrameOptionsValue`: Prevents clickjacking attacks
- `browserXssFilter`: Enables browser's built-in XSS protection
- `contentTypeNosniff`: Prevents MIME type sniffing
- `stsSeconds`: Duration for HSTS (HTTP Strict Transport Security)
- `referrerPolicy`: Controls information in the Referer header
- `permissionsPolicy`: Restricts browser feature usage

### 2. Custom Response Headers

Adds or modifies HTTP response headers for various purposes.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-response-headers
spec:
  headers:
    customResponseHeaders:
      X-Application-Version: "v1.2.3"     # Add custom version header
      Server: "CustomServer"              # Override server header
      Cache-Control: "public, max-age=86400" # Set caching policy
      X-Powered-By: ""                    # Remove X-Powered-By by setting empty value
```

**Usage Notes:**
- Use empty string values to remove headers
- Useful for adding application-specific headers
- Can override default headers sent by Traefik or backend services
- Helps with debugging, versioning, and cache control

### 3. Custom Request Headers

Modifies HTTP request headers before they reach backend services.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-request-headers
spec:
  headers:
    customRequestHeaders:
      X-Forwarded-Prefix: "/api"          # Add path prefix information
      X-API-Version: "v2"                # Add API version for backends
      X-Real-IP: "{{ .ClientIP }}"       # Dynamic value using Traefik templating
```

**Usage Notes:**
- Useful for backend routing and request identification
- Can pass client information to backend services
- Helps with microservice communication
- Supports Traefik templating for dynamic values

### 4. CORS (Cross-Origin Resource Sharing)

Configures cross-origin policies to allow or restrict web resources being requested from another domain.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: cors-headers
spec:
  headers:
    # Origins that are allowed to make requests
    accessControlAllowOriginList:
      - "https://app.example.com"
      - "https://admin.example.com"
    
    # Allow all origins with browser credentials
    # accessControlAllowOriginListRegex:
    #   - "^https://.*\.example\.com$"
    
    # HTTP methods allowed for CORS requests
    accessControlAllowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      - OPTIONS
    
    # Headers allowed to be sent in CORS requests
    accessControlAllowHeaders:
      - "Authorization"
      - "Content-Type"
      - "X-Requested-With"
    
    # Headers exposed to browser JavaScript
    accessControlExposeHeaders:
      - "Content-Length"
      - "X-Kuma-Revision"
    
    # Allow credentials (cookies, auth headers)
    accessControlAllowCredentials: true
    
    # How long browsers can cache CORS results (seconds)
    accessControlMaxAge: 3600
    
    # Add Vary: Origin header
    addVaryHeader: true
```

**Usage Notes:**
- `accessControlAllowOriginList`: Specific origins allowed to access the resource
- `accessControlAllowOriginListRegex`: Regex patterns for allowed origins
- `accessControlAllowMethods`: HTTP methods permitted for cross-origin requests
- `accessControlAllowHeaders`: Headers that can be used in the actual request
- `accessControlExposeHeaders`: Headers that browsers can access
- `accessControlAllowCredentials`: Whether requests can include credentials
- `accessControlMaxAge`: Duration browsers should cache preflight results
- `addVaryHeader`: Adds Vary: Origin header to improve caching behavior

### 5. SSL/TLS Configuration

Enforces secure HTTPS connections and configures TLS-related headers.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: ssl-headers
spec:
  headers:
    sslRedirect: true                  # Redirect HTTP to HTTPS
    sslTemporaryRedirect: false        # Use permanent redirects
    sslHost: "secure.example.com"      # Host to redirect to for SSL
    sslForceHost: true                 # Force SSL Host regardless of request
    stsSeconds: 31536000               # HSTS max-age
    stsIncludeSubdomains: true         # Include subdomains in HSTS
    stsPreload: true                   # Allow inclusion in browser preload lists
```

**Usage Notes:**
- `sslRedirect`: Forces clients to use HTTPS
- `sslTemporaryRedirect`: When false, uses 301 (permanent) redirects
- `sslHost`: Custom hostname for SSL redirects
- `stsSeconds`: How long browsers should remember to use HTTPS
- Complements Traefik's TLS configuration
- Helps achieve A+ ratings on SSL testing services

## Request Modification

### 1. URL Path Rewrite

Replaces the entire path of the request URL with a new path.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: path-rewrite
spec:
  replacePath:
    path: "/new-path"  # Fixed replacement path
```

**Usage Notes:**
- Completely replaces the request path with the specified value
- Query parameters are preserved
- Useful for legacy URL support or backend service compatibility
- Cannot use path variables or regex captures
- For more complex path manipulation, use `replacePathRegex`

### 2. Regex Path Rewrite

Replaces the path of the request URL using regular expression patterns and captures.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: regex-path-rewrite
spec:
  replacePathRegex:
    regex: "^/api/v1/([^/]+)/(.*)"  # Pattern with capture groups
    replacement: "/api/v2/$1/$2"     # Replacement with references to captures
```

**Usage Notes:**
- `regex`: Regular expression with capture groups
- `replacement`: New path that can reference capture groups with `$1`, `$2`, etc.
- More flexible than simple path replacement
- Useful for API versioning, URL normalization, or complex routing
- Be careful with regex patterns to avoid unexpected matches

### 3. Strip Prefix

Removes a specified prefix from the request path before forwarding to the backend service.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: strip-prefix
spec:
  stripPrefix:
    prefixes:             # List of prefixes to strip
      - "/api"
      - "/v1"
    forceSlash: true      # Ensures path starts with a slash after stripping
```

**Usage Notes:**
- Removes the specified prefixes from the beginning of the path
- Multiple prefixes can be specified in order of priority
- `forceSlash: true` ensures the resulting path starts with a slash
- Useful when backend services don't expect path prefixes
- Often used with Kubernetes Ingress to route different paths to different services

### 4. Strip Prefix Regex

Removes a prefix from the request path using a regular expression pattern.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: strip-prefix-regex
spec:
  stripPrefixRegex:
    regex:                # List of regex patterns to match and strip
      - "^/[a-z]{2}-[A-Z]{2}/.*"  # Matches language-country code prefixes
```

**Usage Notes:**
- Removes the part of the path that matches the regex pattern
- Multiple patterns can be specified in order of priority
- More flexible than `stripPrefix` for complex prefix patterns
- Useful for locale-based routing or dynamic path structures

### 5. Add Path Prefix

Prepends a path prefix to the request path before forwarding to the backend service.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: add-prefix
spec:
  addPrefix:
    prefix: "/api/v1"  # Prefix to add to the request path
```

**Usage Notes:**
- Adds the specified prefix to the beginning of the path
- Useful when backend services expect a specific path prefix
- Often used to route requests to the correct API version
- Complements `stripPrefix` for path transformations

### 6. Custom Request Headers

Adds or modifies HTTP headers in the request before forwarding to the backend service.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: add-request-headers
spec:
  headers:
    customRequestHeaders:
      X-Script-Name: "/my-app"                # Static header value
      X-Forwarded-For: "{{ .ClientIP }}"     # Dynamic value using Traefik templating
      X-Request-ID: "{{ .RequestID }}"       # Request ID for tracing
      X-Real-IP: "{{ .ClientIP }}"           # Client's real IP address
      X-Forwarded-Proto: "{{ .Request.Proto }}"  # Original protocol (HTTP/1.1, HTTP/2)
```

**Usage Notes:**
- Adds or replaces headers in the request
- Supports static values and dynamic values using Traefik templating
- Useful for passing additional information to backend services
- Common use cases include:
  - Adding authentication information
  - Setting proxy-related headers
  - Passing client information
  - Request tracing and correlation

### 7. Query Manipulation

Modifies query parameters in the request URL.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: query-manipulator
spec:
  queryString:
    add:                      # Add new query parameters
      version: "v2"
      source: "traefik"
    set:                      # Set/replace existing parameters
      format: "json"
    remove:                   # Remove parameters
      - "debug"
      - "trace"
```

**Usage Notes:**
- `add`: Adds new query parameters without modifying existing ones
- `set`: Sets or replaces existing query parameters
- `remove`: Removes specified query parameters
- Useful for standardizing API requests
- Can be used to add default parameters or remove sensitive information

## Response Modification

### 1. Custom Response Headers

Adds or modifies HTTP headers in the response sent back to the client.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: add-response-headers
spec:
  headers:
    customResponseHeaders:
      X-Powered-By: "Traefik"
      X-Custom-Response: "Value"
      X-Response-Time: "{{ .ResponseTime }}"          # Dynamic value using Traefik templating
      X-Server-ID: "{{ env "HOSTNAME" }}"           # Environment variable
      Cache-Control: "public, max-age=3600"           # Cache control directives
```

**Usage Notes:**
- Adds or replaces headers in the response
- Supports static values and dynamic values using Traefik templating
- Useful for adding security headers, cache control, or custom metadata
- Can be used to remove sensitive headers by setting empty values
- Common use cases include:
  - Adding security headers
  - Setting cache control directives
  - Adding server identification
  - Response timing information

### 2. Compress Responses

Compresses HTTP responses to reduce bandwidth usage and improve load times.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: compress
spec:
  compress:
    excludedContentTypes:           # MIME types to exclude from compression
      - "image/jpeg"
      - "image/png"
      - "application/zip"
    minResponseBodyBytes: 1024      # Only compress responses larger than this size (bytes)
```

**Usage Notes:**
- Automatically compresses responses using gzip or deflate based on client Accept-Encoding
- Empty configuration `compress: {}` uses sensible defaults
- `excludedContentTypes`: MIME types that should not be compressed (already compressed formats)
- `minResponseBodyBytes`: Minimum response size to compress (avoids overhead for small responses)
- Significantly reduces bandwidth usage for text-based responses (HTML, CSS, JS, JSON)
- Browsers automatically decompress the content

### 3. Buffering

Buffers the response from the backend service before sending it to the client.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: response-buffer
spec:
  buffering:
    maxRequestBodyBytes: 10485760    # 10MB max request body to buffer
    memRequestBodyBytes: 2097152     # 2MB max request body to keep in memory
    maxResponseBodyBytes: 10485760   # 10MB max response body to buffer
    memResponseBodyBytes: 2097152    # 2MB max response body to keep in memory
    retryExpression: "IsNetworkError() && Attempts() < 3"  # Retry condition
```

**Usage Notes:**
- Buffers the entire request/response before forwarding
- Allows for retrying failed requests without client awareness
- Useful for unreliable backends or when modifying response bodies
- `maxRequestBodyBytes`/`maxResponseBodyBytes`: Maximum size to buffer
- `memRequestBodyBytes`/`memResponseBodyBytes`: Maximum size to keep in memory (larger values use disk)
- `retryExpression`: Condition for retrying the request to the backend

### 4. Error Pages

Provides custom error pages for specific HTTP status codes.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-error-pages
spec:
  errors:
    status:                          # HTTP status codes to handle
      - "500-599"                   # Server errors
      - "404"                       # Not found
    service:
      name: error-pages-service      # Kubernetes service for error pages
      port: 80
    query: "/{status}.html"         # Path format for the error service
```

**Usage Notes:**
- Redirects to a custom error page service when specified status codes occur
- `status`: List of status codes or ranges to handle
- `service`: Kubernetes service that hosts the error pages
- `query`: Path format for the error service, with `{status}` placeholder
- Improves user experience by providing friendly error pages
- Can include branding, helpful information, and navigation options

## Observability & Logging

### 1. Access Logs

Configures detailed access logging for requests passing through the middleware.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: access-logs
spec:
  plugin:
    traefik-logs:
      filePath: "/var/log/traefik/access.log"  # Log file path
      format: "json"                         # Log format (json or common)
      bufferingSize: 100                     # Number of log lines to buffer
```

**Usage Notes:**
- Requires the Traefik Pilot plugin system or Traefik Enterprise
- `filePath`: Where to write the access logs
- `format`: Log format (json, common, or custom format)
- `bufferingSize`: Buffer size for better performance
- Useful for debugging, auditing, and compliance
- Consider log rotation for production environments

### 2. Request ID

Adds a unique identifier to each request for tracing and correlation.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: request-id
spec:
  plugin:
    requestId:
      headerName: "X-Request-ID"             # Header name for the request ID
      format: "uuid-v4"                      # ID format (uuid-v4, uuid-v5, etc.)
      propagate: true                        # Forward the ID to backend services
```

**Usage Notes:**
- Requires the Traefik Pilot plugin system or Traefik Enterprise
- `headerName`: HTTP header to store the request ID
- `format`: Format of the generated ID
- `propagate`: Whether to forward the ID to backend services
- Essential for distributed tracing and request correlation
- Helps with troubleshooting and performance analysis

### 3. Metrics (Prometheus)

Exposes metrics for monitoring and observability.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: metrics
spec:
  plugin:
    traefik-prometheus:
      buckets: [0.1, 0.3, 1.2, 5.0]         # Response time buckets in seconds
      addEntryPointsLabels: true             # Add entrypoint labels to metrics
      addServicesLabels: true                # Add service labels to metrics
      addRoutersLabels: true                 # Add router labels to metrics
      namespace: "traefik"                   # Prometheus namespace for metrics
```

**Usage Notes:**
- Requires the Traefik Pilot plugin system or Traefik Enterprise
- `buckets`: Histogram buckets for response time measurements
- `addXLabels`: Whether to add specific labels to metrics
- `namespace`: Prefix for metric names
- Enables monitoring of request counts, response times, error rates, etc.
- Can be integrated with Prometheus and Grafana for visualization

## IP Filtering

### 1. IP Whitelisting

Restricts access to specific IP addresses or CIDR ranges.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: ip-whitelist
spec:
  ipWhiteList:
    sourceRange:                            # List of allowed IP ranges
      - "127.0.0.1/32"                      # Localhost
      - "192.168.1.0/24"                    # Local network
      - "10.0.0.0/8"                        # Private network
    ipStrategy:                             # Optional: IP extraction strategy
      depth: 1                              # X-Forwarded-For depth
      excludedIPs:                          # IPs to ignore when using depth
        - "10.0.0.1"
```

**Usage Notes:**
- `sourceRange`: CIDR notation for allowed IP addresses
- `ipStrategy`: How to determine the client's IP address
- `depth`: Position in X-Forwarded-For header to use (for proxied requests)
- `excludedIPs`: IP addresses to ignore when using depth
- Useful for restricting access to admin interfaces or internal APIs
- Returns 403 Forbidden for non-matching IPs

### 2. IP Blacklisting

Blocks access from specific IP addresses or CIDR ranges.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: ip-blacklist
spec:
  plugin:
    ipBlacklist:
      sourceRange:                          # List of blocked IP ranges
        - "1.2.3.4/32"                      # Single IP address
        - "5.6.7.0/24"                      # IP range
      ipStrategy:                           # Optional: IP extraction strategy
        depth: 0                            # Use the remote address
        excludedIPs: []                     # No IPs to exclude
```

**Usage Notes:**
- Requires the Traefik Pilot plugin system or Traefik Enterprise
- `sourceRange`: CIDR notation for blocked IP addresses
- `ipStrategy`: How to determine the client's IP address
- Useful for blocking known malicious IPs or abusive users
- Can be combined with rate limiting for comprehensive protection
- Returns 403 Forbidden for matching IPs

## Error Handling

### 1. Custom Error Pages

Provides custom error pages for specific HTTP status codes.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-errors
spec:
  errors:
    status:                                 # HTTP status codes to handle
      - "400-499"                           # Client errors
      - "500-599"                           # Server errors
    service:                                # Service that provides error pages
      name: error-pages-service
      port: 80
    query: "/error/{status}.html"           # Path format for error pages
```

**Usage Notes:**
- `status`: List of status codes or ranges to handle
- `service`: Kubernetes service that hosts the error pages
- `query`: Path format for the error service, with `{status}` placeholder
- Improves user experience by providing friendly error pages
- Can include branding, helpful information, and navigation options

### 2. Retry on Error

Automatically retries failed requests to improve resilience.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: retry-on-error
spec:
  retry:
    attempts: 3                             # Number of retry attempts
    initialInterval: "100ms"                # Initial delay between retries
    retryOnStatusCodes:                     # Status codes that trigger retries
      - 500
      - 502
      - 503
      - 504
```

**Usage Notes:**
- `attempts`: Maximum number of attempts (including the initial request)
- `initialInterval`: Starting delay between retries (uses exponential backoff)
- `retryOnStatusCodes`: HTTP status codes that should trigger a retry
- Only retries idempotent methods (GET, HEAD, PUT, DELETE, OPTIONS, TRACE) by default
- Improves reliability without requiring client-side retry logic

### 3. Circuit Breaker

Protects backend services by temporarily stopping traffic when error rates exceed thresholds.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.5 || ResponseCodeRatio(500, 600, 0, 600) > 0.5"
    checkPeriod: "10s"                      # How often to check the expression
    fallbackDuration: "30s"                 # How long to keep circuit open after triggering
```

**Usage Notes:**
- `expression`: Condition that triggers the circuit breaker
- Common metrics in expressions:
  - `NetworkErrorRatio()`: Ratio of network errors
  - `ResponseCodeRatio(min1, max1, min2, max2)`: Ratio of responses with status codes in range [min1:max1] vs [min2:max2]
  - `LatencyAtQuantileMS(quantile)`: Response time in milliseconds at given quantile
  - `ResponseCodeCount(code, period)`: Count of responses with given status code
- When triggered, returns 503 Service Unavailable to clients
- Prevents cascading failures and allows backend services time to recover

## Middleware Chaining

### 1. Basic Middleware Chain

Combines multiple middlewares into a single, reusable chain.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: combined-middleware
spec:
  chain:
    middlewares:                            # List of middlewares to chain
      - name: basic-auth                    # Authentication first
      - name: rate-limit                    # Then rate limiting
      - name: compress                      # Finally compression
```

**Usage Notes:**
- `middlewares`: List of middleware names to chain together
- Order matters: middlewares are applied in the order listed
- Simplifies configuration by grouping common middleware combinations
- Middlewares must be defined in the same namespace or referenced with namespace prefix
- Useful for applying consistent security and performance policies

### 2. Advanced Middleware Chain

Creates a complex chain with middlewares from different namespaces.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: advanced-chain
  namespace: default
spec:
  chain:
    middlewares:                            # List of middlewares to chain
      - name: security-headers              # Apply security headers first
        namespace: security                 # From security namespace
      - name: rate-limit                    # Then rate limiting
        namespace: default                  # From default namespace
      - name: compress                      # Finally compression
        namespace: optimization             # From optimization namespace
```

**Usage Notes:**
- `name`: Name of the middleware to include in the chain
- `namespace`: Namespace where the middleware is defined (optional)
- Enables organization of middlewares by function or team
- Allows reuse of middlewares across different namespaces
- Provides flexibility for complex deployment scenarios

## Implementation Notes

### 1. Attaching Middleware to an Ingress

Applies middleware to a Kubernetes Ingress resource.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    # Format: <namespace>-<middleware-name>@kubernetescrd
    traefik.ingress.kubernetes.io/router.middlewares: default-basic-auth@kubernetescrd,default-rate-limit@kubernetescrd
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

**Usage Notes:**
- Use the annotation `traefik.ingress.kubernetes.io/router.middlewares` to attach middlewares
- Format: `<namespace>-<middleware-name>@kubernetescrd`
- Multiple middlewares are separated by commas
- Order matters: middlewares are applied from left to right
- Middlewares must be defined before they can be referenced

### 2. Attaching Middleware to an IngressRoute

Applies middleware to a Traefik IngressRoute resource.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: my-ingressroute
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`example.com`)
    kind: Rule
    services:
    - name: my-service
      port: 80
    middlewares:
    - name: basic-auth                      # Reference to middleware in same namespace
    - name: rate-limit
    - name: security-headers@security       # Reference to middleware in different namespace
```

**Usage Notes:**
- Use the `middlewares` field in the route definition
- Format for same namespace: `name: <middleware-name>`
- Format for different namespace: `name: <middleware-name>@<namespace>`
- Order matters: middlewares are applied in the order listed
- More flexible and powerful than Ingress annotations
- Native to Traefik CRD architecture

### 3. Testing Middlewares

Best practices for testing and debugging Traefik middlewares.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: test-middleware
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`test.example.com`)
    kind: Rule
    services:
    - name: whoami                          # Simple echo service for testing
      port: 80
    middlewares:
    - name: middleware-to-test              # Reference to middleware being tested
```

**Usage Notes:**
- Use a simple echo service like `whoami` for testing
- Test one middleware at a time to isolate issues
- Check Traefik logs for middleware errors
- Use `curl` with verbose output to inspect headers and responses
- Consider using a dedicated test domain or path
- Validate middleware behavior before applying to production traffic

### 4. Middleware Best Practices

- **Security First**: Apply authentication and security headers before other middlewares
- **Performance Optimization**: Apply compression and caching after security middlewares
- **Error Handling**: Place retry and circuit breaker middlewares close to the backend service
- **Namespace Organization**: Group middlewares by function (security, performance, routing)
- **Reusability**: Create generic middlewares that can be reused across multiple services
- **Documentation**: Add comments to middleware definitions explaining their purpose and configuration
- **Monitoring**: Set up alerts for middleware failures and performance issues
- **Testing**: Thoroughly test middlewares in a staging environment before deploying to production
- **Versioning**: Use version labels or namespaces to manage middleware versions
- **Gradual Rollout**: Implement new middlewares on a subset of traffic before full deployment

## Other Useful Middleware

### 1. Regex-based Redirect (redirectRegex)

Provides powerful regex-based URL redirection with pattern matching and capture groups.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-regex
spec:
  redirectRegex:
    regex: "^https://app\.example\.com/([a-z]+)/([0-9]+)/(.*)$"
    replacement: "https://app.example.com/$1/$3?id=$2"
    permanent: true
```

**Usage Notes:**
- More flexible than simple redirectScheme middleware
- Uses regex capture groups (parentheses) to extract parts of the URL
- Referenced in replacement with $1, $2, etc.
- Useful for URL restructuring, API versioning, and domain migrations
- Test regex patterns carefully to avoid redirect loops

### 2. Retry Failed Requests (retry)

Automatically retries failed requests to improve resilience against transient errors.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: retry-middleware
spec:
  retry:
    attempts: 3                  # Number of retry attempts
    initialInterval: 100ms       # Time between retries (increases exponentially)
    retryOnStatusCodes:          # Status codes that trigger retries
      - 500
      - 502
      - 503
      - 504
```

**Usage Notes:**
- Only retries idempotent methods (GET, HEAD, PUT, DELETE, OPTIONS, TRACE) by default
- `attempts`: Maximum number of attempts (including the initial request)
- `initialInterval`: Starting delay between retries (uses exponential backoff)
- `retryOnStatusCodes`: HTTP status codes that should trigger a retry
- Improves reliability without requiring client-side retry logic
- Be cautious with high attempt values to avoid overwhelming struggling backends

### 3. Custom Error Pages (errorPage)

Displays custom error pages when backend services return specific error codes.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-error-pages
spec:
  errors:
    status:
      - "500-599"                # Error code range
    service:
      name: error-pages-service  # Service that serves error pages
      port: 80
    query: "/error/{status}.html" # Path format on the error service
```

**Usage Notes:**
- Requires a service that hosts your custom error pages
- `status`: Can be specific codes ("404", "500") or ranges ("500-599")
- `query`: Supports {status} placeholder for the actual error code
- Improves user experience by providing helpful error information
- Can include branding, support contact info, and self-help resources

### 4. Strip Prefixes via Regex (stripPrefixRegex)

Removes URL path prefixes that match a regex pattern before forwarding to backend services.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: strip-prefix-regex
spec:
  stripPrefixRegex:
    regex:
      - "^/api/v[0-9]+/"  # Strips API version prefix
```

**Usage Notes:**
- More flexible than stripPrefix for dynamic path handling
- Useful for API versioning where backend doesn't need version info
- Can use multiple regex patterns for complex path manipulations
- Helps maintain clean internal routing while supporting external URL conventions

### 5. Add Prefix to URL Path (addPrefix)

Prepends a prefix to the URL path before forwarding to backend services.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: add-prefix
spec:
  addPrefix:
    prefix: "/api"  # Prefix to add to all incoming requests
```

**Usage Notes:**
- Useful when backend services expect a specific path prefix
- Can route external requests to internal API paths
- Often used with path-based routing to map clean URLs to backend structures
- Simple but powerful for service composition and API gateway patterns

### 6. HTTP Digest Authentication (digestAuth)

Provides more secure authentication than Basic Auth by not transmitting passwords in plaintext.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: digest-auth
spec:
  digestAuth:
    secret: digest-auth-secret  # Reference to a Kubernetes secret
    removeHeader: false         # Optional: keeps auth headers for backend services
    realm: "My Secure API"     # Required: realm for digest authentication
```

**Creating the Secret:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: digest-auth-secret
type: Opaque
data:
  users: <base64-encoded-htdigest-content>
```

**Usage Notes:**
- The secret must contain a key named `users` with htdigest formatted content
- Generate with: `htdigest -c digestfile "realm-name" username`
- More secure than Basic Auth as it uses a challenge-response mechanism
- The realm name in the middleware must match the realm used when generating credentials

### 7. Whitelist IPs (whitelist)

Restricts access to specific IP addresses or CIDR ranges.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: ip-whitelist
spec:
  ipWhiteList:
    sourceRange:
      - "127.0.0.1/32"          # Localhost
      - "10.0.0.0/8"            # Private network range
      - "192.168.1.0/24"        # Local subnet
    ipStrategy:
      depth: 1                  # Optional: for handling proxies
      excludedIPs:              # Optional: IPs to ignore when using depth
        - "127.0.0.1"
```

**Usage Notes:**
- Provides network-level access control
- The `depth` parameter helps with X-Forwarded-For header handling in proxy environments
- Can be combined with other auth methods for defense in depth
- Useful for admin interfaces or internal APIs

### 8. Rate Limit Headers (rateLimitHeaders)

Adds rate limit information headers to responses without actually limiting requests.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-headers
spec:
  rateLimit:
    average: 100
    burst: 50
    period: 1m
    sourceCriterion:
      ipStrategy:
        depth: 1
    addRateLimitHeaders: true  # Adds headers without blocking requests
```

**Usage Notes:**
- Adds headers like `X-Rate-Limit-Limit`, `X-Rate-Limit-Remaining`, and `X-Rate-Limit-Reset`
- Useful for API clients to understand their usage limits
- Can be used in monitoring mode before enforcing actual rate limits
- Helps clients implement their own rate limiting or backoff strategies

### 9. Buffer Requests (buffer)

Buffers the entire request before forwarding it to the backend service.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: buffer-middleware
spec:
  buffering:
    maxRequestBodyBytes: 10485760  # 10MB max request size
    memRequestBodyBytes: 2097152   # 2MB in memory before using disk
    maxResponseBodyBytes: 10485760 # 10MB max response size
    memResponseBodyBytes: 2097152  # 2MB in memory before using disk
    retryExpression: "IsNetworkError() && Attempts() <= 2"  # Retry logic
```

**Usage Notes:**
- Protects backends from slow clients (slow loris attacks)
- Useful for services that expect fast transmission of request body
- Can retry requests if backend fails after receiving partial request
- Adds memory/disk overhead on the Traefik side
- `retryExpression` allows conditional retry logic

### 10. Forward Auth to External Service (forwardAuth)

Delegates authentication to an external service, enabling complex auth flows like OAuth, OIDC, or custom auth systems.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: forward-auth
spec:
  forwardAuth:
    address: "https://auth.example.com/verify"  # External auth service endpoint
    trustForwardHeader: true                    # Trust X-Forwarded-* headers
    authResponseHeaders:                        # Headers to copy from auth response
      - "X-Auth-User"
      - "X-Auth-Email"
      - "X-Auth-Roles"
    authRequestHeaders:                         # Headers to forward to auth service
      - "Authorization"
      - "Cookie"
    tls:                                        # Optional TLS configuration
      caSecret: auth-ca-cert
      insecureSkipVerify: false
```

**Usage Notes:**
- The external auth service must return HTTP 2xx for successful auth, anything else is considered a failure
- `authResponseHeaders` defines which headers from the auth service get forwarded to backend services
- `authRequestHeaders` specifies which headers from the original request are sent to the auth service
- Enables integration with identity providers like Auth0, Okta, Keycloak, etc.
- Can implement complex authorization logic beyond simple username/password

### 11. Enable Gzip Compression (compress)

Compresses responses using gzip to reduce bandwidth and improve loading times.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: compress-middleware
spec:
  compress:
    excludedContentTypes:  # MIME types to exclude from compression
      - "image/jpeg"
      - "image/png"
      - "application/zip"
    minResponseBodyBytes: 1024  # Only compress responses larger than this
```

**Usage Notes:**
- Significantly reduces bandwidth for text-based content (HTML, CSS, JS, JSON)
- `excludedContentTypes`: MIME types that shouldn't be compressed (already compressed formats)
- `minResponseBodyBytes`: Minimum size threshold for compression
- Improves page load times and reduces bandwidth costs
- Slight CPU overhead on the Traefik side

### 12. Add/Remove Headers (customHeaders)

Adds, modifies, or removes HTTP headers in requests and responses.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-headers
spec:
  headers:
    customRequestHeaders:  # Headers added/modified in requests to backends
      X-Real-IP: "{{ .ClientIP }}"
      X-Forwarded-Prefix: "/api"
    customResponseHeaders:  # Headers added/modified in responses to clients
      X-Application-Version: "v1.2.3"
      Server: "CustomServer"
    # Remove headers by setting empty value
    customResponseHeaders:
      X-Powered-By: ""
```

**Usage Notes:**
- Useful for adding application metadata, security headers, or debugging info
- Can remove sensitive headers that might leak implementation details
- Supports Traefik templating for dynamic values
- Helps with microservice communication and client information

### 13. Basic Auth with Multiple Users (basicauth)

Implements HTTP Basic Authentication to protect routes with username/password credentials for multiple users.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: basic-auth-multi
spec:
  basicAuth:
    secret: basic-auth-secret  # Reference to a Kubernetes secret
    removeHeader: false        # Optional: keeps the Authorization header for backend services
    realm: "My Secure API"    # Optional: realm name shown in browser auth prompt
```

**Creating the Secret with Multiple Users:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
type: Opaque
data:
  users: <base64-encoded-htpasswd-content-with-multiple-users>
```

**Usage Notes:**
- The secret must contain a key named `users` with htpasswd formatted content
- Generate credentials with: `htpasswd -nb username1 password1 && htpasswd -nb username2 password2`
- Each user should be on a separate line in the htpasswd file
- Simple to implement but transmits credentials with minimal encryption (Base64)
- Suitable for internal tools or development environments

### 14. Circuit Breaker Based on Failure Ratio (circuitBreaker)

Protects backend services by temporarily stopping traffic when error rates exceed thresholds.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.5 || ResponseCodeRatio(500, 600, 0, 600) > 0.5"
    checkPeriod: 10s        # Optional: how often to check the expression
    fallbackDuration: 30s   # Optional: how long to keep circuit open after triggering
```

**Usage Notes:**
- `expression`: Condition that triggers the circuit breaker
- Common metrics in expressions:
  - `NetworkErrorRatio()`: Ratio of network errors
  - `ResponseCodeRatio(min1, max1, min2, max2)`: Ratio of responses with status codes in range [min1:max1] vs [min2:max2]
  - `LatencyAtQuantileMS(quantile)`: Response time in milliseconds at given quantile
  - `ResponseCodeCount(code, period)`: Count of responses with given status code
- When triggered, returns 503 Service Unavailable to clients
- Prevents cascading failures and allows backend services time to recover

### 15. OAuth2 Proxy (oauth2)

Implements OAuth2 authentication flow using an external provider.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: oauth2-auth
spec:
  forwardAuth:
    address: "https://oauth-proxy.example.com/oauth2/auth"
    trustForwardHeader: true
    authResponseHeaders:
      - "X-Auth-Request-Access-Token"
      - "X-Auth-Request-User"
      - "X-Auth-Request-Email"
      - "Authorization"
```

**Usage Notes:**
- Requires an OAuth2 proxy service like oauth2-proxy, Dex, or a custom implementation
- Enables authentication with providers like Google, GitHub, Microsoft, etc.
- The proxy handles the OAuth flow and token validation
- More complex to set up but provides better security and user experience than Basic Auth

### 16. Client IP Source Strategy (ipStrategy)

Configures how Traefik determines the client's real IP address in proxy environments.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: ip-strategy
spec:
  ipWhiteList:  # Used here as an example, but applies to any middleware using client IP
    sourceRange:
      - "192.168.1.0/24"
    ipStrategy:
      depth: 2                # Use the 2nd IP in X-Forwarded-For
      excludedIPs:            # IPs to skip when counting depth
        - "10.0.0.1"
        - "10.0.0.2"
```

**Usage Notes:**
- `depth`: Position in X-Forwarded-For header (0 means use the last IP)
- `excludedIPs`: IPs to ignore when calculating depth (typically load balancers or proxies)
- Critical for accurate client identification in multi-proxy environments
- Used by rate limiting, IP filtering, and logging middlewares
- Helps prevent IP spoofing in security-sensitive applications

### 17. Number of Retries (retryAttempt)

Specifies the number of retry attempts for failed requests.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: retry-attempts
spec:
  retry:
    attempts: 5                  # Maximum number of retry attempts
    initialInterval: 100ms       # Initial delay between retries
```

**Usage Notes:**
- Part of the retry middleware configuration
- Higher values increase resilience but may delay error responses
- Consider backend capacity when setting high values
- Useful for handling transient network issues or backend restarts

### 18. Rate Limit Per Time Period (rateLimitPeriod)

Limits requests based on a specific time window.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-period
spec:
  rateLimit:
    average: 100      # Average requests per period
    burst: 50         # Maximum initial burst size
    period: 1m        # Time period (1 minute in this example)
```

**Usage Notes:**
- `period`: Time window for rate limiting (valid units: s, m, h)
- Longer periods are useful for APIs with expected bursts of activity
- For a 1-minute period with average 100, clients can make 100 requests per minute
- More intuitive for API rate limits than requests-per-second

### 19. Modify Response Body (responseModifier)

Experimental middleware to modify the response body content.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: response-modifier
spec:
  plugin:
    responseModifier:
      rewrites:
        - regex: "<title>(.*)</title>"
          replacement: "<title>Modified: $1</title>"
```

**Usage Notes:**
- Experimental feature requiring Traefik Enterprise or custom plugins
- Useful for content transformation, branding, or fixing legacy responses
- Can impact performance for large responses
- Use with caution in production environments

### 20. Modify Request Body (requestModifier)

Experimental middleware to modify the request body content.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: request-modifier
spec:
  plugin:
    requestModifier:
      rewrites:
        - regex: "\"apiVersion\":\s*\"v1\""
          replacement: "\"apiVersion\":\"v2\""
```

**Usage Notes:**
- Experimental feature requiring Traefik Enterprise or custom plugins
- Useful for API version translation or request normalization
- Can impact performance for large requests
- Use with caution in production environments

### 21. Expose Metrics (prometheusExporter)

Exposes Traefik metrics in Prometheus format.

```yaml
# This is typically configured in Traefik's static configuration
# rather than as a middleware
metrics:
  prometheus:
    addEntryPointsLabels: true
    addServicesLabels: true
    entryPoint: metrics
```

**Usage Notes:**
- Configured in Traefik's static configuration, not as a standard middleware
- Exposes metrics on a dedicated entrypoint (typically on port 8082)
- Provides valuable data for monitoring and alerting
- Can be scraped by Prometheus for visualization in Grafana

### 22. Redirect on Request (requestRedirect)

Redirects requests based on request properties.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: request-redirect
spec:
  redirectRegex:
    regex: "^(.*)\?redirect=(.*)$"
    replacement: "$2"
    permanent: false
```

**Usage Notes:**
- Redirects based on request parameters or patterns
- Useful for implementing redirect services or URL shorteners
- Can extract redirect targets from query parameters
- Set `permanent: false` for temporary redirects (HTTP 302)

### 23. Forward Response Headers (forwardResponseHeaders)

Forwards specific headers from backend responses to clients.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: forward-response-headers
spec:
  headers:
    customResponseHeaders:
      X-Custom-Backend-Header: "{{ .ResponseHeader.Get \"X-Backend-Info\" }}"
```

**Usage Notes:**
- Allows selective forwarding of backend headers
- Useful for preserving backend metadata or debugging information
- Can rename headers during forwarding
- Supports templating for dynamic values

### 24. Forward Request Headers (forwardRequestHeaders)

Forwards specific headers from client requests to backend services.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: forward-request-headers
spec:
  headers:
    customRequestHeaders:
      X-Forwarded-User: "{{ .Request.Header.Get \"User\" }}"
      X-Original-Method: "{{ .Request.Method }}"
```

**Usage Notes:**
- Allows selective forwarding of client headers
- Useful for passing client context to backend services
- Can rename headers during forwarding
- Supports templating for dynamic values

### 25. Show Custom Error Page (customErrorPage)

Displays custom error pages for specific HTTP error codes.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: custom-error-page
spec:
  errors:
    status:
      - "404"
      - "500-599"
    service:
      name: error-pages-service
      port: 80
    query: "/error/{status}.html"
```

**Usage Notes:**
- Similar to errorPage middleware but with more specific configuration
- Requires a service hosting your custom error pages
- `status`: Specific error codes or ranges to handle
- `query`: Path format on the error service, with {status} placeholder
- Improves user experience with branded, helpful error pages

### 26. WebAuthn Authentication (webAuthn)

Implements WebAuthn (FIDO2) passwordless authentication.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: webauthn-auth
spec:
  plugin:
    webauthn:
      relyingPartyID: "example.com"
      relyingPartyName: "Example Corp"
      relyingPartyOrigins:
        - "https://app.example.com"
      userVerification: "preferred"
      sessionDuration: "24h"
```

**Usage Notes:**
- Requires a Traefik plugin or Enterprise edition
- Enables passwordless authentication using security keys or biometrics
- More secure than password-based authentication
- Requires client support for WebAuthn (modern browsers)
- Complex to set up but provides excellent security

### 27. Load Balancing Strategy (loadBalancer)

Configures how requests are distributed among backend servers.

```yaml
# This is typically configured in Traefik's service definition
# rather than as a middleware
apiVersion: traefik.containo.us/v1alpha1
kind: TraefikService
metadata:
  name: service-with-lb
spec:
  weighted:
    services:
      - name: service1
        port: 80
        weight: 3
      - name: service2
        port: 80
        weight: 1
    sticky:
      cookie:
        name: lb-cookie
        secure: true
        httpOnly: true
```

**Usage Notes:**
- Configured in service definition, not as a standard middleware
- `weight`: Relative traffic distribution (service1 gets 75%, service2 gets 25%)
- `sticky`: Enables session stickiness using cookies
- Useful for canary deployments, A/B testing, or blue/green deployments
- Can implement various load balancing algorithms

### 28. WebSocket Support (websocket)

Enables WebSocket protocol support for real-time bidirectional communication.

```yaml
# WebSocket support is built into Traefik and doesn't require
# a specific middleware configuration
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: websocket-route
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`ws.example.com`)
    kind: Rule
    services:
    - name: websocket-service
      port: 80
```

**Usage Notes:**
- WebSocket support is built-in and doesn't require specific middleware
- Traefik automatically detects and handles WebSocket protocol upgrades
- Supports both ws:// and wss:// (secure WebSockets)
- Useful for chat applications, real-time dashboards, and collaborative tools
- Maintains persistent connections, so consider resource implications

### 29. Cross-Origin Resource Sharing Headers (cors)

Configures cross-origin policies to allow or restrict web resources being requested from another domain.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: cors-headers
spec:
  headers:
    # Origins that are allowed to make requests
    accessControlAllowOriginList:
      - "https://app.example.com"
      - "https://admin.example.com"
    
    # Allow all origins with browser credentials
    # accessControlAllowOriginListRegex:
    #   - "^https://.*\.example\.com$"
    
    # HTTP methods allowed for CORS requests
    accessControlAllowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      - OPTIONS
    
    # Headers allowed to be sent in CORS requests
    accessControlAllowHeaders:
      - "Authorization"
      - "Content-Type"
      - "X-Requested-With"
    
    # Headers exposed to browser JavaScript
    accessControlExposeHeaders:
      - "Content-Length"
      - "X-Kuma-Revision"
    
    # Allow credentials (cookies, auth headers)
    accessControlAllowCredentials: true
    
    # How long browsers can cache CORS results (seconds)
    accessControlMaxAge: 3600
    
    # Add Vary: Origin header
    addVaryHeader: true
```

**Usage Notes:**
- `accessControlAllowOriginList`: Specific origins allowed to access the resource
- `accessControlAllowOriginListRegex`: Regex patterns for allowed origins
- `accessControlAllowMethods`: HTTP methods permitted for cross-origin requests
- `accessControlAllowHeaders`: Headers that can be used in the actual request
- `accessControlExposeHeaders`: Headers that browsers can access
- `accessControlAllowCredentials`: Whether requests can include credentials
- `accessControlMaxAge`: Duration browsers should cache preflight results
- `addVaryHeader`: Adds Vary: Origin header to improve caching behavior

### 30. Distributed Tracing Support (openTracing)

Enables distributed tracing for request flows across microservices.

```yaml
# This is typically configured in Traefik's static configuration
# rather than as a middleware
tracing:
  serviceName: traefik
  spanNameLimit: 100
  jaeger:
    samplingServerURL: http://jaeger:5778/sampling
    samplingType: const
    samplingParam: 1.0
    localAgentHostPort: jaeger:6831
    traceContextHeaderName: uber-trace-id
    disableAttemptReconnecting: true
```

**Usage Notes:**
- Configured in Traefik's static configuration, not as a standard middleware
- Supports various tracing backends (Jaeger, Zipkin, Datadog, etc.)
- Provides end-to-end visibility of request flows across services
- Helps with performance analysis and troubleshooting
- Minimal performance impact when properly configured

## Attach Middleware to Ingress

```yaml
metadata:
  annotations:
    traefik.ingress.kubernetes.io/router.middlewares: default-redirect-to-https@kubernetescrd,default-basic-auth@kubernetescrd
```

Final Notes
- For basicAuth and digestAuth, create secrets with user credentials.
- Middleware CRDs are namespaced; make sure to use correct namespaces in annotations.
- Not all middleware are enabled by default; ensure your Traefik version supports the ones you need.
- Middleware chaining allows powerful composition of behaviors.
- Check [Traefik official docs](https://doc.traefik.io/traefik/middlewares/){:target="_blank"} for latest and experimental features.