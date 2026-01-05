# Heltec LoRa V3 - Nodo LoRaWAN (OTAA)

Este proyecto implementa un nodo final LoRaWAN utilizando la placa de desarrollo Heltec WiFi LoRa 32 V3. El firmware simula la lectura de un sensor de temperatura, muestra los valores en la pantalla OLED integrada y transmite los datos a un servidor de red (Network Server) como ChirpStack o The Things Network (TTN) utilizando el protocolo de activación OTAA (Over-The-Air Activation).

## Características

* Hardware: Optimizado para Heltec LoRa V3 (ESP32-S3 + SX1262).
* Conexión: LoRaWAN Clase A vía OTAA.
* Gestión de Energía: Ciclo de sueño profundo (Deep Sleep) entre transmisiones para ahorro de batería.
* Interfaz: Visualización de estado y datos en pantalla OLED (0.96").
* Datos: Generación de temperatura simulada (random) y codificación en bytes.

## Requisitos de Hardware y Software

* Placa: Heltec WiFi LoRa 32 V3.
* IDE: Arduino IDE.
* Board Manager: Heltec ESP32 Series Dev-boards (Instalado a través del Gestor de Tarjetas).
* Librerías:
    * LoRaWan_APP.h (Incluida en el paquete de Heltec).
    * HT_SSD1306Wire.h (Driver para la pantalla OLED).

## Configuración del Código

Antes de cargar el código, es obligatorio modificar las credenciales de seguridad en el archivo LoRaWan.ino para que coincidan con las generadas en tu servidor ChirpStack.

### 1. Credenciales LoRaWAN (OTAA)

Busca las siguientes líneas en el código y reemplaza los valores hexadecimales con tus claves (asegúrate de usar el formato correcto, generalmente MSB para ChirpStack):

```cpp
// DEBES reemplazar esto con tus llaves de ChirpStack
uint8_t devEui[8] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 }; // Device EUI
uint8_t appEui[8] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 }; // Join EUI (App EUI)
uint8_t appKey[16] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 }; // App Key
```


### 2. Máscara de Canales (Región US915)
El código está preconfigurado para la región US915 en la sub-banda 1 (Canales 0-7), lo cual es ideal para gateways de 8 canales (como el Heltec o Dragino).

/* 0x00FF habilita los primeros 8 canales (FSB 1). 
   Cambiar si tu gateway usa otra sub-banda.
*/
uint16_t userChannelsMask[6]={ 0x00FF,0x0000,0x0000,0x0000,0x0000,0x0000 };

### Explicación del Código
El código funciona bajo una máquina de estados controlada por la librería LoRaWan_APP:

Setup: Inicializa la comunicación serial, configura pines de energía (VextON) e inicia la pantalla OLED.

Loop (Máquina de Estados):

DEVICE_STATE_INIT: Inicializa la pila LoRaWAN con la clase y región definidas.

DEVICE_STATE_JOIN: Intenta unirse a la red mediante OTAA (LoRaWAN.join()).

DEVICE_STATE_SEND:

Despierta la pantalla y sensores.

Genera una temperatura aleatoria (prepareTxFrame).

Empaqueta el dato en el buffer appData.

Muestra el valor en la OLED.

Envía el paquete (LoRaWAN.send()).

DEVICE_STATE_CYCLE: Calcula el tiempo de espera aleatorio para la próxima transmisión.

DEVICE_STATE_SLEEP: Pone el microcontrolador en modo de bajo consumo.
