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
Dirección IP: 192.168.12.241
Servidor HTTP iniciado
```

### Conclusión
Se establece un servidor web simple en un ESP32 que muestra una página HTML básica cuando se accede a la dirección IP del dispositivo desde un navegador web.

### Código
```cpp
#include <WiFi.h>
#include <WebServer.h>

// SSID & Password
const char* ssid = "Nelfi";
const char* password = "12345678";

WebServer server(80);

// HTML, CSS y JavaScript para el juego 3 en raya
String HTML = "<!DOCTYPE html>\
<html>\
<head>\
    <meta charset='UTF-8'>\
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>\
    <title>Blackjack con ESP32</title>\
    <style>\
        body { font-family: Arial, sans-serif; text-align: center; background-color: #1a3d22; color: white; }\
        h1 { color: #f39c12; }\
        .card { display: inline-block; width: 80px; height: 120px; margin: 10px; border-radius: 10px; text-align: center; font-size: 24px; line-height: 120px; background-color: #f39c12; color: #2c3e50; box-shadow: 3px 3px 15px rgba(0, 0, 0, 0.3); font-weight: bold; }\
        .card img { width: 60px; height: 90px; border-radius: 5px; }\
        .hand { margin: 20px 0; }\
        .buttons { margin-top: 20px; }\
        button { padding: 10px 20px; background-color: #27ae60; color: white; border: none; cursor: pointer; font-size: 18px; margin: 10px; }\
        button:hover { background-color: #2ecc71; }\
        .status { margin-top: 20px; font-size: 24px; }\
        .hidden-card { background-color: #34495e; color: transparent; font-size: 0; }\
        .restart-button { display: none; margin-top: 20px; }\
        .separator { margin: 20px; border-top: 2px solid #fff; width: 80%; }\
        .hand-title { font-size: 24px; color: #ecf0f1; margin: 10px 0; }\
    </style>\
</head>\
<body>\
    <h1>Blackjack - Juega contra el Croupier</h1>\
    <div class='hand-title'>Tu Mano</div>\
    <div class='hand' id='player-hand'></div>\
    <div class='separator'></div>\
    <div class='hand-title'>Mano del Croupier</div>\
    <div class='hand' id='dealer-hand'></div>\
    <div class='buttons'>\
        <button onclick='hit()'>Pedir Carta</button>\
        <button onclick='stand()'>Plantarse</button>\
    </div>\
    <div class='status' id='status'>Bienvenido al Blackjack</div>\
    <button class='restart-button' id='restart-button' onclick='restartGame()'>Volver a Jugar</button>\
    <script>\
        const suits = ['♠', '♥', '♦', '♣'];\
        const values = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];\
        let deck = [];\
        let playerHand = [];\
        let dealerHand = [];\
        let gameOver = false;\
        let playerScore = 0;\
        let dealerScore = 0;\
        let dealerScoreRevealed = false;\
\
        function createDeck() {\
            deck = [];\
            for (let suit of suits) {\
                for (let value of values) {\
                    deck.push({value: value, suit: suit});\
                }\
            }\
            shuffle(deck);\
        }\
\
        function shuffle(array) {\
            for (let i = array.length - 1; i > 0; i--) {\
                const j = Math.floor(Math.random() * (i + 1));\
                [array[i], array[j]] = [array[j], array[i]];\
            }\
        }\
\
        function startGame() {\
            createDeck();\
            playerHand = [deck.pop(), deck.pop()];\
            dealerHand = [deck.pop(), deck.pop()];\
            gameOver = false;\
            dealerScoreRevealed = false;\
            playerScore = calculateScore(playerHand);\
            dealerScore = calculateScore(dealerHand);\
            document.getElementById('restart-button').style.display = 'none';\
            updateUI();\
        }\
\
        function calculateScore(hand) {\
            let score = 0;\
            let aceCount = 0;\
            for (let card of hand) {\
                if (card.value === 'A') {\
                    aceCount++;\
                    score += 11;\
                } else if (['K', 'Q', 'J'].includes(card.value)) {\
                    score += 10;\
                } else {\
                    score += parseInt(card.value);\
                }\
            }\
            while (score > 21 && aceCount > 0) {\
                score -= 10;\
                aceCount--;\
            }\
            return score;\
        }\
\
        function updateUI() {\
            document.getElementById('player-hand').innerHTML = playerHand.map(card => `<div class='card'>${card.value}${card.suit}</div>`).join('');\
            document.getElementById('dealer-hand').innerHTML = dealerHand.map((card, index) => `<div class='card ${index === 0 && !dealerScoreRevealed ? 'hidden-card' : ''}'>${index === 0 && !dealerScoreRevealed ? '🔒' : card.value + card.suit}</div>`).join('');\
            document.getElementById('status').innerText = gameOver ? (playerScore > 21 ? 'Te has pasado. Croupier gana.' : (dealerScore > 21 ? 'Croupier se pasa. Tú ganas.' : (playerScore === dealerScore ? 'Empate.' : (playerScore > dealerScore ? '¡Tú ganas!' : 'Croupier gana.')))) : `Tu puntaje: ${playerScore} | Puntaje Croupier: ${dealerScoreRevealed ? dealerScore : '???'}`;\
            if (gameOver) {\
                document.getElementById('restart-button').style.display = 'block';\
            }\
        }\
\
        function hit() {\
            if (gameOver) return;\
            playerHand.push(deck.pop());\
            playerScore = calculateScore(playerHand);\
            if (playerScore > 21) {\
                gameOver = true;\
            }\
            updateUI();\
        }\
\
        function stand() {\
            if (gameOver) return;\
            while (dealerScore < 17) {\
                dealerHand.push(deck.pop());\
                dealerScore = calculateScore(dealerHand);\
            }\
            dealerScoreRevealed = true;\
            gameOver = true;\
            updateUI();\
        }\
\
        function restartGame() {\
            startGame();\
        }\
\
        startGame();\
    </script>\
</body>\
</html>";

// Manejo de la raíz
void handle_root() {
    server.send(200, "text/html", HTML);
}

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.println("\nWiFi conectado con IP: " + WiFi.localIP().toString());
    server.on("/", handle_root);
    server.begin();
    Serial.println("Servidor HTTP iniciado");
}

void loop() {
    server.handleClient();
}
```

### Descripción
Apartir del Ip generado, se puede acceder a una página web básica generada por el ESP32 que consiste en el juego de cartas del 21.
![image](https://github.com/user-attachments/assets/b73c37a6-422b-4fe4-bbdb-0855f5696fe0)


## PRÁCTICA B - COMUNICACIÓN BLUETOOTH CON EL MÓVIL

### Objetivo
Crear una conexión Bluetooth entre el ESP32 y un dispositivo móvil para permitir la transmisión de datos entre ambos.

### Materiales
- ESP32-S3
- Aplicación móvil: Serial Bluetooth Terminal

### Procedimiento

#### Código:

```cpp
#include <Arduino.h>
#include <NimBLEDevice.h>

// Definir UUIDs para el servicio y la característica
#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"

NimBLEServer* pServer = nullptr;
NimBLECharacteristic* pCharacteristic = nullptr;

void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando BLE con NimBLE!");

    // Inicializar el dispositivo BLE con un nombre
    NimBLEDevice::init("ESP32-NimBLE");

    // Crear el servidor BLE
    pServer = NimBLEDevice::createServer();

    // Crear el servicio BLE
    NimBLEService* pService = pServer->createService(SERVICE_UUID);

    // Crear la característica BLE con permisos de lectura y escritura
    pCharacteristic = pService->createCharacteristic(
                          CHARACTERISTIC_UUID,
                          NIMBLE_PROPERTY::READ | NIMBLE_PROPERTY::WRITE
                      );

    // Establecer un valor inicial para la característica
    pCharacteristic->setValue("Hola desde ESP32 NimBLE");

    // Iniciar el servicio
    pService->start();

    // Iniciar la publicidad para que otros dispositivos puedan encontrar el servidor
    NimBLEAdvertising* pAdvertising = NimBLEDevice::getAdvertising();
    pAdvertising->addServiceUUID(SERVICE_UUID);
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);
    pAdvertising->setMinPreferred(0x12);
    NimBLEDevice::startAdvertising();

    Serial.println("¡Servidor BLE NimBLE activo!");
}

void loop() {
    delay(2000); // Pequeña pausa en el loop
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

## **Ejercicios de mejora de nota:**
### **Ejercicio de mejora 1:**
**Codigo:**
```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <WebServer.h>

void handle_root();
// Configuración del Access Point (AP)
const char* ssid = "Nautilus";  // Enter your SSID here 
const char* password = "20000Leguas";  //Enter your Password here 

WebServer server(80); // Servidor web en el puerto 80

// Página HTML que se mostrará en el navegador
String HTML = "<!DOCTYPE html>\
<html>\
<head><title>ESP32 Access Point</title></head>\
<body>\
<h1>ESP32 en Modo AP</h1>\
<p>Estás conectado al ESP32 en modo Access Point.</p>\
</body>\
</html>";

// Manejo de la raíz "/"
void handle_root() {
    server.send(200, "text/html", HTML);
}

void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando Access Point...");

    // Configurar el ESP32 como Access Point (sin conexión a internet)
    WiFi.softAP(ssid, password);

    // Mostrar la dirección IP del AP
    Serial.print("AP IP address: ");
    Serial.println(WiFi.softAPIP());

    // Configurar el servidor web
    server.on("/", handle_root);
    server.begin();
    Serial.println("Servidor web iniciado en modo AP.");
}

void loop() {
    server.handleClient(); // Manejar solicitudes de clientes
}

```
Este código configura un ESP32 como punto de acceso WiFi (AP) y servidor web en el puerto 80. Se define un SSID y una contraseña con WiFi.softAP(), y se imprime la IP asignada con WiFi.softAPIP().
El servidor web responde en la raíz ("/") con una página HTML simple mediante server.on("/", handle_root). Finalmente, server.handleClient() mantiene el servidor activo.


### **Ejercicio de mejora 2:**
**Codigo:**
```cpp
#include <Arduino.h>
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>

// Definir UUIDs para el servicio y la característica
#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"

void setup() {
  Serial.begin(115200);
  Serial.println("Iniciando trabajo con BLE!");

  // Inicializar el dispositivo BLE con un nombre
  BLEDevice::init("MyESP32");

  // Crear el servidor BLE
  BLEServer *pServer = BLEDevice::createServer();

  // Crear el servicio BLE
  BLEService *pService = pServer->createService(SERVICE_UUID);

  // Crear la característica BLE con permisos de lectura y escritura
  BLECharacteristic *pCharacteristic = pService->createCharacteristic(
                                         CHARACTERISTIC_UUID,
                                         BLECharacteristic::PROPERTY_READ |
                                         BLECharacteristic::PROPERTY_WRITE
                                       );

  // Establecer un valor inicial para la característica
  pCharacteristic->setValue("Hola Mundo desde ESP32");
  
  // Iniciar el servicio
  pService->start();

  // Iniciar la publicidad para que otros dispositivos puedan encontrar el servidor
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pAdvertising->setScanResponse(true);
  pAdvertising->setMinPreferred(0x06);  // Funciones que ayudan con problemas de conexión en iPhone
  pAdvertising->setMinPreferred(0x12);
  BLEDevice::startAdvertising();

  Serial.println("¡Característica definida! Ahora puedes leerla en tu dispositivo.");
}

void loop() {
  // Código principal que se ejecuta repetidamente
  delay(2000);
}

```

Aquí, el ESP32 actúa como un servidor BLE, permitiendo que otros dispositivos lean y escriban datos. Se crea un servicio BLE con un UUID único y una característica BLE con permisos de lectura/escritura.
En setup(), se inicializa BLE (BLEDevice::init()), se crea el servidor y se configura la publicidad con BLEDevice::startAdvertising(). El loop() solo mantiene el programa corriendo con delay(2000), ya que BLE funciona en segundo plano.






  

