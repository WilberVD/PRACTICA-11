# PRACTICA-11
ENVIAR DATOS A BASE DE DATOS SERVIDOR  LOCAL
## Introducción

EN ESTE REPOSITORIO VEREMOS LA CONFIGURACION PARA PODER ENVIAR LOS DATOS DE NUESTRA PLACA **ES32** A UNA **BASE DE DATOS** LOCAL



### Descripción

La ```Esp32``` se conecta mediante wi-fi a un servidor y puede realizar diferentes actividades deseadas por el usuario ; Cabe aclarar que en  esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/).


## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Node-RED
- XAMPP
- SENSOR DE TEMPERATURA Y HUMEDAD

## Instrucciones
Instalar el software Node-RED
instalar paqueterias 
Instalar el software XAMPP
instalar paqueterias  y complementos para el correcto funcionamiento de nuestros programas


## COMPLEMENTO MYSQL(NODE-RED)
![](https://github.com/WilberVD/PRACTICA-11/blob/main/lib.jpg)

### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "18.193.219.109";

const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 2;   //Pin digital 3 para el Echo del sensor

String username_mqtt="educatronicosiot";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  Serial.begin(9600);//iniciailzamos la comunicación
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
long t; //timepo que demora en llegar el eco
long d; //distancia en centimetros

digitalWrite(Trigger, HIGH);
delayMicroseconds(10);          //Enviamos un pulso de 10us
digitalWrite(Trigger, LOW);
  
t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
d = t/59;             //escalamos el tiempo a una distancia en cm
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    doc["DISTANCIA"] = String(d);
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("wilbertemperatura", output.c_str());
  }
}
```

2. AGREGAR UNA FUNCION EN NODE RED ANTES DE MYSQL Y AGREGAR EL SIGUIENTE CODIGO(EN LA FUNCION)
```
   var query = "INSERT INTO `wilberbase`(`ID`, `FECHA`, `DEVICE`, `TEMPERATURA`, `HUMEDAD`) VALUES (NULL, current_timestamp(), '";
query = query + msg.payload.DEVICE + "','";
query = query + msg.payload.TEMPERATURA + "','";
query = query + msg.payload.HUMEDAD + "');'";
msg.topic = query;
return msg
 ```
### CONFIGURACION EN NODE RED
![](https://github.com/WilberVD/PRACTICA-11/blob/main/node%20res.jpg)
### CONFIGURACION DE MYSQL
![](https://github.com/WilberVD/PRACTICA-11/blob/main/confmysq.jpg)


### Instrucciónes de operación

1.CREAR BASE DE DATOS EN XAMPP
![](https://github.com/WilberVD/PRACTICA-11/blob/main/BASECONF.jpg)

2.Iniciar simulador.
3. Visualizar los datos en el monitor serial.



### Cuando haya funcionado, verás los valores dentro del monitor serial como se muestra en la siguente imagen.

![](https://github.com/WilberVD/PRACTICA-11/blob/main/mingraf.jpg)
![](https://github.com/WilberVD/PRACTICA-11/blob/main/datos.jpg)

# Créditos

Desarrollado por Wilber Valladares Diaz

- [GitHub](WilberVD (github.com))
- [LINK DEL PROGRAMA EN WOKI](https://wokwi.com/projects/388080729985489921)
