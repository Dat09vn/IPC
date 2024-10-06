**1. What is HTTP?**

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the World Wide Web. It defines how messages are formatted and transmitted, and how web servers and browsers should respond to various requests. It is an application-layer protocol used for transmitting hypermedia (like HTML) documents, enabling users to interact with websites.

**2. Key Characteristics of HTTP:**

- Client-Server Model:
  - HTTP operates on a client-server model. The client (usually a browser or an HTTP client like curl) sends requests, and the server responds with the requested resources (like web pages, images, or data).
- Stateless:
  - HTTP is stateless, meaning each request from the client to the server is independent, and the server does not retain any information (state) between requests. Each HTTP request must contain all necessary information for the server to understand and process it.
- Request-Response Protocol:
  - HTTP works as a request-response protocol. The client sends an HTTP request (e.g., GET, POST), and the server returns an HTTP response with a status code and the requested resource or data.
- HTTP Methods: HTTP defines several request methods, including:
  - GET: Requests data from the server. It's typically used to retrieve information.
  - POST: Sends data to the server. It's used when submitting forms or uploading files.
  - PUT: Updates or replaces data on the server.
  - DELETE: Deletes a resource on the server.
  - HEAD: Similar to GET, but only retrieves headers, not the actual content.
- HTTP Status Codes: Responses from the server include status codes to indicate the outcome of the request:
  - 200 OK: The request was successful, and the server returned the requested data.
  - 404 Not Found: The server cannot find the requested resource.
  - 500 Internal Server Error: The server encountered an error while processing the request.
- Headers:
  - Both requests and responses include headers that carry additional information (e.g., content type, caching policies, authentication tokens).
- Media Type:
  - HTTP can handle various types of media: HTML, text, images, videos, and other content. The server specifies the type of content it's returning using the Content-Type header.
- HTTP/1.1 vs. HTTP/2 vs. HTTP/3:
There are different versions of HTTP, each improving performance and efficiency:
  - HTTP/1.1: The most commonly used version. It introduced persistent connections, allowing multiple requests to be sent over a single connection.
  - HTTP/2: A more efficient version, offering features like multiplexing (sending multiple requests over the same connection), header compression, and server push (server sending resources to the client before they are requested).
  - HTTP/3: The latest version built on QUIC (a protocol based on UDP). It further improves speed and reliability, especially in environments with high packet loss.

**3. How HTTP Works:**

- Client Request:

The client (browser or HTTP client) initiates a request by sending an HTTP message to the server. For example:
```
GET /index.html HTTP/1.1
Host: www.example.com
```
The GET method requests the index.html file from the server.

- Server Response:

The server processes the request and sends back a response, typically in two parts: headers and the body (content):
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1024

<html>
<body>
<h1>Welcome to the website</h1>
</body>
</html>
```
The response includes the status code (200 OK means success) and the requested HTML page in the body.

**4. Use Cases of HTTP:**
- Web Browsing: When you visit a website, your browser makes HTTP requests to load the web pages and display them to you.
- API Communication: Many web-based services use HTTP to communicate between client applications and servers through RESTful APIs.
- File Transfer: HTTP is also used for downloading and uploading files between servers and clients.

**5. Using Poco lib**

Steps to implement HTTP Client-Sever:
- Server Setup:
  - Step 1: Create a request handler (MyRequestHandler) to handle HTTP requests (e.g., process POST requests).
  - Step 2: Implement a request handler factory (MyRequestHandlerFactory) to create appropriate handlers based on the request (e.g., handle POST requests only).
  - Step 3: Implement the HTTPServerApp class to set up and start the server:
  - Bind the server to a port (e.g., port 8080).
  - Use Poco::Net::HTTPServer to start the server and listen for incoming requests.
  - Wait for termination signals (like CTRL+C).
  - Compile and run the server.
- Client Setup:
  - Step 1: Use Poco::Net::HTTPClientSession to create a session and connect to the server (localhost:8080).
  - Step 2: Prepare an HTTP POST request (Poco::Net::HTTPRequest), set headers, and define the body content.
  - Step 3: Send the request using the session's sendRequest method and send the body data.
  - Step 4: Receive the server's response using receiveResponse and print the response status and body content.
  - Compile and run the client.

Server code:
```c++
#include "Poco/Net/HTTPClientSession.h"
#include "Poco/Net/HTTPRequest.h"
#include "Poco/Net/HTTPResponse.h"
#include "Poco/StreamCopier.h"
#include "Poco/URI.h"
#include <iostream>
#include <sstream>

using namespace Poco::Net;
using namespace Poco;

int main() {
    try {
        // Create a URI object for the request
        URI uri("http://localhost:8080/hello");
        std::string path = uri.getPathAndQuery();  // Path of the request (e.g., "/hello")
        if (path.empty()) path = "/";

        // Create a session with the server (localhost on port 8080)
        HTTPClientSession session(uri.getHost(), uri.getPort());

        // Prepare an HTTP GET request
        HTTPRequest req(HTTPRequest::HTTP_GET, path, HTTPMessage::HTTP_1_1);
        session.sendRequest(req);

        // Receive the response
        HTTPResponse res;
        std::istream &rs = session.receiveResponse(res);

        // Print the response status
        std::cout << "Response Status: " << res.getStatus() << " " << res.getReason() << std::endl;
        std::cout << "Get content type: " << res.getContentType() << std::endl;
        // Copy the response body to stdout
        StreamCopier::copyStream(rs, std::cout);
    } catch (const Exception &ex) {
        std::cerr << "Exception: " << ex.displayText() << std::endl;
    }

    return 0;
}
```

Client code:
```c++
#include "Poco/Net/HTTPServer.h"
#include "Poco/Net/HTTPRequestHandler.h"
#include "Poco/Net/HTTPRequestHandlerFactory.h"
#include "Poco/Net/HTTPServerRequest.h"
#include "Poco/Net/HTTPServerResponse.h"
#include "Poco/Net/ServerSocket.h"
#include "Poco/ThreadPool.h"
#include "Poco/Util/ServerApplication.h"
#include <iostream>
#include <sstream>

using namespace Poco::Net;
using namespace Poco::Util;
using namespace Poco;

// Handler for the `/hello` URI
class HelloRequestHandler : public HTTPRequestHandler {
public:
    void handleRequest(HTTPServerRequest &request, HTTPServerResponse &response) override {
        std::cout << "Request URI: " << request.getURI() << std::endl;
        std::cout << "Request Method: " << request.getMethod() << std::endl;
        response.setContentType("text/html");
        response.setStatusAndReason(HTTPResponse::HTTP_OK, "Request success");
        std::ostream &out = response.send();
        out << "<html><head><title>Hello</title></head>";
        out << "<body><h1>Hello from /hello</h1></body></html>";
        out.flush();
    }
};

// Handler for the `/goodbye` URI
class GoodbyeRequestHandler : public HTTPRequestHandler {
public:
    void handleRequest(HTTPServerRequest &request, HTTPServerResponse &response) override {
        std::cout << "Request URI: " << request.getURI() << std::endl;
        std::cout << "Request Method: " << request.getMethod() << std::endl;
        response.setContentType("text/html");
        response.setStatus(HTTPResponse::HTTP_OK);
        std::ostream &out = response.send();
        out << "<html><head><title>Goodbye</title></head>";
        out << "<body><h1>Goodbye from /goodbye</h1></body></html>";
        out.flush();
    }
};

// Handler for other URIs or fallback if no specific handler is matched
class DefaultRequestHandler : public HTTPRequestHandler {
public:
    void handleRequest(HTTPServerRequest &request, HTTPServerResponse &response) override {
        std::cout << "Request URI: " << request.getURI() << std::endl;
        std::cout << "Request Method: " << request.getMethod() << std::endl;
        response.setContentType("text/html");
        response.setStatus(HTTPResponse::HTTP_OK);
        std::ostream &out = response.send();
        out << "<html><head><title>Default</title></head>";
        out << "<body><h1>Default Handler: No specific URI match</h1></body></html>";
        out.flush();
    }
};

// Request handler factory that returns different handlers based on the URI
class MyRequestHandlerFactory : public HTTPRequestHandlerFactory {
public:
    HTTPRequestHandler *createRequestHandler(const HTTPServerRequest &request) override {
        const std::string &uri = request.getURI();
        if (uri == "/hello") {
            return new HelloRequestHandler;
        } else if (uri == "/goodbye") {
            return new GoodbyeRequestHandler;
        } else {
            return new DefaultRequestHandler;  // Default handler if no match
        }
    }
};

class MyHTTPServerApp : public ServerApplication {
protected:
    int main(const std::vector<std::string> &) override {
        // Create the server socket on port 8080
        ServerSocket svs(8080);

        // Create a custom thread pool
        ThreadPool threadPool(4, 16, 60);

        // Configure HTTP server parameters
        HTTPServerParams *params = new HTTPServerParams;
        params->setMaxQueued(100);   // Max queued requests
        params->setMaxThreads(16);   // Max concurrent threads

        // Create the HTTPServer with our custom factory
        HTTPServer server(new MyRequestHandlerFactory(), threadPool, svs, params);

        // Start the server
        server.start();
        std::cout << "Server started on port 8080" << std::endl;

        // Wait for CTRL-C or kill signal to stop the server
        waitForTerminationRequest();

        // Stop the server
        server.stop();

        return Application::EXIT_OK;
    }
};

int main(int argc, char **argv) {
    MyHTTPServerApp app;
    return app.run(argc, argv);
}
```


**TEST: curl -X GET http://localhost:8080/hello**

**EXPLAIN CODE:**

1. Poco::waitForTerminationRequest() is a utility function provided by the Poco::Util::ServerApplication class, used to make a server application wait for a termination request (such as a signal from the operating system). Typically, it waits for signals like SIGINT (triggered by Ctrl+C) or SIGTERM, which are commonly used to stop server processes.

2. HTTPServer::start() How It Works:
- Thread Pool Activation: The HTTPServer or TCPServer uses a thread pool to handle multiple client connections concurrently. Calling start() activates this thread pool, and worker threads start waiting for incoming requests.
- Listening on a Socket: The server listens on the specified port (e.g., 8080 in the example) for incoming connections. It uses a ServerSocket or a SecureServerSocket (if using SSL) to bind to the port.
- Handling Requests: When a new connection is received, a thread from the pool is assigned to handle the request. The HTTPRequestHandler (created by the factory) processes the request and sends back the appropriate response.
- Concurrency: The server can handle multiple connections at the same time because of the underlying thread pool. The pool size is either the default value or can be configured explicitly using HTTPServerParams.

3. HTTPServer::Stop()
- Stops Accepting New Connections: The server stops accepting new connections on the port it was bound to. Any new connection attempts will be refused.
- Completes Ongoing Requests: The server finishes processing any requests that are currently being handled. It does not abruptly terminate the existing connections; instead, it waits for the threads handling those requests to complete their work.
- Stops the Thread Pool: The thread pool used to manage client connections is stopped. After all active threads have finished processing, they are properly cleaned up.
- Closes the Server Socket: The server socket, which was bound to a specific port (e.g., port 8080), is closed, meaning the server is no longer listening on that port.
  
To handle a POST request using Poco::Net::HTTPServer, you'll need to implement a request handler that can process incoming POST data from the client. Typically, POST requests are used to send data to the server (like form submissions or JSON payloads).
