**1. What is Websocket?**

A WebSocket is a communication protocol that provides full-duplex (two-way) communication over a single TCP connection between a client (such as a web browser) and a server. Unlike traditional HTTP, where communication is initiated by the client and the server responds, WebSocket allows both the client and server to send messages to each other at any time.

**2. Key features of WebSocket:**
- Full-Duplex: Communication can happen in both directions simultaneously.
- Persistent Connection: Once a WebSocket connection is established, it remains open, allowing for real-time data transfer without the need for repeated HTTP requests.
- Low Latency: Due to its persistent nature, WebSocket reduces the latency associated with establishing multiple connections, making it ideal for real-time applications like chat apps, online games, or live data feeds.

**3. Use cases:**
- Real-time chat applications.
- Live data streams (like stock market updates or multiplayer games).
- IoT (Internet of Things) communication.
- Collaborative editing (like Google Docs).

**4. Using Poco:**

A WebSocket connection typically starts as an HTTP handshake and then upgrades to the WebSocket protocol, allowing for real-time communication.

Poco C++ libraries provide WebSocket support through the Poco::Net::WebSocket class. You can use it to implement both WebSocket clients and servers. Here’s a basic overview of how you can use WebSockets with Poco:
- Setting Up a WebSocket Server with Poco
- Create an HTTP server: The WebSocket connection starts with an HTTP handshake, so you need an HTTP server that can handle WebSocket requests.
- Upgrade to WebSocket: During the HTTP handshake, if the request is for a WebSocket, you upgrade the connection.

```c++
#include <Poco/Net/HTTPServer.h>
#include <Poco/Net/HTTPRequestHandler.h>
#include <Poco/Net/HTTPRequestHandlerFactory.h>
#include <Poco/Net/HTTPServerRequest.h>
#include <Poco/Net/HTTPServerResponse.h>
#include <Poco/Net/ServerSocket.h>
#include <Poco/Net/WebSocket.h>
#include <Poco/Net/HTTPServerParams.h>
#include <Poco/Exception.h>
#include <iostream>

class WebSocketRequestHandler : public Poco::Net::HTTPRequestHandler
{
public:
    void handleRequest(Poco::Net::HTTPServerRequest& request, Poco::Net::HTTPServerResponse& response) override
    {
        try
        {
            Poco::Net::WebSocket ws(request, response);  // Upgrade the connection to WebSocket
            std::cout << "WebSocket connection established" << std::endl;

            char buffer[1024];
            int flags;
            int n;
            do
            {
                n = ws.receiveFrame(buffer, sizeof(buffer), flags);
                std::cout << "Received: " << std::string(buffer, n) << std::endl;
                ws.sendFrame(buffer, n, flags);  // Echo the received message
            } while (n > 0 && (flags & Poco::Net::WebSocket::FRAME_OP_BITMASK) != Poco::Net::WebSocket::FRAME_OP_CLOSE);
        }
        catch (Poco::Exception& exc)
        {
            std::cerr << "WebSocket Error: " << exc.displayText() << std::endl;
        }
    }
};

class WebSocketRequestHandlerFactory : public Poco::Net::HTTPRequestHandlerFactory
{
public:
    Poco::Net::HTTPRequestHandler* createRequestHandler(const Poco::Net::HTTPServerRequest& request) override
    {
        if (request.getURI() == "/ws")
        {
            return new WebSocketRequestHandler;  // Handle WebSocket requests at /ws
        }
        else
        {
            return nullptr;
        }
    }
};

int main(int argc, char** argv)
{
    try
    {
        Poco::Net::ServerSocket serverSocket(8080);  // Listening on port 8080
        Poco::Net::HTTPServer server(new WebSocketRequestHandlerFactory(), serverSocket, new Poco::Net::HTTPServerParams);
        server.start();

        std::cout << "WebSocket Server is running on port 8080" << std::endl;

        std::cin.get();  // Keep the server running
        server.stop();
    }
    catch (Poco::Exception& exc)
    {
        std::cerr << "Server error: " << exc.displayText() << std::endl;
    }

    return 0;
}
```

**Setting Up a WebSocket Client with Poco**

To create a WebSocket client in Poco, you can use the Poco::Net::WebSocket class to connect to a WebSocket server

```c++
#include <Poco/Net/HTTPClientSession.h>
#include <Poco/Net/HTTPRequest.h>
#include <Poco/Net/HTTPResponse.h>
#include <Poco/Net/WebSocket.h>
#include <iostream>

int main()
{
    try
    {
        Poco::Net::HTTPClientSession session("localhost", 8080);  // Connect to the server
        Poco::Net::HTTPRequest request(Poco::Net::HTTPRequest::HTTP_GET, "/ws", Poco::Net::HTTPRequest::HTTP_1_1);
        Poco::Net::HTTPResponse response;

        // Perform the WebSocket handshake
        Poco::Net::WebSocket ws(session, request, response);

        std::string msg = "Hello, WebSocket server!";
        ws.sendFrame(msg.data(), msg.size(), Poco::Net::WebSocket::FRAME_TEXT);

        char buffer[1024];
        int flags;
        int n = ws.receiveFrame(buffer, sizeof(buffer), flags);
        std::cout << "Received: " << std::string(buffer, n) << std::endl;

        ws.close();
    }
    catch (Poco::Exception& exc)
    {
        std::cerr << "WebSocket Error: " << exc.displayText() << std::endl;
    }

    return 0;
}
```

**5. Different between WebSocket and Regular socket:**

At a fundamental level, WebSockets do have similarities with traditional sockets (e.g., TCP/IP), particularly in how they handle low-level communication with methods like sendFrame and receiveFrame. Both involve sending and receiving messages over a persistent connection.
However, there are significant differences and enhancements with WebSockets over traditional sockets that make them particularly well-suited for modern web applications:

- Protocol Overhead:
    - WebSocket: Starts with an HTTP handshake and upgrades the connection for persistent communication. It uses a structured frame format.
    - Regular Socket: Uses a raw TCP connection without HTTP, and application-layer protocols must be manually defined.
- Full-Duplex Communication:
    - WebSocket: Supports full-duplex communication within a web-friendly framework, allowing simultaneous bidirectional communication.
    - Regular Socket: Also supports full-duplex, but requires manual management of communication protocols.
- Message Framing:
    - WebSocket: Automatically handles message framing (text, binary, control frames).
    - Regular Socket: Sends/receives raw data streams; developers handle message framing.
- Control Frames:
    - WebSocket: Includes built-in control frames for connection management (ping/pong, close).
    - Regular Socket: Lacks control frames; heartbeat or connection handling must be custom-built.
- Web Browser Support:

    - WebSocket: Natively supported by browsers, enabling real-time web communication.
    - Regular Socket: Not supported by browsers, requiring additional technologies for web-based communication.
- Security:

    - WebSocket: Can operate over secure channels (WSS), utilizing HTTP/HTTPS security.
    - Regular Socket: Requires separate implementation for secure connections (e.g., using SSL/TLS).
- High-Level API:

    - WebSocket: Provides higher-level abstractions (send/receive frames, ping/pong, etc.).
    - Regular Socket: Offers a lower-level API focused on sending/receiving raw data, leaving message handling to the developer.
- Client-Server Context:

    - WebSocket: Primarily used in client-server models, initiated by browsers using HTTP upgrade.
    - Regular Socket: More general-purpose, used in various communication contexts like peer-to-peer, but without automatic integration with web technologies.

 **6. Differences between HTTP and WebSocket Connection**
 
6.1:
- WebSocket is a bidirectional communication protocol that can send the data from the client to the server or from the server to the client by reusing the established connection channel. The connection is kept alive until terminated by either the client or the server.
- The HTTP protocol is a unidirectional protocol that works on top of TCP protocol which is a connection-oriented transport layer protocol, we can create the connection by using HTTP request methods after getting the response HTTP connection get closed.

6.2:
- Almost all the real-time applications like (trading, monitoring, notification) services use WebSocket to receive the data on a single communication channel.
- Simple RESTful application uses HTTP protocol which is stateless.

6.3:
- All the frequently updated applications used WebSocket because it is faster than HTTP Connection.
- It is used when we do not want to retain a connection for a particular amount of time or reuse the connection for transmitting data; An HTTP connection is slower than WebSockets.
