In the context of web development, an HTTP method refers to a specific action that a client (usually a web browser or app) wants to perform on a resource, such as retrieving data, submitting a form, or updating information on a server. The most common HTTP methods are:

**1. GET:** Requests data from a server. It is used to retrieve information, and it should not alter the state of the server (i.e., it's a "safe" method). Data is sent in the URL.

Example: GET /api/users to retrieve a list of users.

**2. POST:** Sends data to the server to create a new resource. It typically changes the server's state.

Example: POST /api/users with a JSON body to create a new user.

**3. PUT:** Updates an existing resource on the server with the provided data. If the resource does not exist, it can create a new one (though some implementations only allow updates).

Example: PUT /api/users/123 with a JSON body to update user 123's information.

**4. DELETE:** Removes a resource from the server.

Example: DELETE /api/users/123 to delete the user with ID 123.

**5. PATCH:** Partially updates an existing resource. Unlike PUT, which usually requires the full representation of a resource, PATCH only sends the fields that need updating.

Example: PATCH /api/users/123 to update just the email of user 123.

**6. HEAD:** Similar to GET, but it only retrieves the headers (metadata) and not the actual content of the resource.

Example: HEAD /api/users to check if the resource is available without downloading the data.

**7. OPTIONS:** Describes the communication options available for a resource or the server. It can be used to check which methods are supported by the server.

Example: OPTIONS /api/users to find out what HTTP methods are allowed for /api/users.

These methods are the building blocks of RESTful APIs, and different use cases apply depending on what you need to accomplish with the server.

Server code:
```c++
#include <Poco/Net/HTTPServer.h>
#include <Poco/Net/HTTPRequestHandler.h>
#include <Poco/Net/HTTPRequestHandlerFactory.h>
#include <Poco/Net/HTTPServerRequest.h>
#include <Poco/Net/HTTPServerResponse.h>
#include <Poco/Net/ServerSocket.h>
#include <Poco/Util/ServerApplication.h>
#include <Poco/JSON/Parser.h>
#include <Poco/JSON/Object.h>
#include <Poco/JSON/Stringifier.h>
#include <iostream>
#include <map>
 
// User structure
struct User {
    std::string id;
    std::string name;
    std::string email;
};
 
// In-memory user storage
std::map<std::string, User> users;
 
// Custom request handler for User operations
class UserRequestHandler : public Poco::Net::HTTPRequestHandler {
public:
    void handleRequest(Poco::Net::HTTPServerRequest& request, Poco::Net::HTTPServerResponse& response) override {
        response.setContentType("application/json");
        std::ostream& out = response.send();
        std::string userId = request.getURI().substr(1);  // Extract user ID from the URI
 
        if (request.getMethod() == Poco::Net::HTTPRequest::HTTP_GET) {
            if (users.find(userId) != users.end()) {
                // Retrieve user
                Poco::JSON::Object jsonResponse;
                jsonResponse.set("id", users[userId].id);
                jsonResponse.set("name", users[userId].name);
                jsonResponse.set("email", users[userId].email);
                Poco::JSON::Stringifier::stringify(jsonResponse, out);
            } else {
                response.setStatus(Poco::Net::HTTPResponse::HTTP_NOT_FOUND);
                out << "{\"error\": \"User not found\"}";
            }
        } else if (request.getMethod() == Poco::Net::HTTPRequest::HTTP_POST) {
            // Create a new user
            Poco::JSON::Parser parser;
            auto json = parser.parse(request.stream()).extract<Poco::JSON::Object::Ptr>();
            User user{json->getValue<std::string>("id"), json->getValue<std::string>("name"), json->getValue<std::string>("email")};
            users[user.id] = user;
            out << "{\"message\": \"User created\"}";
        } else if (request.getMethod() == Poco::Net::HTTPRequest::HTTP_PUT) {
            // Update existing user
            if (users.find(userId) != users.end()) {
                Poco::JSON::Parser parser;
                auto json = parser.parse(request.stream()).extract<Poco::JSON::Object::Ptr>();
                users[userId].name = json->getValue<std::string>("name");
                users[userId].email = json->getValue<std::string>("email");
                out << "{\"message\": \"User updated\"}";
            } else {
                response.setStatus(Poco::Net::HTTPResponse::HTTP_NOT_FOUND);
                out << "{\"error\": \"User not found\"}";
            }
        } else if (request.getMethod() == Poco::Net::HTTPRequest::HTTP_DELETE) {
            // Delete user
            if (users.find(userId) != users.end()) {
                users.erase(userId);
                out << "{\"message\": \"User deleted\"}";
            } else {
                response.setStatus(Poco::Net::HTTPResponse::HTTP_NOT_FOUND);
                out << "{\"error\": \"User not found\"}";
            }
        } else {
            response.setStatus(Poco::Net::HTTPResponse::HTTP_BAD_REQUEST);
            out << "{\"error\": \"Unsupported HTTP method\"}";
        }
        out.flush();
    }
};
 
// Factory to create request handlers
class UserRequestHandlerFactory : public Poco::Net::HTTPRequestHandlerFactory {
public:
    Poco::Net::HTTPRequestHandler* createRequestHandler(const Poco::Net::HTTPServerRequest&) override {
        return new UserRequestHandler;
    }
};
 
// Main application class
class UserHTTPServerApp : public Poco::Util::ServerApplication {
protected:
    int main(const std::vector<std::string>&) override {
        Poco::Net::ServerSocket socket(8080);
        Poco::Net::HTTPServerParams::Ptr params = new Poco::Net::HTTPServerParams;
        Poco::Net::HTTPServer server(new UserRequestHandlerFactory, socket, params);
 
        server.start();
        std::cout << "HTTP Server started on port 8080..." << std::endl;
 
        waitForTerminationRequest();
        server.stop();
 
        std::cout << "HTTP Server stopped." << std::endl;
        return Application::EXIT_OK;
    }
};
 
int main(int argc, char** argv) {
    UserHTTPServerApp app;
    return app.run(argc, argv);
}
```
Client code:
```c++
#include <Poco/Net/HTTPClientSession.h>
#include <Poco/Net/HTTPRequest.h>
#include <Poco/Net/HTTPResponse.h>
#include <Poco/StreamCopier.h>
#include <Poco/JSON/Object.h>
#include <Poco/JSON/Stringifier.h>
#include <iostream>
#include <sstream>
 
// Function to send an HTTP GET request to retrieve a user
void getUser(const std::string& userId) {
    Poco::Net::HTTPClientSession session("localhost", 8080);
    Poco::Net::HTTPRequest request(Poco::Net::HTTPRequest::HTTP_GET, "/" + userId, Poco::Net::HTTPRequest::HTTP_1_1);
    Poco::Net::HTTPResponse response;
    session.sendRequest(request);
    std::istream& resStream = session.receiveResponse(response);
    std::stringstream ss;
    Poco::StreamCopier::copyStream(resStream, ss);
    std::cout << "GET Response: " << ss.str() << std::endl;
}
 
// Function to send an HTTP POST request to create a user
void createUser(const std::string& userId, const std::string& name, const std::string& email) {
    Poco::Net::HTTPClientSession session("localhost", 8080);
    Poco::Net::HTTPRequest request(Poco::Net::HTTPRequest::HTTP_POST, "/", Poco::Net::HTTPRequest::HTTP_1_1);
    request.setContentType("application/json");
 
    Poco::JSON::Object json;
    json.set("id", userId);
    json.set("name", name);
    json.set("email", email);
 
    std::ostringstream oss;
    Poco::JSON::Stringifier::stringify(json, oss);
    request.setContentLength(oss.str().length());
 
    std::ostream& os = session.sendRequest(request);
    os << oss.str();
 
    Poco::Net::HTTPResponse response;
    std::istream& resStream = session.receiveResponse(response);
    std::stringstream ss;
    Poco::StreamCopier::copyStream(resStream, ss);
    std::cout << "POST Response: " << ss.str() << std::endl;
}
 
// Function to send an HTTP PUT request to update a user
void updateUser(const std::string& userId, const std::string& name, const std::string& email) {
    Poco::Net::HTTPClientSession session("localhost", 8080);
    Poco::Net::HTTPRequest request(Poco::Net::HTTPRequest::HTTP_PUT, "/" + userId, Poco::Net::HTTPRequest::HTTP_1_1);
    request.setContentType("application/json");
 
    Poco::JSON::Object json;
    json.set("name", name);
    json.set("email", email);
 
    std::ostringstream oss;
    Poco::JSON::Stringifier::stringify(json, oss);
    request.setContentLength(oss.str().length());
 
    std::ostream& os = session.sendRequest(request);
    os << oss.str();
 
    Poco::Net::HTTPResponse response;
    std::istream& resStream = session.receiveResponse(response);
    std::stringstream ss;
    Poco::StreamCopier::copyStream(resStream, ss);
    std::cout << "PUT Response: " << ss.str() << std::endl;
}
 
// Function to send an HTTP DELETE request to delete a user
void deleteUser(const std::string& userId) {
    Poco::Net::HTTPClientSession session("localhost", 8080);
    Poco::Net::HTTPRequest request(Poco::Net::HTTPRequest::HTTP_DELETE, "/" + userId, Poco::Net::HTTPRequest::HTTP_1_1);
    Poco::Net::HTTPResponse response;
    session.sendRequest(request);
    std::istream& resStream = session.receiveResponse(response);
    std::stringstream ss;
    Poco::StreamCopier::copyStream(resStream, ss);
    std::cout << "DELETE Response: " << ss.str() << std::endl;
}
 
// Main function to test the client
int main() {
    // Create users
    createUser("1", "Alice", "alice@example.com");
    createUser("2", "Bob", "bob@example.com");
 
    // Retrieve users
    getUser("1");
    getUser("2");
 
    // Update a user
    updateUser("1", "Alice Smith", "alice.smith@example.com");
    getUser("1");  // Retrieve updated user
 
    // Delete a user
    deleteUser("2");
    getUser("2");  // Try to retrieve deleted user
 
    return 0;
}
```
