# PRACTICA-3
# Práctica 3: WIFI Y BLUETOOTH

## Introducción de la práctica

El objetivo de esta práctica es trabajar con las funcionalidades de WiFi y Bluetooth utilizando el microprocesador ESP32. Se llevará a cabo la generación de un servidor web desde la placa y se establecerá una comunicación serie mediante Bluetooth con un dispositivo móvil.

## PRÁCTICA A - CONEXIÓN WIFI Y PÁGINA WEB BÁSICA

### Objetivo
El objetivo principal de este proyecto es conectar el ESP32-S3 a una red WiFi y crear una página web básica accesible a través de su dirección IP.

### Materiales
- ESP32-S3

### Procedimiento

#### Código:

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <Arduino.h>

const char* ssid = "Nautilus";  
const char* password = "20000Leguas";  
WebServer server(80);

void handle_root();

void setup() {
    Serial.begin(115200);
    Serial.println("Conectando a la red Wi-Fi...");
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }

    Serial.println("\nConexión establecida");
    Serial.print("Dirección IP: ");
    Serial.println(WiFi.localIP());

    server.on("/", handle_root);
    server.begin();
    Serial.println("Servidor HTTP iniciado");
}

void loop() {
    server.handleClient();
}
```

### Descripción
Este programa conecta el ESP32-S3 a una red WiFi y crea un servidor web. Una vez establecida la conexión, muestra la dirección IP en la terminal. Desde un navegador, se puede acceder a una página web básica generada por el ESP32.

### Salida esperada en la terminal:
```
Conexión WiFi establecida
Dirección IP: 192.168.58.26
Servidor HTTP iniciado
```

### Conclusión
Se establece un servidor web simple en un ESP32 que muestra una página HTML básica cuando se accede a la dirección IP del dispositivo desde un navegador web.

## PRÁCTICA B - COMUNICACIÓN BLUETOOTH CON EL MÓVIL

### Objetivo
Crear una conexión Bluetooth entre el ESP32 y un dispositivo móvil para permitir la transmisión de datos entre ambos.

### Materiales
- ESP32-S1
- Aplicación móvil: Serial Bluetooth Terminal

### Procedimiento

#### Código:

```cpp
#include "BluetoothSerial.h"
#if !defined(CONFIG_BT_ENABLED) || !defined(CONFIG_BLUEDROID_ENABLED)
#error Bluetooth is not enabled! Please run `make menuconfig` to and enable it
#endif
BluetoothSerial SerialBT;
void setup() {
Serial.begin(115200);
SerialBT.begin("ESP32test"); //Bluetooth device name
Serial.println("The device started, now you can pair it with bluetooth!");
}
void loop() {
if (Serial.available()) {
SerialBT.write(Serial.read());
}
if (SerialBT.available()) {
Serial.write(SerialBT.read());
}
delay(20);
}
```

### Descripción
Este código establece una conexión Bluetooth entre el ESP32 y un dispositivo móvil, permitiendo la transmisión de datos bidireccionalmente entre la comunicación serie USB y la conexión Bluetooth.

#### Funcionamiento:
- **`setup()`**: Inicializa la conexión y define el nombre del dispositivo Bluetooth.
- **`loop()`**: Verifica la disponibilidad de datos y los transmite entre la conexión USB y Bluetooth.

### Salidas esperadas:
Si se envían datos desde otro dispositivo al ESP32 a través de Bluetooth:
- Los datos serán recibidos y leídos por el ESP32 a través de `SerialBT.read()` y enviados a trav
