**1.Network programming basics:**

- The Poco::Net::IPAddress class stores an IPv4 or IPv6 host address.
- A Poco::Net::SocketAddress combines an IPAddress with a port number, thus identifying the endpoint of an IP network connection.
- The Poco::Net::DNS class provides an interface to the Domain Name System, mapping domain names to IP addresses and vice versa.
- Address information for a host is returned in the form of a Poco::Net::HostEntry object. A HostEntry contains a host's primary name, a list of aliases, and a list of IP addresses.

The program performs a **DNS lookup** for www.google.com:
```c++
#include "Poco/Net/DNS.h"
#include <iostream>
 
using Poco::Net::DNS;
using Poco::Net::IPAddress;
using Poco::Net::HostEntry;
 
int main(int argc, char** argv)
{
    // Perform DNS lookup for "www.google.com"
    const HostEntry& entry = DNS::hostByName("www.google.com");
     
    // Print the canonical name
    std::cout << "Canonical Name: " << entry.name() << std::endl;
     
    // Print aliases (if any)
    const HostEntry::AliasList& aliases = entry.aliases();
    HostEntry::AliasList::const_iterator aliasIt = aliases.begin();
    for (; aliasIt != aliases.end(); ++aliasIt)
        std::cout << "Alias: " << *aliasIt << std::endl;
     
    // Print addresses (IP addresses)
    const HostEntry::AddressList& addrs = entry.addresses();
     
 
    // 3 way to interator vector IPAddress
    //way 1:
    //for(const Poco::Net::IPAddress &it : addrs) {
    //    std::cout << it.toString() << std::endl;
    //}
     
    //way 2:
    for(int i = 0; i < addrs.size(); i++) {
        std::cout << addrs[i] << std::endl;
    }
     
    //way 3:
    //HostEntry::AddressList::const_iterator addrIt = addrs.begin();
    //for (; addrIt != addrs.end(); ++addrIt)
    //    std::cout << "Address: " << addrIt->toString() << std::endl;
 
    return 0;
}
```
**2. Socket**

POCO sockets are a very thin layer on top of BSD sockets and thus incur a minimal performance overhead – basically an additional call to a (virtual) function.

A Socket object stores only a pointer to a corresponding SocketImpl object. SocketImpl objects are reference counted. 

Poco::Net::ServerSocket is used to create a TCP server socket.

Poco::Net::Socket is the root class of the sockets inheritance tree. It supports methods that can be used with some or all kinds of sockets, like:

- select() and poll()
- setting and getting various socket options (timeouts, buffer sizes, reuse address flag, etc.)
- getting the socket's address and the peer's address

Poco::Net::StreamSocket is used for creating a TCP connection to a server.

Use sendBytes() and receiveBytes() to send and receive data, or use the Poco::Net::SocketStream class, which provides an I/O streams interface to a StreamSocket.

This program demonstrates how to perform **low-level HTTP communication** in C++ using **sockets:**
```c++
#include "Poco/Net/SocketAddress.h"
#include "Poco/Net/StreamSocket.h"
#include "Poco/Net/SocketStream.h"
#include "Poco/StreamCopier.h"
#include <iostream>
int main(int argc, char** argv)
{
    Poco::Net::SocketAddress sa("www.appinf.com", 80);
    Poco::Net::StreamSocket socket(sa);
 
    //way1:
    // Poco::Net::SocketStream str(socket);
    // str << "GET / HTTP/1.1\r\n"
    // "Host: www.appinf.com\r\n"
    // "\r\n";
    // str.flush();
    // Poco::StreamCopier::copyStream(str, std::cout);
 
    //way2:
    // Define the HTTP request as a C-style string
    const char* httpRequest = 
        "GET / HTTP/1.1\r\n"
        "Host: www.appinf.com\r\n"
        "Connection: close\r\n"  // Close connection after response
        "\r\n";
     
    // Send the HTTP request using sendBytes
    int bytesSent = socket.sendBytes(httpRequest, std::strlen(httpRequest));
    std::cout << "Sent " << bytesSent << " bytes to the server." << std::endl;
 
    // Buffer to store the response
    char buffer[1024];
    int bytesReceived = 0;
 
    // Continuously receive data from the server until no more data is available
    while ((bytesReceived = socket.receiveBytes(buffer, sizeof(buffer) - 1)) > 0)
    {
        // Null-terminate the received data and print it
        buffer[bytesReceived] = '\0';
        std::cout << buffer;
    }
    return 0;
}
```


The Program for local socket using Poco lib:
```c++:
//Server:
#include "Poco/Net/ServerSocket.h"
#include "Poco/Net/StreamSocket.h"
#include "Poco/Net/SocketStream.h"
#include "Poco/Net/SocketAddress.h"
#include <iostream>
int main(int argc, char** argv)
{
    Poco::Net::ServerSocket srv(8080); // does bind + listen
    while(1)
    {
        Poco::Net::StreamSocket ss = srv.acceptConnection();
        Poco::Net::SocketStream str(ss);
        str << "HTTP/1.0 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "\r\n"
        "<html><head><title>My 1st Web Server</title></head>"
        "<body><h1>Hello, world!</h1></body></html>"
        << std::flush;
        std::cout << "\nSent message to client" << std::endl;
    }
    return 0;
}

//Client:
#include "Poco/Net/SocketAddress.h"
#include "Poco/Net/StreamSocket.h"
#include "Poco/Net/SocketStream.h"
#include "Poco/StreamCopier.h"
#include <iostream>
int main(int argc, char** argv)
{
    Poco::Net::SocketAddress sa(8080);
    Poco::Net::StreamSocket socket(sa);
    Poco::Net::SocketStream str(socket);
    Poco::StreamCopier::copyStream(str, std::cout);
    std::cout << std::endl;
    return 0;
}
```

