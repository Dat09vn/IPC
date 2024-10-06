**1. What is JSON?**

JSON (JavaScript Object Notation) is a lightweight data-interchange format that is easy for humans to read and write and easy for machines to parse and generate. It is commonly used for transmitting data between a server and a web application as text.

JSON is language-independent but uses conventions familiar to programmers of the C-family languages, including C, C++, Java, JavaScript, Perl, Python, and many others.

**2. Structure of JSON:**

- Objects: An unordered collection of key/value pairs. Keys must be strings, and values can be strings, numbers, arrays, Booleans, or other JSON objects.
```JSON
{
  "name": "Alice",
  "age": 25,
  "isStudent": false
}
```
- Arrays: An ordered list of values. Each value can be any of the types: string, number, object, array, or a special value.
```JSON
[
  "apple",
  "banana",
  "cherry"
]
```
- Data types:
```
String: "Hello"
Number: 123, 45.67
Boolean: true, false
Null: null
```
- Example of JSON for a person:
```JSON
{
  "firstName": "John",
  "lastName": "Doe",
  "age": 30,
  "isEmployed": true,
  "skills": ["C++", "Python", "Networking"],
  "address": {
    "street": "123 Main St",
    "city": "Hanoi",
    "country": "Vietnam"
  }
}
```
**3. JSON is widely used for APIs, configuration files, and data exchange between client-server applications.**

3.1 APIs (Application Programming Interfaces):

JSON is often used as the data format for RESTful APIs. When a client (e.g., a web application or mobile app) interacts with a server via an API, the server typically responds with data in JSON format. This makes it easy for the client to parse the response and use the data.

Example: A request to a weather API might return JSON like this:
```JSON
{
  "location": "Hanoi",
  "temperature": 30,
  "unit": "Celsius",
  "condition": "Sunny"
}
```
In this example, the server sends weather data in JSON format, and the client can easily interpret and display it.

3.2 Configuration Files:

JSON is commonly used for configuration files because of its clear and readable structure. It allows developers to define settings, preferences, or parameters for applications in a way that’s easy to understand and modify.

Example: A configuration file for a web server might look like this:
```JSON
{
  "server": {
    "host": "localhost",
    "port": 8080
  },
  "database": {
    "username": "admin",
    "password": "password123",
    "host": "localhost",
    "port": 5432
  }
}
```
3.3 Data Exchange Between Client and Server:

In web applications, JSON is frequently used for exchanging data between the client (browser) and server (backend). For example, when submitting a form, the client might send data as JSON to the server, which can then process the data and respond with JSON.

Example: A client might send a user registration form as JSON:
```JSON
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "securepassword"
}
```
The server would process this data and could respond with a success or error message in JSON format.

**4. Using Poco::JSON**

In C++, when using the Poco library, handling JSON can be done with the Poco::JSON namespace. The Poco::JSON::Parser is used for parsing JSON strings into objects, and Poco::JSON::Object or Poco::Dynamic::Var for manipulating and accessing JSON data.

```c++
#include <iostream>
#include <Poco/JSON/Object.h>
#include <Poco/JSON/Parser.h>
#include <Poco/Dynamic/Var.h>
#include <sstream>

int main() {
    // Multi-level JSON string
    std::string jsonString = R"({
        "name": "Alice",
        "age": 25,
        "isStudent": false,
        "address": {
            "street": "123 Main St",
            "city": "Hanoi",
            "zipCode": "100000"
        },
        "skills": ["C++", "Python", "Networking"]
    })";

    // Create a JSON parser object
    Poco::JSON::Parser parser;

    // Parse the JSON string
    Poco::Dynamic::Var result = parser.parse(jsonString);

    // Convert result to JSON object
    Poco::JSON::Object::Ptr jsonObject = result.extract<Poco::JSON::Object::Ptr>();

    // Access top-level values
    std::string name = jsonObject->getValue<std::string>("name");
    int age = jsonObject->getValue<int>("age");
    bool isStudent = jsonObject->getValue<bool>("isStudent");

    std::cout << "Name: " << name << ", Age: " << age << ", Is Student: " << std::boolalpha << isStudent << std::endl;

    // Access nested object (address)
    Poco::JSON::Object::Ptr address = jsonObject->getObject("address");
    std::string street = address->getValue<std::string>("street");
    std::string city = address->getValue<std::string>("city");
    std::string zipCode = address->getValue<std::string>("zipCode");

    std::cout << "Address: " << street << ", " << city << ", " << zipCode << std::endl;

    // Access array (skills)
    Poco::JSON::Array::Ptr skills = jsonObject->getArray("skills");
    std::cout << "Skills: ";
    for (size_t i = 0; i < skills->size(); ++i) {
        std::cout << skills->getElement<std::string>(i) << (i < skills->size() - 1 ? ", " : "");
    }
    std::cout << std::endl;

    // Modify nested JSON values (e.g., change age and city)
    jsonObject->set("age", 26);
    address->set("city", "Ho Chi Minh City");

    // Serialize the modified JSON object back to a string
    std::ostringstream oss;
    jsonObject->stringify(oss);
    std::string modifiedJsonString = oss.str();

    std::cout << "Modified JSON: " << modifiedJsonString << std::endl;

    return 0;
}
```

- Parsing JSON:
  - We use Poco::JSON::Parser to parse a JSON string.
  - The parsed result is stored in a Poco::Dynamic::Var, which is then extracted into a Poco::JSON::Object::Ptr.
- Accessing JSON Values:
  - You can access values from the JSON object using the getValue<>() method, specifying the type you expect.
- Modifying JSON:
  - The set method allows you to modify the JSON data, such as changing the value of age.
- Serializing JSON:
  - The stringify method serializes the modified JSON object back into a string for output or transmission.
    
