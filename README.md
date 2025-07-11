# Reclaim Interceptor

A powerful, middleware-based HTTP request/response interceptor for web browsers. This userscript provides robust interception capabilities for both `fetch` and `XMLHttpRequest` APIs, allowing you to monitor, modify, and analyze all HTTP traffic in web applications.

## 🚀 Features

### Core Capabilities

- **Universal HTTP Interception**: Intercepts both `fetch` and `XMLHttpRequest` APIs
- **Middleware Architecture**: Plugin-based system for processing requests and responses
- **Multiple Interception Methods**: Choose between Proxy, Direct Replacement, or Getter/Setter approaches
- **Request/Response Tracking**: Unique ID tracking for each request throughout its lifecycle
- **Content-Type Aware Parsing**: Intelligent parsing of JSON, text, and binary responses
- **MSW-Inspired Architecture**: Modern, clean design patterns based on Mock Service Worker
- **Zero Dependencies**: Pure JavaScript with no external dependencies

### Advanced Features

- **Override-Friendly**: Allows other scripts to override fetch while maintaining interception
- **Cleanup System**: Proper restoration of original browser APIs
- **Error Resilience**: Robust error handling for edge cases
- **Prototype Chain Support**: Works across different JavaScript contexts
- **Body Stream Protection**: Handles already-consumed response bodies gracefully

## 📦 Installation

### Tampermonkey/Greasemonkey

1. Install [Tampermonkey](https://www.tampermonkey.net/) or [Greasemonkey](https://www.greasespot.net/)
2. Copy the `script.js` content
3. Create a new userscript and paste the code
4. Save and enable the script

### Manual Integration

```javascript
// Include the script in your project
<script src="path/to/script.js"></script>;

// Access the interceptor globally
window.reclaimInterceptor.addRequestMiddleware(/* your middleware */);
```

## 🔧 Configuration

### Basic Setup

```javascript
const interceptor = new RequestInterceptor({
  disableFetch: false, // Enable/disable fetch interception
  disableXHR: false, // Enable/disable XHR interception
  useProxyForFetch: true, // Use Proxy-based fetch interception (default)
  useGetterForFetch: false, // Use getter/setter approach (most robust)
});
```

### Interception Methods

#### 1. Proxy Method (Default)

```javascript
const interceptor = new RequestInterceptor({
  useProxyForFetch: true,
  useGetterForFetch: false,
});
```

- **Pros**: Clean, preserves original function behavior
- **Cons**: May not work in all environments
- **Best for**: Modern browsers, standard use cases

#### 2. Getter/Setter Method (Most Robust)

```javascript
const interceptor = new RequestInterceptor({
  useProxyForFetch: false,
  useGetterForFetch: true,
});
```

- **Pros**: Maximum compatibility, prevents override conflicts
- **Cons**: Slightly more complex
- **Best for**: Production environments, maximum reliability

#### 3. Direct Replacement Method

```javascript
const interceptor = new RequestInterceptor({
  useProxyForFetch: false,
  useGetterForFetch: false,
});
```

- **Pros**: Simple, fast
- **Cons**: Can be overridden by other scripts
- **Best for**: Development, testing

## 📖 Usage Examples

### Basic Request/Response Logging

```javascript
// Log all requests
interceptor.addRequestMiddleware(async (requestData) => {
  console.log(`📤 ${requestData.method} ${requestData.url}`, {
    headers: requestData.headers,
    body: requestData.body,
  });
}, "request_logger");

// Log all responses
interceptor.addResponseMiddleware(async (response, requestData) => {
  console.log(`📥 ${response.status} ${requestData.url}`, {
    headers: response.headers,
    body: response.body,
  });
}, "response_logger");
```

### Request Modification

```javascript
// Add authentication headers
interceptor.addRequestMiddleware(async (requestData) => {
  if (requestData.url.includes("/api/")) {
    requestData.headers["Authorization"] = "Bearer " + getAuthToken();
  }
}, "auth_injector");

// Add CORS headers
interceptor.addRequestMiddleware(async (requestData) => {
  requestData.headers["X-Requested-With"] = "XMLHttpRequest";
  requestData.headers["X-Custom-Header"] = "ReclamInterceptor";
}, "cors_headers");
```

### Response Analysis

```javascript
// Monitor API performance
interceptor.addResponseMiddleware(async (response, requestData) => {
  const duration = Date.now() - requestData.timestamp;

  if (duration > 5000) {
    console.warn(
      `🐌 Slow request detected: ${requestData.url} (${duration}ms)`
    );
  }

  // Track API errors
  if (response.status >= 400) {
    console.error(`❌ API Error: ${response.status} ${requestData.url}`, {
      response: response.body,
      request: requestData,
    });
  }
}, "performance_monitor");
```

### Data Collection

```javascript
// Collect analytics data
interceptor.addResponseMiddleware(async (response, requestData) => {
  const analyticsData = {
    url: requestData.url,
    method: requestData.method,
    status: response.status,
    duration: Date.now() - requestData.timestamp,
    size: response.headers["content-length"] || 0,
    timestamp: requestData.timestamp,
  };

  // Send to analytics service
  sendToAnalytics(analyticsData);
}, "analytics_collector");
```

### Request Filtering

```javascript
// Block specific domains
interceptor.addRequestMiddleware(async (requestData) => {
  const blockedDomains = ["malicious-site.com", "tracker.ads"];
  const url = new URL(requestData.url);

  if (blockedDomains.includes(url.hostname)) {
    console.log(`🚫 Blocked request to: ${requestData.url}`);
    // Note: Actual blocking would require additional implementation
  }
}, "domain_blocker");
```

## 🏗️ Architecture

### Request Data Structure

```javascript
{
  id: "req_1640123456789_abc123",     // Unique request identifier
  url: "https://api.example.com/data", // Request URL
  method: "POST",                      // HTTP method
  headers: {                          // Request headers object
    "Content-Type": "application/json",
    "Authorization": "Bearer token123"
  },
  body: "request body data",          // Request body (if any)
  request: Request,                   // Original Request object
  timestamp: 1640123456789            // Request timestamp
}
```

### Response Data Structure

```javascript
{
  id: "req_1640123456789_abc123",     // Matching request ID
  url: "https://api.example.com/data", // Response URL
  status: 200,                        // HTTP status code
  statusText: "OK",                   // HTTP status text
  headers: {                          // Response headers object
    "content-type": "application/json",
    "cache-control": "no-cache"
  },
  body: { data: "parsed response" },  // Parsed response body
  isMockedResponse: false,            // Always false for real responses
  originalResponse: Response,         // Original Response object
  timestamp: 1640123456790            // Response timestamp
}
```

### Middleware Lifecycle

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   HTTP Request  │───▶│  Request         │───▶│   Actual        │
│   (fetch/XHR)   │    │  Middlewares     │    │   Network       │
└─────────────────┘    └──────────────────┘    │   Request       │
                                               └─────────────────┘
                                                        │
┌─────────────────┐    ┌──────────────────┐           ▼
│   Return to     │◀───│  Response        │    ┌─────────────────┐
│   Caller        │    │  Middlewares     │◀───│   HTTP Response │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 🔧 API Reference

### Constructor

```javascript
new RequestInterceptor(options);
```

**Options:**

- `disableFetch` (boolean): Disable fetch interception
- `disableXHR` (boolean): Disable XHR interception
- `useProxyForFetch` (boolean): Use Proxy-based fetch interception
- `useGetterForFetch` (boolean): Use getter/setter approach

### Methods

#### `addRequestMiddleware(middleware, id?)`

Add a middleware function to process requests.

```javascript
interceptor.addRequestMiddleware(async (requestData) => {
  // Process request
}, "optional_id");
```

#### `addResponseMiddleware(middleware, id?)`

Add a middleware function to process responses.

```javascript
interceptor.addResponseMiddleware(async (response, requestData) => {
  // Process response
}, "optional_id");
```

#### `removeRequestMiddleware(id)`

Remove a request middleware by ID.

```javascript
interceptor.removeRequestMiddleware("middleware_id");
```

#### `removeResponseMiddleware(id)`

Remove a response middleware by ID.

```javascript
interceptor.removeResponseMiddleware("middleware_id");
```

#### `getMiddlewareIds()`

Get all registered middleware IDs.

```javascript
const ids = interceptor.getMiddlewareIds();
// Returns: { request: [...], response: [...] }
```

#### `updateOptions(options)`

Update configuration at runtime.

```javascript
interceptor.updateOptions({
  useGetterForFetch: true,
});
```

#### `cleanup()`

Clean up and restore original browser APIs.

```javascript
interceptor.cleanup();
```

## 🛠️ Advanced Usage

### Dynamic Middleware Management

```javascript
// Add conditional middleware
const authMiddleware = async (requestData) => {
  if (needsAuth(requestData.url)) {
    requestData.headers["Authorization"] = await getToken();
  }
};

// Add middleware
const middlewareId = interceptor.addRequestMiddleware(authMiddleware);

// Later, remove it
interceptor.removeRequestMiddleware(middlewareId);
```

### Error Handling

```javascript
interceptor.addResponseMiddleware(async (response, requestData) => {
  try {
    if (response.status === 401) {
      // Handle unauthorized
      await refreshToken();
      // Optionally retry request
    }
  } catch (error) {
    console.error("Middleware error:", error);
  }
}, "error_handler");
```

## 🚨 Troubleshooting

### Common Issues

#### "Body stream already read" Error

**Cause**: Response body consumed multiple times  
**Solution**: The interceptor now handles this automatically with proper cloning

#### Middleware Not Executing

**Cause**: JavaScript errors in middleware functions  
**Solution**: Check browser console for errors, add try/catch blocks

#### Interceptor Not Working

**Cause**: Script loading after page requests  
**Solution**: Ensure script loads before any HTTP requests

#### Override Conflicts

**Cause**: Other scripts overriding fetch/XHR  
**Solution**: Use getter/setter method (`useGetterForFetch: true`)

### Debug Mode

```javascript
// Enable detailed logging
const debug = {
  log: (...args) => console.log("🔍 [Debug]:", ...args),
  error: (...args) => console.error("❌ [Error]:", ...args),
  info: (...args) => console.info("ℹ️ [Info]:", ...args),
};
```

## 🔒 Security Considerations

- **Cross-Origin Requests**: Be aware of CORS implications when modifying headers
- **Sensitive Data**: Avoid logging sensitive information like passwords or tokens
- **Performance Impact**: Heavy middleware can affect page performance
- **Content Security Policy**: May be restricted by strict CSP headers

## 📈 Performance Tips

1. **Keep Middleware Lightweight**: Avoid heavy computations in middleware
2. **Use Async/Await Properly**: Don't block request flow unnecessarily
3. **Filter Early**: Add URL/method checks at the start of middleware
4. **Batch Operations**: Group related middleware operations
5. **Clean Up**: Remove unused middleware to reduce overhead

## 🤝 Contributing

### Development Setup

```bash
git clone https://github.com/your-username/reclaim-interceptor.git
cd reclaim-interceptor
```

### Code Style

- Use ES6+ features
- Follow async/await patterns
- Add JSDoc comments for new functions
- Maintain backward compatibility

### Testing

- Test across different browsers
- Verify with various websites
- Check performance impact
- Test middleware combinations

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Inspired by [Mock Service Worker (MSW)](https://mswjs.io/)
- Built for the [Reclaim Protocol](https://reclaimprotocol.org/) ecosystem
- Community feedback and contributions

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/your-username/reclaim-interceptor/issues)
- **Documentation**: This README and inline code comments

---

**Version**: 2025-07-11  
**Author**: Abdul Rashid Reshamwala
