
# **Monitoreo de Señales**  

## **Objetivo**  
En esta práctica, aprenderás a configurar un servidor en **Node.js** que recibirá datos de un **ESP32** a través de **WebSockets** utilizando la biblioteca `socket.io@1.7.2`. Además, el servidor ofrecerá una página web donde se visualizarán los datos en tiempo real.  

Esta práctica se enfoca en el **monitoreo de una señales**, permitiendo la comunicación entre el ESP32 y el servidor Node.js mediante **WebSockets**.  

---  

## **1. Instalación del Entorno y Configuración del Servidor Node.js**  

Para ejecutar el servidor, es necesario contar con **Node.js** instalado en tu computadora.  

### **1.1 Instalación de Node.js**  
Si no tienes instalado **Node.js**, descárgalo e instálalo desde la página oficial:  

🔗 [Descargar Node.js](https://nodejs.org/)  

Selecciona la versión LTS (Long-Term Support), ya que es la más estable.  

Una vez instalado, verifica que la instalación fue exitosa ejecutando el siguiente comando en la terminal o consola:  

```sh
node -v
```

Esto te devolverá la versión instalada de Node.js.  

También verifica que **npm** (Node Package Manager) esté instalado ejecutando:  

```sh
npm -v
```

Si ambos comandos muestran una versión, significa que la instalación fue exitosa.  

---

### **1.2 Creación del Proyecto Node.js**  

1. Abre una terminal o consola de comandos y crea una nueva carpeta para el proyecto:  

   ```sh
   mkdir servidor_esp32
   cd servidor_esp32
   ```

2. Inicializa un nuevo proyecto de Node.js con el siguiente comando:  

   ```sh
   npm init -y
   ```

   Esto generará un archivo `package.json` con la configuración básica del proyecto.  

---

### **1.3 Instalación de Dependencias**  

Para que el servidor funcione correctamente, es necesario instalar los siguientes paquetes:  

| Paquete  | Descripción |
|----------|------------|
| `express` | Framework para crear el servidor web. |
| `socket.io@1.7.2` | Biblioteca de WebSockets compatible con el ESP32. |
| `nodemon` | Herramienta para reiniciar automáticamente el servidor cuando hay cambios en el código. |

Ejecuta el siguiente comando para instalar las dependencias:  

```sh
npm install express socket.io@1.7.2
```

Y para instalar `nodemon` (opcional pero recomendado para desarrollo):  

```sh
npm install -g nodemon
```

---

### **1.4 Creación del Servidor Node.js**  

Dentro de la carpeta del proyecto, crea un archivo llamado **`server.js`** y copia el siguiente código:

```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

// Servir archivos estáticos desde la carpeta 'public'
app.use(express.static('public'));

io.on('connection', (socket) => {
    console.log('Nuevo dispositivo conectado.');

    // Recibir datos del ESP32
    socket.on('data', (data) => {
        console.log(`Datos recibidos del ESP32: ${data}`);
        // Aquí se puede agregar procesamiento de datos o almacenamiento
    });

    // Recibir datos desde la página web
    socket.on('desde_cliente', (data) => {
        console.log(`Datos recibidos desde la página: ${data}`);
        io.sockets.emit("desde_servidor_comando", data);
    });

    // Recibir datos desde el ESP32 y retransmitirlos a los clientes conectados
    socket.on('desde_esp32', (data) => {
        console.log(`ESP32: ${data}`);
        io.sockets.emit("retransmision_esp32", data);
    });

    // Manejar desconexiones
    socket.on('disconnect', () => {
        console.log('Dispositivo desconectado.');
    });
});

// Iniciar el servidor en el puerto 3000
server.listen(3000, () => {
    console.log('Servidor escuchando en puerto 3000');
});
```

---

### **1.5 Explicación del Código**  

1. **Importación de Módulos:**  
   - `express`: Para manejar el servidor HTTP.  
   - `http`: Para crear un servidor HTTP que pueda usar **WebSockets**.  
   - `socket.io`: Para manejar conexiones WebSockets.  

2. **Creación del Servidor:**  
   - Se crea una instancia de `express()`.  
   - Se usa `http.createServer(app)` para crear un servidor HTTP.  
   - Se integra **Socket.io** con `socketIo(server)`.  

3. **Servidor de Archivos Estáticos:**  
   - `app.use(express.static('public'))` sirve archivos desde la carpeta `public`.  
   - Esto permitirá más adelante que el cliente web cargue una interfaz gráfica para ver los datos.  

4. **Manejo de Conexiones WebSockets:**  
   - `io.on('connection', (socket) => { ... })` se ejecuta cuando un nuevo dispositivo (ESP32 o página web) se conecta.  
   - Se definen eventos personalizados como:  
     - `socket.on('data', (data) => { ... })` para recibir datos del **ESP32**.  
     - `socket.on('desde_cliente', (data) => { ... })` para recibir comandos de la página web.  
     - `socket.on('desde_esp32', (data) => { ... })` para retransmitir datos del ESP32 a todos los clientes.  

5. **Manejo de Desconexiones:**  
   - `socket.on('disconnect', () => { ... })` detecta cuando un dispositivo se desconecta.  

6. **Inicio del Servidor:**  
   - `server.listen(3000, () => { ... })` inicia el servidor en el **puerto 3000**.  

---

### **1.6 Ejecución del Servidor**  

Antes de ejecutar el servidor, asegúrate de que tienes los archivos correctos y las dependencias instaladas. Luego, en la terminal, ejecuta:

```sh
nodemon server.js
```

Si no instalaste `nodemon`, puedes ejecutar:

```sh
node server.js
```

Si todo está correcto, deberías ver el mensaje:  

```
Servidor escuchando en puerto 3000
```

Esto significa que el servidor está en funcionamiento. **Déjalo ejecutándose**, ya que en la siguiente parte configuraremos el ESP32 para enviar datos al servidor.  

Este es el **primer paso** de la práctica. Ahora pasaremos a configurar el **ESP32** para enviar datos al servidor. 🚀

---

### **2. Configuración del ESP32 y Envío de Datos al Servidor**

En esta sección, configuraremos el ESP32 para que se conecte a una red WiFi, establezca comunicación con el servidor Node.js mediante **WebSockets** y envíe datos analógicos en tiempo real.

---

## **2.1 Instalación de Bibliotecas Requeridas**

Para que el ESP32 pueda comunicarse con el servidor usando **WebSockets**, es necesario instalar las siguientes bibliotecas en el **Arduino IDE** o en **PlatformIO**:

### **Bibliotecas necesarias**
1. **WebSockets** de Markus Sattler  
   🔗 Descarga desde el Administrador de Bibliotecas de Arduino como **WebSockets by Markus Sattler**  
2. **SocketIoClient** de Vincent  
   🔗 Descarga desde el Administrador de Bibliotecas de Arduino como **SocketIoClient by Vincent**  

Para instalar en **Arduino IDE**, sigue estos pasos:
1. Abre **Arduino IDE**.
2. Ve a **Herramientas** → **Administrar bibliotecas...**
3. Busca **WebSockets by Markus Sattler** e instálala.
4. Busca **SocketIoClient by Vincent** e instálala.

### **Modificación en la biblioteca `SocketIoClient`**
La biblioteca de `SocketIoClient` tiene un problema con la función `hexdump()`, por lo que debes **comentar la línea donde se llama a esta función**.

1. Abre el Administrador de Bibliotecas en **Arduino IDE**.
2. Busca `SocketIoClient`, selecciona la versión de Vincent y ábrela en la carpeta de instalación.
3. Busca el archivo **`SocketIoClient.cpp`** dentro de la carpeta de la biblioteca.
4. Comenta la línea donde se llama a `hexdump()`, agregando `//` al inicio de la línea.

```cpp
// hexdump(payload, length);
```

5. Guarda los cambios.

Si no realizas esta modificación, el ESP32 podría arrojar errores al compilar o ejecutar el código.

---

## **2.2 Esquemático de Conexión del ESP32**
Para esta práctica, utilizaremos una señal analógica conectada a una entrada del ESP32 y varios LEDs para responder a los comandos del servidor.

### **Materiales Necesarios**
- **ESP32**
- **Fuente de señal analógica** (potenciómetro o sensor de voltaje)
- **Resistencias y LEDs** para indicar comandos del servidor

### **Conexiones:**
| Componente | Pin del ESP32 |
|------------|--------------|
| Señal analógica (sensor/potenciómetro) | **GPIO 34** |
| LED 1 | **GPIO 12** |
| LED 2 | **GPIO 14** |
| LED 3 | **GPIO 27** |
| LED 4 | **GPIO 26** |
| LED integrado | **GPIO 2** |

---

## **2.3 Código para el ESP32**
A continuación, copia y carga este código en tu ESP32.  

```cpp
#include <WiFi.h>
#include <SocketIoClient.h>

#define ONBOARD_LED 2  // LED integrado en la placa ESP32

// Configuración de la red WiFi y servidor
const char* ssid     = "nombre-red";  // Cambia esto por el nombre de tu red WiFi
const char* password = "contrasena";  // Cambia esto por la contraseña de tu red
const char* server   = "server-ip";   // IP del servidor Node.js
const uint16_t port  = 3000;          // Puerto en el que corre el servidor

// Variables para el envío de datos
String mensaje;
uint64_t now = 0;
uint64_t timestamp = 0;

int valor;
float valor_f;

// Inicialización del cliente de WebSockets
SocketIoClient socketIO;

void setup() {
  Serial.begin(115200);
  
  // Conectar a la red WiFi
  conectar_WiFiSTA();
  
  // Configurar WebSockets y definir eventos
  socketIO.begin(server, port);
  socketIO.on("desde_servidor_comando", procesar_mensaje_recibido);

  // Configurar pines de salida
  pinMode(ONBOARD_LED, OUTPUT);
  pinMode(12, OUTPUT);
  pinMode(14, OUTPUT);
  pinMode(27, OUTPUT);
  pinMode(26, OUTPUT);
}

void loop() {
  now = millis();

  // Enviar datos al servidor cada segundo
  if (now - timestamp > 1000) {
    timestamp = now;

    // Leer señal analógica y convertirla a un rango de 0 a 200
    valor = analogRead(34);
    valor_f = ((float)valor / 4095) * 200;
    
    // Enviar el dato al servidor
    mensaje = "\"" + String(valor_f) + "\"";
    socketIO.emit("desde_esp32", mensaje.c_str());
  }
  
  // Mantener la conexión con el servidor
  socketIO.loop();
}

// Función para conectar a la red WiFi
void conectar_WiFiSTA() {
   delay(10);
   Serial.println("\nConectando a WiFi...");
   WiFi.mode(WIFI_STA);
   WiFi.begin(ssid, password);
   
   while (WiFi.status() != WL_CONNECTED) { 
     delay(100);
     Serial.print(".");
   }
   
   Serial.println("\nConexión exitosa");
   Serial.print("Dirección IP asignada: ");
   Serial.println(WiFi.localIP());
}

// Función para procesar comandos recibidos desde el servidor
void procesar_mensaje_recibido(const char* payload, size_t length) {
  Serial.printf("Mensaje recibido: %s\n", payload);
  String paystring = String(payload);
  
  if (paystring == "ON") {
    digitalWrite(ONBOARD_LED, HIGH);
    digitalWrite(12, HIGH);
    digitalWrite(14, HIGH);
    digitalWrite(27, HIGH);
    digitalWrite(26, HIGH);
  } 
  else if (paystring == "OFF") {
    digitalWrite(ONBOARD_LED, LOW);
    digitalWrite(12, LOW);
    digitalWrite(14, LOW);
    digitalWrite(27, LOW);
    digitalWrite(26, LOW);
  }
}
```

---

## **2.4 Explicación del Código**

1. **Conexión a WiFi:**  
   - Se usa `WiFi.begin(ssid, password)` para conectar el ESP32 a la red.  
   - `WiFi.status()` verifica si la conexión se ha establecido correctamente.  
   - `WiFi.localIP()` imprime la dirección IP asignada al ESP32.  

2. **Lectura de Señal Analógica:**  
   - Se usa `analogRead(34)` para leer un **valor entre 0 y 4095** (resolución de 12 bits).  
   - Se normaliza el valor a un rango de **0 a 200** para facilitar la interpretación.  

3. **Comunicación con el Servidor:**  
   - Se establece la conexión con `socketIO.begin(server, port)`.  
   - Cada **segundo**, se envía el valor analógico con `socketIO.emit("desde_esp32", mensaje.c_str());`.  
   - `socketIO.loop()` mantiene la conexión activa.  

4. **Recepción de Comandos del Servidor:**  
   - La función `procesar_mensaje_recibido()` maneja mensajes enviados desde el servidor.  
   - Si el mensaje recibido es `"ON"`, **enciende todos los LEDs**.  
   - Si el mensaje recibido es `"OFF"`, **apaga todos los LEDs**.  

---

## **2.5 Subida del Código al ESP32**
1. **Conecta el ESP32 a la computadora mediante USB**.
2. **Selecciona la placa**: En **Arduino IDE**, ve a **Herramientas** → **Placa** → **ESP32 Dev Module**.
3. **Selecciona el puerto correcto** en **Herramientas** → **Puerto**.
4. **Carga el código** haciendo clic en **Subir**.
5. **Abre el monitor serie** (`115200 baudios`) para verificar la conexión con WiFi y el servidor.

---

## **2.6 Comprobación**
- Si todo funciona bien, en el **monitor serie** verás que el ESP32 se conecta a WiFi y envía datos al servidor.  
- En la terminal del servidor Node.js (`server.js`), deberías ver mensajes con los datos recibidos del ESP32.  
- Ahora **déjalo corriendo**, porque en el siguiente paso crearemos la interfaz web para visualizar los datos en tiempo real. 🚀

---

### **3. Creación de la Interfaz Web para Monitoreo y Control**

En este paso, desarrollaremos la **interfaz web** que permitirá visualizar en tiempo real los datos enviados desde el **ESP32** y enviar comandos al servidor para controlar dispositivos como LEDs.

---

## **3.1 La Carpeta `public` y su Función en el Servidor**

En el **paso 1**, configuramos el servidor en Node.js con la siguiente línea:

```javascript
app.use(express.static('public'));
```

Esta línea indica que todos los archivos estáticos (HTML, CSS y JavaScript) se servirán desde la carpeta **`public`**.

Por lo tanto, dentro de nuestro proyecto, debemos crear una carpeta **`public`** y colocar los siguientes archivos dentro de ella:

📁 `public/`  
┣ 📜 `index.html` (Interfaz gráfica)  
┣ 📜 `index.js` (Lógica en JavaScript)  
┗ 📜 `style.css` (Estilos)  

La estructura del código es la siguiente:

- **`index.html`**: Define la interfaz gráfica con botones, texto y una sección donde se mostrarán los datos recibidos del **ESP32**.
- **`index.js`**: Se encarga de la comunicación en **WebSockets** con el servidor y la actualización de la interfaz.
- **`style.css`**: Define los estilos visuales de los botones y elementos de la página.

---

## **3.2 Código de `index.html`**

Este archivo define la estructura visual de la interfaz web:

```html
<!DOCTYPE html>

<html>

<head>
    <title> Telecomunicaciones</title>
    <script type="text/javascript" src="socket.io/socket.io.js"></script>
    <script type="text/javascript" src="https://www.google.com/jsapi"></script>
    <script type="text/javascript" src="index.js"></script>
    <link rel="stylesheet" type="text/css" href="style.css">
</head>

<body>
    <h2>Telecomunicaciones - Monitoreo y Control</h2>
    <div id="div_dato"></div>
    <div id="div_comando"></div>
    <div id="div_grafica"></div>

    <br><br>
    <div>
        <input type="button" class="button button1" value="ENCENDER LED" onclick="encender()">
        <input type="button" class="button button1" value="APAGAR LED" onclick="apagar()">
        <br><br>
        <input type="text" id="txt_comando">
        <input type="button" class="button button1" value="ENVIAR COMANDO" onclick="enviar_comando()">
    </div>
</body>

</html>
```

### **Explicación General**
1. **Carga de bibliotecas**:
   - **`socket.io/socket.io.js`**: Permite la comunicación en tiempo real con el servidor.
   - **Google Charts (`https://www.google.com/jsapi`)**: Se usará para graficar los datos en tiempo real.
   - **`index.js`**: Script propio para manejar eventos y la comunicación con el servidor.
   - **`style.css`**: Archivo de estilos para personalizar la apariencia de la página.

2. **Secciones de la Interfaz**:
   - **`div_dato`**: Muestra el valor analógico enviado por el ESP32.
   - **`div_comando`**: Muestra el último comando enviado desde la página al ESP32.
   - **`div_grafica`**: Espacio donde se generará una gráfica en tiempo real.

3. **Botones de control**:
   - **"ENCENDER LED"**: Envía un comando al servidor para encender los LEDs del ESP32.
   - **"APAGAR LED"**: Apaga los LEDs del ESP32.
   - **Campo de texto y botón "ENVIAR COMANDO"**: Permite enviar comandos personalizados al ESP32.

---

## **3.3 Código de `index.js`**

Este archivo maneja la comunicación entre la interfaz web y el servidor.

```javascript
var socket = io.connect();
var explodedValues = [2];

socket.on('retransmision_esp32', function(data){
    var cadena = `<div> <strong> &#128269; Dato:  <font color="orange">` + data + `</strong> </div>`;
    document.getElementById("div_dato").innerHTML = cadena;
    explodedValues[1] = parseFloat(data);
    drawVisualization();
});

socket.on('desde_servidor_comando', function(data){
    var cadena = `<div> <strong> &#128187; Comando:  <font color="green">` + data + `</strong> </div>`;
    document.getElementById("div_comando").innerHTML = cadena;
    explodedValues[0] = parseFloat(data);
    drawVisualization();
});

function encender() {
    socket.emit("desde_cliente", "ON");
}

function apagar() {
    socket.emit("desde_cliente", "OFF");
}

function enviar_comando() {
    var comando = document.getElementById('txt_comando').value;
    socket.emit('desde_cliente', comando);
}

function drawVisualization() {
    var data = google.visualization.arrayToDataTable([
        ['Tracker', '1',{ role: 'style' }],
        ['Dato', explodedValues[0],'color: orange'],
        ['Otro dato', explodedValues[1],'color: blue']
    ]);

    var view = new google.visualization.DataView(data);
    view.setColumns([0, {
        type: 'number',
        label: data.getColumnLabel(1),
        calc: function () {return 0;}
    }]);

    var chart = new google.visualization.BarChart(document.getElementById('div_grafica'));

    var options = {
        title: "Monitoreo",
        width: 1200,
        height: 300,
        bar: { groupWidth: "95%" },
        legend: { position: "none" },
        animation: {
            duration: 0
        },
        hAxis: {
            minValue: 0,
            maxValue: 200
        }
    };

    var runOnce = google.visualization.events.addListener(chart, 'ready', function () {
        google.visualization.events.removeListener(runOnce);
        chart.draw(data, options);
    });

    chart.draw(view, options);
}

google.load('visualization', '1', {packages: ['corechart'], callback: drawVisualization});
```

### **Explicación General**
1. **WebSockets**:
   - Se conecta al servidor con `var socket = io.connect();`.
   - Escucha los datos del ESP32 (`socket.on('retransmision_esp32', function(data){...}`).
   - Muestra los datos en `div_dato` y los grafica.
   - Escucha los comandos del servidor (`socket.on('desde_servidor_comando', function(data){...}`).

2. **Funciones de control**:
   - **`encender()`**: Envía `"ON"` al servidor para encender los LEDs.
   - **`apagar()`**: Envía `"OFF"` para apagar los LEDs.
   - **`enviar_comando()`**: Envía el comando ingresado en la caja de texto.

3. **Gráfica en tiempo real**:
   - Utiliza Google Charts para visualizar los valores recibidos.
   - `drawVisualization()` actualiza la gráfica cada vez que se recibe un nuevo dato.

---

## **3.4 Código de `style.css`**

Este archivo define los estilos para los botones y la interfaz.

```css
/* Estilo para botones */
.button1 {
    background-color: white;
    color: black;
    border: 2px solid #4CAF50;
}

.button1:hover {
    background-color: #3e8e41;
}

.button1:active {
    background-color: #3e8e41;
    box-shadow: 0 5px #666;
    transform: translateY(4px);
}
```

### **Explicación General**
- Los botones tienen un borde verde (`#4CAF50`).
- Cambian de color cuando el usuario pasa el mouse sobre ellos (`hover`).
- Tienen un efecto de presión cuando se presionan (`active`).

---

## **3.5 Prueba Final**
1. **Asegúrate de que el servidor está corriendo** (`nodemon server.js`).
2. **Abre tu navegador y visita**:  
   ```
   http://localhost:3000/
   ```
3. **Deberían ver la interfaz web con los datos en tiempo real.** 

---


