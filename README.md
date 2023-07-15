# Esp32 Portal Cautivo

<img src="https://github.com/IDiegoUlises/Esp32-Portal-Cautivo/blob/main/Imagenes/IMG_20230714_215346.jpg" width="400" height="800" />
* Al conectarse a la red WiFi abrira automaticamente el portal cautivo

### Codigo
```c++
#include <DNSServer.h>
#include <WiFi.h>
#include <AsyncTCP.h>
#include "ESPAsyncWebServer.h"

DNSServer dnsServer;
AsyncWebServer server(80);

String user_name;
String proficiency;
bool name_received = false;
bool proficiency_received = false;

const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE HTML><html><head>
  <title>Portal Cautivo Demo</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  </head><body>
  <h3>Portal Cautivo Demo</h3>
  <br><br>
  <form action="/get">
    <br>
    Name: <input type="text" name="name">
    <br>
    ESP32 Habilidad: 
    <select name = "proficiency">
      <option value=Beginner>Beginner</option>
      <option value=Advanced>Advanced</option>
      <option value=Pro>Pro</option>
    </select>
    <input type="submit" value="Submit">
  </form>
</body></html>)rawliteral";

class CaptiveRequestHandler : public AsyncWebHandler {
  public:
    CaptiveRequestHandler() {}
    virtual ~CaptiveRequestHandler() {}

    bool canHandle(AsyncWebServerRequest *request) {
      return true;
    }

    void handleRequest(AsyncWebServerRequest *request) {
      request->send_P(200, "text/html", index_html);
    }
};

void setupServer() {
  server.on("/", HTTP_GET, [](AsyncWebServerRequest * request) {
    request->send_P(200, "text/html", index_html);
    Serial.println("Client Connected");
  });

  server.on("/get", HTTP_GET, [] (AsyncWebServerRequest * request) {
    String inputMessage;
    String inputParam;

    if (request->hasParam("name")) {
      inputMessage = request->getParam("name")->value();
      inputParam = "name";
      user_name = inputMessage;
      Serial.println(inputMessage);
      name_received = true;
    }

    if (request->hasParam("proficiency")) {
      inputMessage = request->getParam("proficiency")->value();
      inputParam = "proficiency";
      proficiency = inputMessage;
      Serial.println(inputMessage);
      proficiency_received = true;
    }
    request->send(200, "text/html", "The values entered by you have been successfully sent to the device <br><a href=\"/\">Return to Home Page</a>");
  });
}


void setup() 
{
  //Inicia el puerto serial
  Serial.begin(115200);
  Serial.println();
  Serial.println("Setting up AP Mode");

  //Se declara como punto de acceso
  WiFi.mode(WIFI_AP);

  //El nombre del punto de acceso
  WiFi.softAP("Esp32-Portal");

  //Imprime la direccion IP asignada
  Serial.print("AP IP address: ");
  Serial.println(WiFi.softAPIP());
  Serial.println("Setting up Async WebServer");

  //Configura el servicio DNS
  setupServer();
  Serial.println("Starting DNS Server");
  dnsServer.start(53, "*", WiFi.softAPIP());
  server.addHandler(new CaptiveRequestHandler()).setFilter(ON_AP_FILTER); //solo cuando se solicita de AP

  //Iniciar el servidor
  server.begin();
  Serial.println("Funcionando Correctamente!");
}

void loop() 
{
  
  dnsServer.processNextRequest();
  
  if (name_received && proficiency_received)
  {
    Serial.print("Hola ");
    Serial.println(user_name);
    Serial.print("Usted ha declarado su habilidad ");
    Serial.println(proficiency);
    name_received = false;
    proficiency_received = false;
    Serial.println("Esperaremos al proximo cliente ahora");
  }
}
```

<img src="https://github.com/IDiegoUlises/Esp32-Portal-Cautivo/blob/main/Imagenes/IMG_20230714_215451.jpg" width="400" height="800" />


