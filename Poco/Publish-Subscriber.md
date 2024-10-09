**1 . Overview:**

To implement a basic publish-subscribe (pub-sub) simulation in C++, you can use the Observer design pattern where:
  - Publishers (Subjects) notify Subscribers (Observers) when an event occurs.
  - Subscribers register to be notified by the publisher.

**2. Basic Structure:**
  1. Subject (Publisher):
  - Manages a list of observers and notifies them of any state changes.
  2. Observer (Subscriber):
  - Defines an interface to be implemented by any class that wants to observe (subscribe) to the subject.

Here's an example implementation:
```c++
#include <iostream>
#include <vector>
#include <string>
 
// Observer interface
class Observer {
public:
    virtual void update(const std::string &message) = 0;
};
 
// Subject class
class Subject {
    std::vector<Observer*> observers;
     
public:
    void subscribe(Observer* observer) {
        observers.push_back(observer);
    }
 
    void unsubscribe(Observer* observer) {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }
 
    void notify(const std::string &message) {
        for (Observer* observer : observers) {
            observer->update(message);
        }
    }
};
 
// Concrete observer (Subscriber)
class ConcreteObserver : public Observer {
    std::string name;
public:
    ConcreteObserver(const std::string &name) : name(name) {}
 
    void update(const std::string &message) override {
        std::cout << "Observer " << name << " received message: " << message << std::endl;
    }
};
 
// Concrete subject (Publisher)
class ConcreteSubject : public Subject {
public:
    void createEvent(const std::string &message) {
        std::cout << "Subject created event: " << message << std::endl;
        notify(message);
    }
};
 
// Main function to simulate pub-sub
int main() {
    ConcreteSubject subject;
 
    ConcreteObserver observer1("Observer1");
    ConcreteObserver observer2("Observer2");
 
    subject.subscribe(&observer1);
    subject.subscribe(&observer2);
 
    subject.createEvent("Event 1");
 
    subject.unsubscribe(&observer2);
 
    subject.createEvent("Event 2");
 
    return 0;
}
```

**Scenario: Logging system using publish-subscribe pattern**

Imagine we have a system where different components (like the console, a file, or a monitoring system) want to log messages. These components subscribe to a Logger (the publisher), which will notify them every time a new log message is available.
```c++
#include <iostream>
#include <vector>
#include <fstream>
#include <string>
#include <algorithm>

// Observer interface (Subscriber)
class LogObserver {
public:
    virtual void onLogMessage(const std::string& message) = 0;
    virtual ~LogObserver() = default;
};

// Concrete Observer: Log to Console
class ConsoleLogger : public LogObserver {
public:
    void onLogMessage(const std::string& message) override {
        std::cout << "[Console] " << message << std::endl;
    }
};

// Concrete Observer: Log to File
class FileLogger : public LogObserver {
    std::ofstream logFile;
public:
    FileLogger(const std::string& fileName) {
        logFile.open(fileName, std::ios::app);
    }

    ~FileLogger() {
        if (logFile.is_open()) {
            logFile.close();
        }
    }

    void onLogMessage(const std::string& message) override {
        if (logFile.is_open()) {
            logFile << "[File] " << message << std::endl;
        }
    }
};

// Subject class (Publisher)
class Logger {
    std::vector<LogObserver*> observers;

public:
    void subscribe(LogObserver* observer) {
        observers.push_back(observer);
    }

    void unsubscribe(LogObserver* observer) {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

    // Publish a log message to all observers
    void logMessage(const std::string& message) {
        std::cout << "[Logger] New message: " << message << std::endl;
        for (LogObserver* observer : observers) {
            observer->onLogMessage(message);
        }
    }
};

// Main function demonstrating real-world usage
int main() {
    Logger logger;

    ConsoleLogger consoleLogger;
    FileLogger fileLogger("logs.txt");

    // Subscribe both console and file loggers to the logger
    logger.subscribe(&consoleLogger);
    logger.subscribe(&fileLogger);

    // Log some messages
    logger.logMessage("System starting up...");
    logger.logMessage("Connecting to database...");
    logger.logMessage("Error: Unable to connect to database!");

    // Unsubscribe file logger and send another log
    logger.unsubscribe(&fileLogger);
    logger.logMessage("File logger unsubscribed.");

    return 0;
}
```
**3. Applying the Observer Pattern to WebSockets using the Poco C++ Libraries involves:**

Creating a system where clients can subscribe to events from a server through WebSocket connections. This setup allows for real-time communication between the server and clients, enabling the server to notify all connected clients whenever an event occurs.

Example Workflow:

1. Server Start: The WebSocket server starts on ws://localhost:8080/websocket.
2. Client Connection: Clients can connect to the server using a WebSocket client.
3. Message Handling: Clients can send messages, and the server will notify all connected clients about the event.
4. Real-Time Notifications: Clients receive updates in real time via the WebSocket connection.

Benefits of Using WebSockets with the Observer Pattern:
- Real-Time Communication: WebSockets allow for two-way communication between the server and clients, making it ideal for real-time applications.
- Scalability: Multiple clients can connect and receive notifications independently, improving the scalability of the application.
- Loose Coupling: The server and clients are decoupled, following the Observer Pattern, which enhances modularity.

Server code:
```c++
#include <Poco/Net/HTTPServer.h>
#include <Poco/Net/HTTPRequestHandler.h>
#include <Poco/Net/HTTPRequestHandlerFactory.h>
#include <Poco/Net/HTTPServerParams.h>
#include <Poco/Net/ServerSocket.h>
#include <Poco/Net/WebSocket.h>
#include <Poco/ThreadPool.h>
#include <Poco/Mutex.h>
#include <Poco/Net/HTTPServerRequest.h>   // Added to fix the incomplete type error
#include <thread>                         // Added to fix std::this_thread error
#include <chrono>                         // Added to fix sleep_for error
#include <vector>
#include <algorithm>
#include <iostream>
#include <mutex>
#include <memory>
 
// Observer Interface
class IObserver {
public:
    virtual void update(const std::string& eventData) = 0;
    virtual ~IObserver() = default;
};
 
// Subject Interface
class ISubject {
public:
    virtual void attach(std::shared_ptr<IObserver> observer) = 0;
    virtual void detach(std::shared_ptr<IObserver> observer) = 0;
    virtual void notify(const std::string& eventData) = 0;
    virtual ~ISubject() = default;
};
 
// Publisher (WebSocketServer)
class WebSocketPublisher : public ISubject {
private:
    std::vector<std::shared_ptr<IObserver>> observers;
    std::mutex mtx;  // Mutex for thread safety
 
public:
    void attach(std::shared_ptr<IObserver> observer) override {
        std::lock_guard<std::mutex> lock(mtx);
        observers.push_back(observer);
    }
 
    void detach(std::shared_ptr<IObserver> observer) override {
        std::lock_guard<std::mutex> lock(mtx);
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }
 
    void notify(const std::string& eventData) override {
        std::lock_guard<std::mutex> lock(mtx);
        for (auto& observer : observers) {
            observer->update(eventData);
        }
    }
 
    void publishEvent(const std::string& eventMessage) {
        std::cout << "[Publisher] Publishing event: " << eventMessage << std::endl;
        notify(eventMessage);
    }
};
 
// WebSocket Client (Observer)
class WebSocketClient : public IObserver {
private:
    Poco::Net::WebSocket& _webSocket;
 
public:
    explicit WebSocketClient(Poco::Net::WebSocket& webSocket) : _webSocket(webSocket) {}
 
    // Update method for the observer pattern
    void update(const std::string& eventData) override {
        try {
            _webSocket.sendFrame(eventData.data(), eventData.size(), Poco::Net::WebSocket::FRAME_TEXT);
        } catch (Poco::Exception& exc) {
            std::cerr << "[WebSocketClient] Error sending frame: " << exc.displayText() << std::endl;
        }
    }
};
 
// WebSocket Request Handler
class WebSocketRequestHandler : public Poco::Net::HTTPRequestHandler {
private:
    WebSocketPublisher& _publisher;
 
public:
    explicit WebSocketRequestHandler(WebSocketPublisher& publisher) : _publisher(publisher) {}
 
    void handleRequest(Poco::Net::HTTPServerRequest& request, Poco::Net::HTTPServerResponse& response) override {
        Poco::Net::WebSocket ws(request, response);
        std::cout << "[WebSocketServer] New client connected" << std::endl;
 
        // Create a new WebSocketClient and attach it to the publisher
        auto client = std::make_shared<WebSocketClient>(ws);
        _publisher.attach(client);
 
        // Handle client communication
        try {
            char buffer[1024];
            int flags;
            int n;
            do {
                n = ws.receiveFrame(buffer, sizeof(buffer), flags);
                if (n > 0) {
                    std::string message(buffer, n);
                    std::cout << "[WebSocketServer] Received message: " << message << std::endl;
 
                    // Publish the received message to all clients
                    _publisher.publishEvent("Echo: " + message);
                }
            } while (n > 0 && (flags & Poco::Net::WebSocket::FRAME_OP_BITMASK) != Poco::Net::WebSocket::FRAME_OP_CLOSE);
        } catch (Poco::Exception& exc) {
            std::cerr << "[WebSocketServer] WebSocket Error: " << exc.displayText() << std::endl;
        }
 
        // Detach client after the connection is closed
        _publisher.detach(client);
        std::cout << "[WebSocketServer] Client disconnected" << std::endl;
    }
};
 
// WebSocket Request Handler Factory
class WebSocketRequestHandlerFactory : public Poco::Net::HTTPRequestHandlerFactory {
private:
    WebSocketPublisher& _publisher;
 
public:
    explicit WebSocketRequestHandlerFactory(WebSocketPublisher& publisher) : _publisher(publisher) {}
 
    Poco::Net::HTTPRequestHandler* createRequestHandler(const Poco::Net::HTTPServerRequest& request) override {
        if (request.getURI() == "/websocket") {
            return new WebSocketRequestHandler(_publisher);
        }
        return nullptr;
    }
};
 
// Main Server
int main() {
    try {
        Poco::Net::ServerSocket socket(8080);  // WebSocket server listens on port 8080
        Poco::Net::HTTPServerParams* params = new Poco::Net::HTTPServerParams;
        params->setMaxQueued(100);
        params->setMaxThreads(4);
 
        // WebSocket Publisher (Subject)
        WebSocketPublisher publisher;
 
        // Create the WebSocket server
        Poco::Net::HTTPServer server(new WebSocketRequestHandlerFactory(publisher), socket, params);
        server.start();
        std::cout << "[WebSocketServer] WebSocket Server started on port 8080" << std::endl;
 
        // Simulate event publishing every few seconds
        int count = 0;
        while (true) {
            std::this_thread::sleep_for(std::chrono::seconds(5));  // Sleep for 5 seconds
            publisher.publishEvent("Event #" + std::to_string(++count));  // Broadcast event to all clients
        }
 
        server.stop();
    } catch (Poco::Exception& e) {
        std::cerr << "Server error: " << e.displayText() << std::endl;
    }
 
    return 0;
}
```

Client code:
```c++
#include <iostream>
#include <Poco/Net/HTTPClientSession.h>
#include <Poco/Net/HTTPRequest.h>
#include <Poco/Net/HTTPResponse.h>
#include <Poco/Net/WebSocket.h>
#include <Poco/Exception.h>
#include <string>
 
int main() {
    try {
        // Create an HTTP session for connecting to the server
        Poco::Net::HTTPClientSession session("localhost", 8080);
         
        // Prepare the HTTP request to upgrade to WebSocket
        Poco::Net::HTTPRequest request(Poco::Net::HTTPRequest::HTTP_GET, "/websocket", Poco::Net::HTTPRequest::HTTP_1_1);
        Poco::Net::HTTPResponse response;
 
        // Perform the WebSocket handshake
        Poco::Net::WebSocket ws(session, request, response);
 
        std::cout << "[Client] Connected to WebSocket server" << std::endl;
 
        // Prepare the message to send
        std::string message = "Hello, WebSocket Server!";
        ws.sendFrame(message.data(), message.size(), Poco::Net::WebSocket::FRAME_TEXT);
 
        std::cout << "[Client] Sent message: " << message << std::endl;
 
        // Receive the server's response
        char buffer[1024];
        int flags;
        int n = ws.receiveFrame(buffer, sizeof(buffer), flags);
 
        if (n > 0) {
            std::string responseMessage(buffer, n);
            std::cout << "[Client] Received message: " << responseMessage << std::endl;
        }
 
        // Close the WebSocket connection
        ws.close();
        std::cout << "[Client] WebSocket connection closed" << std::endl;
    } catch (Poco::Exception& exc) {
        std::cerr << "[Client] WebSocket error: " << exc.displayText() << std::endl;
        return 1;
    }
 
    return 0;
}
```
Also, To test the WebSocket server with a client, you can use client code or command line : websocat ws://localhost:8080/websocket

How It Works:
- Multiple WebSocket clients can connect to the server at the /websocket endpoint.
- Each client is treated as an observer. When the server generates an event (in this case, every 5 seconds), it notifies all connected clients by sending the event data over the WebSocket connection.
- If a client disconnects, it is removed from the list of observers.
