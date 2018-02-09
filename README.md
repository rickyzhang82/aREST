<h1>bREST</h1>


## What
bRest is an extension of [aREST project](https://github.com/marcoschwartz/aREST). Its main purpose is to provide truly flexible RESTful API in ESP8266 chip.

bREST supports the following [HTTP RFC 2616](https://www.ietf.org/rfc/rfc2616.txt) standards:
- Parse one and only one HTTP request `Method SP Request-URI SP HTTP-Version CRLF`. Disregard all the noise.
- Support two methods: GET and PUT. Disregard the rest of HTTP methods.
  + GET method refers to get resource status.
  + PUT method refers to update resource status.
- Support two types of request URI:
  + absoluteURI. i.e. http://wherever.com/pin1/?mode=digital&value=high
  + abs_path. i.e. /pin1/?mode=digital&value=high
- Support one and only one level of URI. It represent the unique ID of your customized resource.
  + For example, a request `PUT http://wherever.com/servo1/?angle=120 HTTP/1.1`. It means update resource `servo1` with angle `120`.
- Support unlimited number of parameters and value pair in URL.

Compared to aREST, bREST is more
- Fault tolerance. If input doesn't follows requirements above, it output errors and stop processing.
- Resource oriented. Your customized class is a resource. A true thinking in RESTful API.
- Object oriented. Your customized class inherits from a observer. bREST will automatically invoke your call back method. Observer pattern is more readable than a function pointer.
- Flexible input and output. Zero restriction on the number of input parameters from URL. Zero restriction on the return JSON string.

## How
First, clone bREST repo to your `Arduino/libraries`

Secondly, design your class by inheriting class `Observer`. Conceptually, your class should be one of resource such as servo, serial port or even a pin.

Thirdly, add your customized class to `bREST`.

See Step 1, Step 2 and Step 3 in sample below:
```C++
// Import required libraries
#include <ESP8266WiFi.h>
#include <bREST.h>

// Create bREST instance
bREST rest = bREST();

// WiFi parameters
const char* ssid = "your_ssid";
const char* password = "your_password";
IPAddress ip(192, 168, 2, 41);
IPAddress gateway(192, 168, 2, 1);
IPAddress subnet_mask(255, 255, 255, 0);
IPAddress dns(192, 168, 2, 1);

// The port to listen for incoming TCP connections
#define LISTEN_PORT           80

// Create an instance of the server
WiFiServer server(LISTEN_PORT);

// Step1: Define customized resource by inheriting Observer
//        Override call back method update()
class SerialPortResource: public Observer {
public:
    SerialPortResource(String resource_id): Observer(resource_id) {}
    virtual ~SerialPortResource(){}
    // override call back function
    void update(HTTP_METHOD method, String parms[], String value[], int parm_count) override {
        Serial.println("*************************************");
        Serial.println("fire SerialPort update()!");
        Serial.print("HTTP Method:");
        Serial.println(get_method(method));
        Serial.println("Parameters and Value:");
        for (int i = 0; i < parm_count; i++) {
            Serial.print(parms[i]);
            Serial.print(" = ");
            Serial.println(value[i]);
        }
        Serial.println("*************************************");
    }
};

// Step 2: Allocate resource with unique ID
SerialPortResource mySerialPort("serial0");

void setup(void)
{
  // Start Serial
  Serial.begin(115200);

  // Connect to WiFi
  WiFi.config(ip, gateway, subnet_mask, dns);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  // Start the server
  server.begin();
  Serial.println("Server started");

  // Print the IP address
  Serial.println(WiFi.localIP());
  // Step 3: Add observer
  rest.add_observer(&mySerialPort);
}

void loop() {

  // Handle REST calls
  WiFiClient client = server.available();
  if (!client) {
    return;
  }
  while(!client.available()){
    delay(1);
  }
  rest.handle(client);

}

```
