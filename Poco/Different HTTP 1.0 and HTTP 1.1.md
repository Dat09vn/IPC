Differences Between HTTP/1.0 and HTTP/1.1
HTTP/1.0 and HTTP/1.1 are two versions of the Hypertext Transfer Protocol used for communication on the web. HTTP/1.1 introduced several enhancements and improvements over HTTP/1.0. Below are the key differences:

**1. Connection Management**
- HTTP/1.0: By default, each request-response cycle uses a new connection. **The server closes the connection after sending the response**, resulting in a "non-persistent" connection. If the client wants to request another resource, a new connection must be established.
- HTTP/1.1: Supports persistent connections by default. **The connection is kept open after the response is sent, and multiple requests can be sent over the same connection (this is called "keep-alive"). This reduces the overhead of establishing a new connection for each request, improving performance.**
- Example: In HTTP/1.0, the client has to send the Connection: keep-alive header to keep the connection open, but in HTTP/1.1, the connection stays open unless explicitly closed by Connection: close.

**2. Pipelining**
- HTTP/1.0: Does not support pipelining, meaning each request must be completed (response fully received) before the next request can be sent.
- HTTP/1.1: Supports HTTP pipelining, which allows the client to send multiple requests before receiving responses. This can reduce latency and improve efficiency, though it is not commonly used due to complexities in implementation.

**3. Host Header**
- HTTP/1.0: The Host header is not mandatory. As a result, HTTP/1.0 assumes that each IP address corresponds to a single server, which is not practical for modern web hosting (e.g., virtual hosting where multiple domains share a single IP).
- HTTP/1.1: The Host header is required. It allows multiple domains to be hosted on the same IP address (virtual hosting), enabling the server to distinguish which website the request is for.

**4. Caching**

- HTTP/1.0: Caching support is limited and handled via the Expires header.
- HTTP/1.1: Introduces more sophisticated caching mechanisms with new headers such as Cache-Control, If-Modified-Since, If-Unmodified-Since, ETag, and others. These headers provide finer control over cache validation and expiration.
- Example:
  - Cache-Control: max-age=3600 specifies that the response can be cached for 1 hour.
  - ETag helps in validating whether the cached version of the resource has changed.

**5. Chunked Transfer Encoding**
- HTTP/1.0: Does not support chunked transfer encoding, meaning the server must know the full size of the response before sending it. If the size is unknown, it cannot start streaming the response.
- HTTP/1.1: Supports chunked transfer encoding, allowing the server to send the response in chunks without needing to know the content length in advance. This is especially useful for dynamically generated content or when the content is being streamed.

**6. Error Reporting**
- HTTP/1.0: Limited status codes (e.g., 200 OK, 404 Not Found, 500 Internal Server Error).
- HTTP/1.1: Introduces new status codes and enhances error reporting. For example, 100 Continue, 409 Conflict, 414 URI Too Long, and 503 Service Unavailable are added in HTTP/1.1 for more specific error handling.

**7. OPTIONS and New Methods**
- HTTP/1.0: Supports only basic methods like GET, POST, and HEAD.
- HTTP/1.1: Adds several new methods:
  - OPTIONS: Used to query the server for supported HTTP methods.
   -PUT: For uploading resources to the server.
  - DELETE: For removing resources.
  - TRACE: For debugging purposes (echoes back the request).
  - PATCH: For partial updates of resources.

**8. Request Size Limits**
- HTTP/1.0: Does not explicitly define size limits for URIs, headers, or request bodies.
- HTTP/1.1: Introduces more formalized limits for better interoperability between clients and servers, though actual limits depend on the server configuration.

**9. Additional Headers and Features**
- Content-Encoding and Transfer-Encoding: HTTP/1.1 provides more robust support for data compression (e.g., gzip) via the Content-Encoding header.
- Byte Range Requests: HTTP/1.1 allows clients to request partial content (byte range requests), enabling more efficient handling of large files, particularly for streaming or resuming downloads.

**10. Upgrade Header**
**- HTTP/1.1: Introduces the Upgrade header, allowing the client and server to upgrade to a different protocol, such as switching from HTTP to WebSocket.**
