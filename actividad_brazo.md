### ✨ Control Distribuido de Brazo Robótico con ESP32 ✨

---

#### 🔹 Objetivo de la actividad

Desarrollar de manera colaborativa un sistema distribuido que permita controlar un brazo robótico de 5 grados de libertad utilizando un ESP32 y servomotores. El control se realizará desde distintos clientes: interfaz web, aplicación de escritorio con botones y cliente con OpenCV, todo coordinado a través de un servidor central.

Cada equipo será responsable de un bloque específico del sistema. Todos los bloques deberán integrarse para lograr el control remoto del brazo en tiempo real.

---

### 🔍 Diagrama del sistema distribuido

![Diagrama del sistema](images/diagrama.png)

---

### 🛠️ Bloques del sistema y asignación de equipos

#### Bloque 1: Control de servomotores en ESP32

**Responsables:**

* ALMAZAN PACHECO MARIA FERNANDA
* ANTUNEZ RUBIO JOVITA MARIA FERNANDA
* CAMACHO LOPEZ VALERIA
* RAMIREZ RODRIGUEZ CARLOS AZAEL
* SILVA PEREZ KARLA EMILIA

**Actividades:**

* Conectar los 5 servomotores al ESP32.
* Programar el ESP32 para recibir órdenes por WiFi y mover los servos correctamente.

---

#### Bloque 2: Comunicación WiFi entre ESP32 y servidor

**Responsables:**

* ARAUJO RODRIGUEZ ARMANDO
* RAMIREZ BARAJAS PORFIRIO EMMANUEL
* RODRIGUEZ PADILLA KEVIN
* VIERA ISABELES MISAEL MOISES

**Actividades:**

* Establecer conexión por WebSocket o HTTP desde ESP32.
* Enviar y recibir datos entre ESP32 y el servidor.

---

#### Bloque 3: Servidor central Node.js

**Responsables:**

* ARELLANO SALONES BRICIA IDALIA
* CERVANTES ARMENTA ARON
* LOPEZ PARDO BRIANNA VANESSA
* RAMIREZ ZEPEDA ALEJANDRO
* VILLA CAMACHO ABIGAIL

**Actividades:**

* Crear servidor Node.js que reciba comandos de los clientes.
* Enviar los comandos al ESP32.

---

#### Bloque 4: Interfaz Web (control desde navegador)

**Responsables:**

* AVILA SANCHEZ GERMAN EMMANUEL
* FONSECA DIAZ IVAN DE JESUS
* GARCIA MONTILLA MARCO ANTONIO
* RAMIREZ RUIZ JOSE DAVID
* RIVERA DURAN VANIA DANIELA

**Actividades:**

* Diseñar una interfaz HTML + JS con botones para mover el brazo.
* Enviar los comandos al servidor mediante AJAX o WebSocket.

---

#### Bloque 5: Cliente OpenCV (control por visión artificial)

**Responsables:**

* AYON MADRID MIGUEL ANGEL
* LOPEZ LOPEZ SANTIAGO
* LOPEZ REYES ANTONIO

**Actividades:**

* Detectar movimientos de la mano o una señal visual con OpenCV.
* Traducirlo a comandos para el brazo.
* Enviar al servidor.

---

#### Bloque 6: Cliente Python con GUI de botones

**Responsables:**

* CAMACHO QUEZADA NICOLE ZOE
* DE LA TORRE GOMEZ JOHANA JAZMIN
* GARCIA MURILLO KARLA VALERIA
* PUENTES HERNANDEZ YOSELINE LIZETH
* URBINA GOMEZ ELIZA YASUNARI

**Actividades:**

* Crear una interfaz de escritorio con botones.
* Cada botón debe enviar una acción al servidor para mover el brazo.

---

#### Bloque 7: Seguridad y validación de comandos

**Responsables:**

* CASTILLO GONZALEZ DANIELA
* GARCIA SANCHEZ LUIS FABIAN
* IBARRA SALAS BRAULIO ALEJANDRO

**Actividades:**

* Validar que los comandos sean válidos.
* Implementar un sistema simple de autenticación.

---

#### Bloque 8: Integración y prueba del sistema

**Responsables:**

* CASTRO CASTRO CLAUDIA XIMENA
* CHIZEK ESPINOZA JOSUE
* ESTRADA CASTILLO IAN ENRIQUE
* LOPEZ ISLAS DIEGO SAUL

**Actividades:**

* Coordinar la integración de todos los bloques.
* Realizar pruebas en tiempo real con los 3 clientes.
* Verificar la comunicación completa del sistema.

---

#### Bloque 9: Supervisión y evaluación en vivo

**Responsables:**

* DURAN MUÑOZ SOFIA CRISTINA
* GALINDO RAMIREZ MARIA BELEN
* SIFUENTES CASTRO ANADALI
* VALLE Z. FLORES ANA KAMILA

**Actividades:**

* Evaluar el desempeño de cada bloque.
* Documentar errores, sugerencias y mejoras.

---

#### Bloque 10: Coordinación general y documentación

**Responsables:**

* HERRERA AGUILAR RAFAEL
* LOPEZ LOPEZ MARIA CRISTINA
* SOLIS MARRUFO XIMENA
* VIDAL ORTIZ ANETTE MARIANA
* VILLALOBOS VALDEZ ALBERTO

**Actividades:**

* Verificar que todos los equipos cumplan con sus funciones.
* Elaborar un breve resumen del sistema y de su funcionamiento.

---

### 🔄 Producto final esperado

Un sistema funcional en el que:

* El brazo robótico se pueda controlar desde tres clientes diferentes (Web, GUI Python y OpenCV).
* El servidor central reciba y distribuya correctamente los comandos.
* La comunicación sea confiable.

---

### ✅ Recomendaciones

* Cada equipo debe probar su parte de forma local antes de integrar.
* Compartir interfaces claras entre bloques (por ejemplo: JSON con nombre de servo y ángulo).
* Documentar todo lo que se construya.

---


