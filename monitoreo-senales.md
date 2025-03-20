
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

---

Este es el **primer paso** de la práctica. Ahora pasaremos a configurar el **ESP32** para enviar datos al servidor. 🚀
