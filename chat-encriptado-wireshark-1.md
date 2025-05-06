# 🌐 **Práctica: Comunicación Cliente-Servidor y Análisis de Paquetes con Wireshark**

## Introducción teórica

---

### 🔶 1️⃣ **Modelos OSI y TCP/IP**

#### 📊 **¿Qué es el modelo OSI?**

El **Modelo OSI (Open Systems Interconnection)** es un marco de referencia que divide la comunicación en redes en **7 capas**, cada una con funciones específicas. Aunque hoy en día no se usa de forma estricta, sigue siendo la **base conceptual** para entender cómo se comunican los sistemas.

| Capa | Nombre          | Función                                                                              |
| ---- | --------------- | ------------------------------------------------------------------------------------ |
| 7    | Aplicación      | Proporciona servicios directamente a las aplicaciones del usuario (HTTP, FTP, etc.). |
| 6    | Presentación    | Traduce datos entre la aplicación y la red (codificación, cifrado).                  |
| 5    | Sesión          | Controla las sesiones (conexiones) entre computadoras.                               |
| 4    | Transporte      | Asegura la entrega de datos de extremo a extremo (TCP, UDP).                         |
| 3    | Red             | Determina la ruta y gestiona la entrega de paquetes (IP).                            |
| 2    | Enlace de datos | Organiza los bits en tramas y maneja errores físicos (Ethernet, Wi-Fi).              |
| 1    | Física          | Transmite bits a través del medio físico (cables, señales de radio).                 |

---

#### 🖥️ **¿Qué es el modelo TCP/IP?**

El modelo **TCP/IP** es la base real de Internet. Tiene 4 capas principales:

| Capa TCP/IP     | Equivalente OSI | Protocolos clave      |
| --------------- | --------------- | --------------------- |
| Aplicación      | OSI 5, 6 y 7    | HTTP, FTP, SMTP, etc. |
| Transporte      | OSI 4           | TCP, UDP              |
| Internet        | OSI 3           | IP, ICMP              |
| Acceso a la red | OSI 1 y 2       | Ethernet, Wi-Fi, PPP  |

🔎 **Relación**: Wireshark te permitirá ver estas capas claramente y analizar cómo se encapsulan los mensajes.

---

---

### 🔶 2️⃣ **¿Qué son los protocolos?**

Un **protocolo** es un conjunto de reglas que define cómo se comunican dos dispositivos en una red. Ejemplos:

* **TCP (Transmission Control Protocol)**: Se asegura de que los datos lleguen completos y en orden.
* **IP (Internet Protocol)**: Encamina los paquetes entre redes.
* **HTTP**: Protocolo para páginas web.
* **Ethernet**: Protocolo de capa de enlace para redes LAN.

Cada protocolo define cómo debe formarse un **mensaje**, cómo interpretarlo y cómo manejar errores.

---

---

### 🔶 3️⃣ **Encabezados (Headers) y Carga útil (Payload)**

Cada vez que enviamos datos por la red, estos se **encapsulan**:

* **Encabezado (Header):** Información sobre el paquete (dirección IP de origen/destino, puertos, número de secuencia, etc.).
* **Payload (Carga útil):** El contenido real que queremos enviar (por ejemplo, tu mensaje: `"Hola mundo"`).

**Ejemplo visual:**

```
+---------------------------+
| Ethernet Header           |
+---------------------------+
| IP Header                 |
+---------------------------+
| TCP Header                |
+---------------------------+
| Mensaje (Payload)         |
+---------------------------+
```

👉 **Importante:** Aunque cifres el *mensaje*, los encabezados siguen visibles porque son necesarios para que el paquete llegue a destino.

---

---

### 🔶 4️⃣ **Wireshark: Herramienta de análisis**

**Wireshark** es un programa para **capturar y analizar** paquetes de red. Podrás:

* Ver el **header y payload** de cada capa.
* Usar **filtros** para buscar mensajes específicos.
* Guardar capturas para su análisis posterior.

💡 **Dato clave:** Como nuestro chat en Java usa **TCP**, podremos ver los datos fácilmente en Wireshark.

---

---

### 🔶 5️⃣ **Encriptación**

👉 **Encriptar** significa transformar un mensaje en algo **ilegible** para protegerlo durante la transmisión.

**Ejemplo:**

* Sin encriptar: `Usuario: Hola mundo`
* Encriptado (AES/Base64): `VXN1YXJpbzogSG9sYSBtdW5kbw==`

🔐 **Claves:**

* **Encriptación simétrica:** Usa la misma clave para cifrar y descifrar.
* **Encriptación asimétrica:** Usa claves pública y privada (más compleja).


---

---

## ✅ **Resumen:**

1. Ejecutar y probar un sistema de **cliente-servidor en Java**.
2. Capturar los mensajes enviados **sin encriptar** usando Wireshark.
3. Analizar las **capas, headers y payloads** de los paquetes.
4. Modificar el código para encriptar los mensajes.
5. Capturar los mensajes nuevamente para verificar que ya no son legibles.
6. Relacionar los resultados con los **modelos OSI y TCP/IP**.

---

---

---

# 🔧 **PARTE 1: Configuración y ejecución del Cliente y Servidor**

---

### 🎯 **Objetivo:**

Que los alumnos comprendan cómo compilar, ejecutar y probar el servidor y al menos dos clientes que se comunican entre sí **sin encriptación**.

---

## 🚩 **Paso 1: Configuración del entorno**

### 🔹 **Requisitos:**

* Java Development Kit (JDK) instalado.
* Wireshark instalado (para más adelante).

✅ **Comando para comprobar Java:**

```bash
java -version
javac -version
```

Debe mostrar la versión instalada.

---

## 🚩 **Paso 2: Preparar los archivos**

### 📁 **Estructura recomendada:**

```
/practica_redes/
├── MyServer.java
├── UI_Cliente.java
```

➡️ Guarda el código del **servidor** en `MyServer.java` y el del **cliente** en `UI_Cliente.java`.

---

## 🚩 **Paso 3: Compilar**

### 1️⃣ Compilar el servidor:

```bash
javac MyServer.java
```

Esto genera el archivo `MyServer.class`.

### 2️⃣ Compilar el cliente:

```bash
javac UI_Cliente.java
```

Esto genera varios `.class` porque hay varias clases en un archivo.

---

## 🚩 **Paso 4: Ejecutar**

### 1️⃣ Iniciar el servidor:

```bash
java MyServer
```

💡 **Nota:** Debe iniciar y quedarse esperando conexiones en el puerto `5001`.

### 2️⃣ Iniciar cada cliente (en diferentes terminales o computadoras):

```bash
java UI_Cliente
```

🚩 **Conexión:**

* Introduce:

  * **Usuario:** (por ejemplo: `Juan`)
  * **IP destino:** IP donde está corriendo el servidor (puedes usar `localhost` si está en la misma PC).
  * **Puerto:** `5001`

🔄 Conecta dos clientes y envía mensajes para verificar que:

* Ambos reciben los mensajes.
* El servidor los reenvía correctamente (ver la consola del servidor).

---

## 🚩 **Paso 5: Captura de pantalla y bitácora**

✅ Toma capturas de:

* El servidor corriendo.
* Dos clientes enviando y recibiendo mensajes.

📒 **Bitácora:**

* Anota:

  * Qué ocurrió cuando un cliente se conectó/desconectó.
  * Qué mensajes viste en la consola del servidor.

---

---

# 🌐 **PARTE 2: Captura del tráfico con Wireshark**

---

## 🎯 **Objetivo:**

* Capturar los paquetes enviados entre el cliente y el servidor usando Wireshark.
* Identificar claramente que los mensajes **no están encriptados**: el payload se puede leer directamente.
* Familiarizarse con la interfaz de Wireshark y aplicar filtros básicos.

---

---

## 🚩 **Paso 1: ¿Qué es Wireshark y para qué lo usaremos?**

**Wireshark** es una herramienta que permite **ver en tiempo real** todo el tráfico que pasa por la tarjeta de red de la computadora. Cada mensaje enviado o recibido **se convierte en un paquete de red**, que se puede analizar en detalle.

✅ **En esta práctica:**

* El cliente manda un mensaje → **Wireshark lo captura.**
* El servidor lo reenvía a los demás → **Wireshark también lo captura.**

Vamos a **ver el mensaje completo**, incluyendo los headers y el payload en **texto plano**.

---

---

## 🚩 **Paso 2: Preparar Wireshark**

### 🔧 **1️⃣ Abrir Wireshark**

* Busca **Wireshark** en el menú de tu computadora y ábrelo.

### 🔧 **2️⃣ Elegir la interfaz correcta**

Cuando inicies Wireshark, verás algo como esto:

```
Lista de interfaces disponibles:
- Ethernet
- Wi-Fi
- Loopback (localhost)
```

💡 **Consejo:**

* Si estás trabajando en **red cableada**, selecciona **Ethernet.**
* Si estás usando **Wi-Fi**, selecciona **Wi-Fi.**
* Si el servidor y cliente están en la **misma PC**, puedes usar la interfaz **Loopback** (suele llamarse `lo` o `Loopback`).

👉 **NOTA IMPORTANTE:**
Para ver tráfico **local (localhost/127.0.0.1)** en Windows, **NO aparece en Wireshark** directamente, a menos que tengas algo como `Npcap` configurado para Loopback. Por eso es mejor trabajar con **direcciones reales de red** (aunque sea en la misma máquina usando la IP local).

---

---

## 🚩 **Paso 3: Iniciar la captura**

1️⃣ Haz doble clic en la interfaz seleccionada para comenzar la captura.

2️⃣ En cuanto empiece la captura, verás una **lista de paquetes** corriendo rápidamente.

3️⃣ **¡Ahora ejecuta el sistema!**

* Asegúrate de que el servidor ya está corriendo.
* Conecta dos clientes y envía algunos mensajes.

---

---

## 🚩 **Paso 4: Aplicar un filtro para enfocarte en tu tráfico**

Sin filtro, hay **muchísimo ruido** (otros paquetes de la red).

### ✅ Usa este filtro para enfocarte:

```
tcp.port == 5001
```

Esto te mostrará **solo los paquetes** que usan el puerto 5001 (el puerto que usa tu servidor y tus clientes).

🔍 **¿Dónde aplicarlo?**

* En la barra de filtro arriba de la lista de paquetes.
* Escribe el filtro y presiona ENTER.

➡️ Ahora verás **solo los mensajes relevantes.**

---

---

## 🚩 **Paso 5: Buscar el mensaje dentro del paquete**

1️⃣ Haz clic en uno de los paquetes que aparezca tras enviar un mensaje.

2️⃣ Wireshark mostrará algo así:

```
- Ethernet II
- Internet Protocol Version 4
- Transmission Control Protocol
- Data (Payload)
```

3️⃣ Despliega la sección **Data (Payload).**

Aquí deberías ver tu mensaje en **texto plano.**

### ✅ **Ejemplo:**

Imagina que enviaste este mensaje:

```
Juan: Hola mundo
```

En la sección **Data** verás algo como esto:

```
4A 75 61 6E 3A 20 48 6F 6C 61 20 6D 75 6E 64 6F
Juan: Hola mundo
```

La parte de la derecha muestra el texto **directamente**.

---

---

## 🚩 **Paso 6: Detener y guardar la captura**

1️⃣ Haz clic en el **ícono rojo** de detener captura.

2️⃣ Guarda la captura para tu reporte:

* Archivo → Guardar como...
* Nombra el archivo, por ejemplo: `captura_mensajes_sin_cifrar.pcapng`

Esto servirá para el análisis en la siguiente parte.

---

---

## 🚩 **Paso 7: Problemas comunes y cómo resolverlos**

| Problema                                              | Solución                                                                                                            |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| No veo tráfico en Wireshark                           | Asegúrate de que elegiste la **interfaz correcta**. Usa Wi-Fi o Ethernet según estés conectado.                     |
| No veo los mensajes en texto plano                    | Verifica que los mensajes realmente fueron enviados durante la captura. Repite el envío mientras Wireshark captura. |
| No aparece tráfico con `tcp.port == 5001`             | ¿Seguro que usas ese puerto? Si cambiaste el puerto en tu código, ajusta el filtro al puerto real.                  |
| Mi firewall bloquea la conexión                       | Permite las conexiones en el firewall o desactívalo temporalmente para la práctica.                                 |
| No captura tráfico `localhost` (127.0.0.1) en Windows | Usa la IP local real (por ejemplo 192.168.x.x) para ver el tráfico en la red real.                                  |

---

---

## 🚩 **Bitácora y entregables para esta parte**

✅ En tu bitácora debes incluir:

1️⃣ Capturas de pantalla:

* Wireshark mostrando los paquetes capturados.
* Un paquete expandido mostrando el **mensaje completo en texto plano.**

2️⃣ Un texto donde expliques:

* ¿Qué interfaz elegiste en Wireshark?
* ¿Qué filtro aplicaste?
* Describe brevemente **cómo lograste encontrar el mensaje** dentro de la captura.

3️⃣ Una reflexión inicial:

* ¿Te sorprendió ver el mensaje en texto plano?
* ¿Qué riesgos implica esto en un sistema real?

---

---

# 🔍 **Resultado esperado:**

* Alumnos viendo claramente que **cualquier persona en la red** puede capturar y leer los mensajes enviados, porque no hay cifrado.
* Captura guardada lista para comparar más adelante con la versión cifrada.

---


# 🌐 **PARTE 3: Análisis del tráfico capturado (Modelo OSI y TCP/IP)**

---

## 🎯 **Objetivo:**

* Analizar en detalle los paquetes capturados con Wireshark.
* Relacionar cada **capa del paquete** con el modelo OSI y el modelo TCP/IP.
* Comprender cada **header** y **campo** de los protocolos involucrados.
* Reconocer dónde está la **carga útil (payload)** y qué información queda **expuesta.**

---

---

## 🚩 **Paso 1: Relacionar los modelos OSI y TCP/IP con la captura**

Antes de analizar un paquete, recordemos cómo se relacionan los modelos **OSI y TCP/IP** con lo que vamos a ver en Wireshark:

| Capa OSI     | Capa TCP/IP  | ¿Se ve en Wireshark? | Ejemplo en esta práctica                                 |
| ------------ | ------------ | -------------------- | -------------------------------------------------------- |
| 7 Aplicación | Aplicación   | ✅ Parcial (payload)  | El mensaje: `Juan: Hola mundo`                           |
| 4 Transporte | Transporte   | ✅ Sí                 | Protocolo TCP: puertos, flags, secuencia                 |
| 3 Red        | Internet     | ✅ Sí                 | Protocolo IP: direcciones IP de origen y destino         |
| 2 Enlace     | Acceso a red | ✅ Sí                 | Ethernet: MAC origen/destino                             |
| 1 Física     | -            | ❌ No (bits/voltaje)  | Solo vemos las capas superiores; esta capa no es visible |

👉 **Wireshark muestra de la Capa 2 hacia arriba.**

---

---

## 🚩 **Paso 2: Seleccionar un paquete para analizar**

🔍 Selecciona **un paquete clave** en tu captura:

* Debe ser **uno que contenga el mensaje enviado** (lo verás porque en la sección **Data** está tu mensaje en texto plano).

Ejemplo: elegiste un paquete donde el mensaje fue:

```
Juan: Hola mundo
```

---

---

## 🚩 **Paso 3: Analizar la capa de enlace de datos (Capa 2 OSI)**

### 📦 **Encabezado Ethernet II**

🔎 Expande la sección **Ethernet II** en Wireshark. Verás algo como:

* **Destination:** `ff:ff:ff:ff:ff:ff` (si es broadcast) o la MAC destino.
* **Source:** La MAC del cliente que envió el paquete.
* **Type:** `IPv4` (0x0800).

✏️ **Anota:**

* ¿Qué MAC está enviando y cuál está recibiendo?
* ¿Qué tipo de protocolo está encapsulado? (en este caso IPv4)

💡 **Relación OSI:**

* Esta capa es la **Capa 2 (Enlace de datos)**.
* En TCP/IP es parte de **Acceso a la red.**

---

---

## 🚩 **Paso 4: Analizar la capa de red (Capa 3 OSI)**

### 🌐 **Encabezado IP (Internet Protocol)**

🔎 Expande la sección **Internet Protocol Version 4 (IPv4)**. Verás:

* **Source IP:** IP de quien envió el mensaje.
* **Destination IP:** IP de quien recibe el mensaje.
* **Version:** IPv4.
* **Header Length:** Tamaño del encabezado.
* **Total Length:** Tamaño total del paquete (header + data).
* **Protocol:** TCP (6).

✏️ **Anota:**

* IP de origen y destino.
* Tamaño del paquete.
* ¿Es IPv4 o IPv6?
* ¿Qué protocolo encapsula? (TCP)

💡 **Relación OSI:**

* Esta es la **Capa 3 (Red)** en OSI.
* En TCP/IP es la **capa Internet.**

---

---

## 🚩 **Paso 5: Analizar la capa de transporte (Capa 4 OSI)**

### 🚚 **Encabezado TCP (Transmission Control Protocol)**

🔎 Expande la sección **Transmission Control Protocol**. Verás:

* **Source Port:** Puerto de origen (aleatorio, algo como 50642).
* **Destination Port:** 5001 (puerto del servidor).
* **Sequence number:** Número de secuencia.
* **Acknowledgment number:** (si aplica).
* **Flags:** Qué está pasando (SYN, ACK, PSH, etc.).
* **Window size:** Control de flujo.

✏️ **Anota:**

* Puerto de origen y destino.
* ¿Qué flags están activos?
* ¿Este paquete es parte de la apertura de la conexión (SYN)? ¿O está enviando datos (PSH + ACK)?

💡 **Relación OSI:**

* Esta es la **Capa 4 (Transporte)** en OSI.
* En TCP/IP es la **capa Transporte.**

---

---

## 🚩 **Paso 6: Analizar la capa de aplicación (Capa 7 OSI)**

### 💬 **Payload (Data)**

🔎 Baja hasta la sección **Data (Payload)**.

Aquí está el mensaje en **texto plano**, algo así como:

```
Juan: Hola mundo
```

Verás los bytes y su representación en ASCII.

✏️ **Anota:**

* ¿Cuál fue el mensaje exacto?
* ¿Se ve claramente legible?

💡 **Relación OSI:**

* Esta es la **Capa 7 (Aplicación)** en OSI.
* En TCP/IP es parte de la **capa Aplicación.**

---

---

## 🚩 **Paso 7: Relacionar todo con los modelos OSI y TCP/IP**

| Capa OSI          | Capa TCP/IP  | Elementos en el paquete (lo que viste)              |
| ----------------- | ------------ | --------------------------------------------------- |
| 7 Aplicación      | Aplicación   | El mensaje: `Juan: Hola mundo`                      |
| 6 Presentación    | -            | (No usamos cifrado ni codificación todavía)         |
| 5 Sesión          | -            | (No manejamos sesiones explícitas en esta práctica) |
| 4 Transporte      | Transporte   | TCP: puertos, flags, secuencias, ventana, checksum  |
| 3 Red             | Internet     | IP: IP origen/destino, TTL, protocolo, checksum     |
| 2 Enlace de datos | Acceso a red | Ethernet: MAC origen/destino, tipo (IPv4)           |
| 1 Física          | -            | (No visible en Wireshark: solo bits/voltajes)       |

---

---

## 🚩 **Ejemplo práctico de análisis (para la bitácora)**

**Paquete capturado:**

* **Ethernet II:**

  * Source: `e4:ce:8f:23:6b:98`
  * Destination: `e4:ce:8f:aa:bb:cc`
  * Type: IPv4

* **IPv4:**

  * Source IP: `192.168.1.101`
  * Destination IP: `192.168.1.102`
  * Total Length: 74

* **TCP:**

  * Source Port: 50642
  * Destination Port: 5001
  * Flags: PSH, ACK
  * Sequence: 1

* **Data:**

  * Payload: `Juan: Hola mundo`

---

---

## 🚩 **Bitácora y entregables para esta parte**

✅ **Capturas de pantalla:**

* Expande y captura cada parte:

  * Ethernet.
  * IP.
  * TCP.
  * Data.

✅ **Análisis escrito:**
1️⃣ **Capa por capa:**

* ¿Qué viste en Ethernet?
* ¿Qué viste en IP?
* ¿Qué viste en TCP?
* ¿Qué viste en Data?

2️⃣ **Comparación modelos:**

* Relaciona tus hallazgos con OSI y TCP/IP.

3️⃣ **Reflexión:**

* ¿Por qué crees que los encabezados (headers) NO están cifrados?
* ¿Qué pasaría si alguien en la red está espiando el tráfico?

---

---

# ✅ **Resultado esperado:**

* Alumnos entienden **exactamente qué contiene cada capa**.
* Visualizan que **solo la Capa 7 (Aplicación)** lleva la data que ellos enviaron.
* Comprenden que **los encabezados son necesarios para el transporte** y no se cifran por defecto.

---

---

¡Perfecto! 🙌 Ahora viene la parte **más interesante**: los alumnos van a **modificar el código** para que los mensajes **se encripten** antes de ser enviados y se **desencripten** al recibirlos. Esto les permitirá comparar después en Wireshark cómo el **payload ya no es legible**.

---

# 🔐 **PARTE 4: Modificar el cliente para encriptar los mensajes**

---

## 🎯 **Objetivo:**

* Añadir **encriptación simple** al código del cliente (en la parte de envío).
* Añadir la **desencriptación** al recibir mensajes.
* Mantener el servidor **sin modificaciones** (solo reenvía los datos).

---

---

## 🚩 **Paso 1: ¿Qué método de encriptación usaremos?**

✅ Para mantenerlo **simple y didáctico**, usaremos **Base64**, que:

* Codifica los datos en un formato que **no es legible fácilmente** pero sigue siendo reversible.
* **NO es seguridad real**, pero sirve para **demostrar cómo cambia el payload**.

(⚠️ Podrías pedir después a los alumnos más avanzados que implementen algo más robusto como AES).

---

---

## 🚩 **Paso 2: Importar la librería necesaria**

Java incluye soporte para Base64 desde Java 8.

En la parte superior de `UI_Cliente.java`, añade:

```java
import java.util.Base64;
```

---

---

## 🚩 **Paso 3: Modificar la función `enviar()`**

### 🔧 **Código original:**

```java
public void enviar(){
    try {
        mensaje = txtMensaje.getText();
        out.println(usuario + ": " + mensaje);
        txtMensaje.setText("");
        txtMensaje.requestFocusInWindow();
    } catch (Exception err) {
        err.printStackTrace();
    }
}
```

---

### 🔐 **Nuevo código con encriptación Base64:**

```java
public void enviar(){
    try {
        mensaje = txtMensaje.getText();
        String completo = usuario + ": " + mensaje;

        // Encriptar usando Base64
        String mensajeEncriptado = Base64.getEncoder().encodeToString(completo.getBytes());

        out.println(mensajeEncriptado);
        txtMensaje.setText("");
        txtMensaje.requestFocusInWindow();
    } catch (Exception err) {
        err.printStackTrace();
    }
}
```

---

---

## 🚩 **Paso 4: Modificar la clase `Monitor` para desencriptar**

### 🔧 **Código original:**

```java
while((msg = in.readLine()) != null){
    System.out.println(msg);
    textArea.setText(textArea.getText() + "\n" + msg);
}
```

---

### 🔐 **Nuevo código con desencriptación Base64:**

```java
while((msg = in.readLine()) != null){
    try {
        // Desencriptar el mensaje recibido
        byte[] decodedBytes = Base64.getDecoder().decode(msg);
        String mensajeDesencriptado = new String(decodedBytes);

        System.out.println(mensajeDesencriptado);
        textArea.setText(textArea.getText() + "\n" + mensajeDesencriptado);
    } catch (IllegalArgumentException e) {
        // Si no es Base64 válido, solo lo muestra tal cual
        textArea.setText(textArea.getText() + "\n(Mensaje no encriptado): " + msg);
    }
}
```

---

---

## 🚩 **Paso 5: Probar el sistema**

1️⃣ **Compilar de nuevo:**

```bash
javac UI_Cliente.java
```

2️⃣ **Ejecutar:**

```bash
java UI_Cliente
```

✅ Ahora cuando envíes un mensaje:

```
Juan: Hola mundo
```

* En la consola del servidor:
  Se verá **encriptado**, algo así como:

```
Sm9hbjogSG9sYSBtdW5kbw==
```

* En el cliente **receptor**:
  El mensaje aparecerá **normal** porque se desencripta:

```
Juan: Hola mundo
```

---

---

## 🚩 **Paso 6: Verificar en Wireshark**

1️⃣ Haz la misma captura que en la Parte 2.
2️⃣ Aplica el filtro: `tcp.port == 5001`.
3️⃣ Selecciona un paquete y mira la sección **Data (Payload)**.

### ✅ **Antes (sin cifrar):**

```
Juan: Hola mundo
```

### ✅ **Ahora (encriptado):**

```
Sm9hbjogSG9sYSBtdW5kbw==
```

💡 **Nota:** Aunque Base64 **no es seguridad real**, ya **no puedes leer el mensaje directamente** sin decodificarlo, y esto se refleja claramente en Wireshark.

---

---

## 🚩 **Refinamiento (opcional): Diferenciar mensajes no encriptados**

El código nuevo en `Monitor` **detecta** si un mensaje no es válido Base64 (por ejemplo, si alguien usa la versión vieja del cliente) y lo muestra así:

```
(Mensaje no encriptado): Hola mundo
```

Esto ayuda a hacer pruebas mixtas fácilmente.

---

---

## 🚩 **Bitácora y entregables para esta parte**

✅ **Capturas de pantalla:**

* Wireshark mostrando un paquete con el mensaje **encriptado**.
* Cliente mostrando los mensajes ya **desencriptados**.

✅ **Código modificado:**

* Incluye las secciones de código que cambiaste (envío y recepción).

✅ **Análisis escrito:**
1️⃣ ¿Cómo cambió el contenido del paquete después de aplicar la encriptación?
2️⃣ ¿El servidor necesitó cambios?
3️⃣ ¿Qué aprendiste sobre la importancia de cifrar mensajes?

---

---

# ✅ **Resultado esperado:**

* Alumnos ven en Wireshark que **el payload ya no es legible.**
* El sistema sigue funcionando igual para los usuarios (porque el cliente desencripta).
* Confirmación de que **el servidor actúa solo como "reenvío ciego"** y **no necesita desencriptar nada.**

---

---

👉 **¿Pasamos ahora a la Parte 5 (nueva captura y comparación + análisis final), o quieres que prepare algún material extra como sugerencias para migrar a un cifrado más fuerte (AES, etc.) o detalles gráficos del flujo encriptado?**






