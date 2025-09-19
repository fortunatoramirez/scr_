
## Práctica – Protocolos y visión con OpenCV + ESP32

**Objetivo general:** Capturar imágenes, detectar un objeto, obtener su centro, y **comunicar** esos datos entre la PC (Python/OpenCV) y el **ESP32** por **UDP**, mostrando la información en la **OLED 128×64**.

---

## Requisitos previos

* **VS Code** y **Python 3.x** instalados (con `pip` en PATH).
* Paquetes Python: `opencv-python`, `numpy`.
* **ESP32** con soporte en Arduino IDE (o PlatformIO).
* OLED **128×64** I²C (dirección típica `0x3C`).
* Librerías Arduino: **Adafruit SSD1306**, **Adafruit GFX**.
* Red WiFi común para PC y ESP32.
* Realizar cualquier ajuste que considere necesario en cualquiera de las etapas.

> Nota Mac: si la cámara no abre con índice `0`, usen `cv.CAP_AVFOUNDATION`.

---

# Estructura de la práctica (4 etapas)

## Etapa 1 — Detección de objetos y coordenada central (PC con OpenCV)

### Objetivo

Detectar un **cuadrado más grande** en la imagen de la cámara, dibujar su contorno y calcular la **coordenada central** (centroide). Mostrar las coordenadas en pantalla.

### Pasos

1. Crear y activar (opcional) un entorno virtual.
2. Instalar OpenCV: `pip install opencv-python`
3. Crear `etapa1_cuadrado.py` con el siguiente código base.

```python
# etapa1_cuadrado.py
import cv2 as cv
import numpy as np

def encontrar_cuadrado_mayor(frame):
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    blur = cv.GaussianBlur(gray, (5,5), 0)
    edges = cv.Canny(blur, 60, 150)
    contours, _ = cv.findContours(edges, cv.RETR_EXTERNAL, cv.CHAIN_APPROX_SIMPLE)

    best = None
    best_area = 0
    for c in contours:
        peri = cv.arcLength(c, True)
        approx = cv.approxPolyDP(c, 0.02*peri, True)
        if len(approx) == 4 and cv.isContourConvex(approx):
            area = cv.contourArea(approx)
            if area > 1000:
                x,y,w,h = cv.boundingRect(approx)
                ratio = w / float(h)
                if 0.7 < ratio < 1.3:  # "cuadrado" relajado
                    if area > best_area:
                        best_area = area
                        best = approx
    return best

def main():
    cap = cv.VideoCapture(0)  # En Mac: cv.VideoCapture(0, cv.CAP_AVFOUNDATION)
    if not cap.isOpened():
        print("No se pudo abrir la cámara"); return

    while True:
        ok, frame = cap.read()
        if not ok: break
        H, W = frame.shape[:2]

        sq = encontrar_cuadrado_mayor(frame)
        if sq is not None:
            cv.drawContours(frame, [sq], -1, (180, 0, 255), 2)
            M = cv.moments(sq)
            if M["m00"] > 1e-3:
                cx = int(M["m10"]/M["m00"])
                cy = int(M["m01"]/M["m00"])
                cv.drawMarker(frame, (cx, cy), (0,255,0), cv.MARKER_CROSS, 18, 2)
                cv.putText(frame, f"Centro: ({cx},{cy})", (10, 30),
                           cv.FONT_HERSHEY_SIMPLEX, 0.8, (255,255,255), 2)
        else:
            cv.putText(frame, "No se detecta cuadrado", (10, 30),
                       cv.FONT_HERSHEY_SIMPLEX, 0.8, (0,0,255), 2)

        cv.putText(frame, "q: salir", (10, 60), cv.FONT_HERSHEY_SIMPLEX, 0.7, (255,255,255), 2)
        cv.imshow("Etapa 1 - Deteccion", frame)
        if cv.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv.destroyAllWindows()

if __name__ == "__main__":
    main()
```

### Verificación

* El video se muestra en tiempo real.
* Cuando coloques un **cuadrado** (hoja con cuadro, caja, un QR), se dibuja el **contorno** y aparece el **centro**.

---

## Etapa 2 — ESP32 listo: WiFi + OLED (escaneo y estado)

### Objetivo

Conectar el **ESP32** a tu WiFi, **escanear** redes cercanas y mostrar en la **OLED**: top redes detectadas, IP y RSSI (potencia) de la red conectada.

### Conexiones I²C

* SDA → **GPIO 21**
* SCL → **GPIO 22**
* VCC → **3.3V**
* GND → **GND**

### Código (Arduino) — `Etapa2_ESP32_OLED_WiFi.ino`

> Cambia `WIFI_SSID` y `WIFI_PASS`.

```cpp
#include <WiFi.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
#define OLED_ADDR     0x3C

const char* WIFI_SSID = "TU_SSID";
const char* WIFI_PASS = "TU_PASSWORD";

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

void oledLines(const String& a="", const String& b="", const String& c="", const String& d="") {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  if(a.length()) display.println(a);
  if(b.length()) display.println(b);
  if(c.length()) display.println(c);
  if(d.length()) display.println(d);
  display.display();
}

void scanAndShowTop() {
  oledLines("Escaneando WiFi...");
  int n = WiFi.scanNetworks(false, true);
  if (n <= 0) { oledLines("Sin redes"); return; }

  struct Item { String s; int r, ch; };
  std::vector<Item> v;
  for (int i=0; i<n; ++i) v.push_back({WiFi.SSID(i), WiFi.RSSI(i), WiFi.channel(i)});
  std::sort(v.begin(), v.end(), [](const Item& A, const Item& B){return A.r > B.r;});

  String l1 = v.size()>0 ? (v[0].s.substring(0,12) + " ("+String(v[0].r)+"dBm)") : "";
  String l2 = v.size()>1 ? (v[1].s.substring(0,12) + " ("+String(v[1].r)+"dBm)") : "";
  String l3 = v.size()>2 ? (v[2].s.substring(0,12) + " ("+String(v[2].r)+"dBm)") : "";
  oledLines("Top redes:", l1, l2, l3);
  delay(3500);
}

void setup() {
  Serial.begin(115200);
  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    Serial.println("OLED no encontrada"); while(true) delay(500);
  }
  oledLines("OLED OK", "Inicializando...");

  WiFi.mode(WIFI_STA);
  scanAndShowTop();

  oledLines("Conectando:", WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  uint32_t t0 = millis();
  while (WiFi.status() != WL_CONNECTED) {
    delay(250);
    if (millis() - t0 > 15000) {
      oledLines("Fallo WiFi", "Revisa SSID/PASS");
      while(true) delay(1000);
    }
  }

  oledLines("WiFi: CONECTADO", "IP: " + WiFi.localIP().toString(),
            "RSSI: " + String(WiFi.RSSI()) + " dBm", "Listo Etapa 3");
}

void loop() {
  static uint32_t last=0;
  if (millis()-last > 3000) {
    last = millis();
    oledLines("WiFi: CONECTADO", "IP: " + WiFi.localIP().toString(),
              "RSSI: " + String(WiFi.RSSI()) + " dBm");
  }
}
```

### Verificación

* Monitor Serie muestra la IP.
* OLED muestra top de redes y luego el estado conectado.

---

## Etapa 3 — Ejemplo **muy sencillo**: enviar algo desde Python al ESP32 (UDP) y verlo en la OLED

> Mantendremos esto **mínimo y claro** para no confundir: se envía **una línea de texto** por UDP y el ESP32 la muestra en la OLED. Sin JSON, sin protocolos extra.

### Paso A — Sketch ESP32 receptor UDP (basado en Etapa 2)

Crea `Etapa3_ESP32_UDP_OLED.ino` (mismo SSID/PASS). **Escucha** en `UDP_PORT = 5005`.

```cpp
#include <WiFi.h>
#include <WiFiUdp.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
#define OLED_ADDR     0x3C

const char* WIFI_SSID = "TU_SSID";
const char* WIFI_PASS = "TU_PASSWORD";
const uint16_t UDP_PORT = 5005;

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
WiFiUDP udp;

void showText(const String& t) {
  display.clearDisplay();
  display.drawRect(0,0,SCREEN_WIDTH,SCREEN_HEIGHT,SSD1306_WHITE);
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(4,4);
  // Imprime hasta 4 líneas de ~21 chars
  for (int i=0; i<4; ++i) {
    int a = i*21, b = min((int)t.length(), a+21);
    if (a < t.length()) display.println(t.substring(a,b));
  }
  display.display();
}

void setup() {
  Serial.begin(115200);
  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    Serial.println("OLED no encontrada"); while(true) delay(500);
  }
  display.clearDisplay(); display.display();

  WiFi.mode(WIFI_STA);
  showText("Conectando WiFi...");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  uint32_t t0 = millis();
  while (WiFi.status() != WL_CONNECTED) {
    delay(200);
    if (millis() - t0 > 15000) { showText("WiFi fallo"); while(true) delay(1000); }
  }

  udp.begin(UDP_PORT);
  showText("UDP listo\nIP: " + WiFi.localIP().toString() + "\nPort: " + String(UDP_PORT));
  Serial.println("Listo UDP en " + WiFi.localIP().toString() + ":" + String(UDP_PORT));
}

void loop() {
  int n = udp.parsePacket();
  if (n > 0) {
    static char buf[128];
    int len = udp.read(buf, sizeof(buf)-1);
    if (len < 1) return;
    buf[len] = '\0';
    String msg = String(buf);
    Serial.println("RX: " + msg);
    showText(msg);
  }
}
```

### Paso B — Script Python **mínimo** para enviar

Crea `enviar_udp_min.py` y ejecútalo así:
`python enviar_udp_min.py --ip 192.168.X.Y --port 5005 "Hola ESP32 desde PC"`

```python
# enviar_udp_min.py
import argparse, socket
p = argparse.ArgumentParser()
p.add_argument("mensaje")
p.add_argument("--ip", required=True)
p.add_argument("--port", type=int, default=5005)
a = p.parse_args()

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.sendto(a.mensaje.encode("utf-8", errors="ignore"), (a.ip, a.port))
print(f"Enviado -> {a.ip}:{a.port}: {a.mensaje}")
sock.close()
```

### Verificación

* En el Monitor Serie del ESP32 debe aparecer el texto recibido.
* La OLED debe mostrar exactamente ese texto (hasta 4 líneas).

---

## Etapa 4 — Integración: visión (PC) ↔ ESP32 (UDP)

### Objetivo

Usar las **piezas anteriores** para enviar al ESP32 las **coordenadas del centro** detectado en la Etapa 1, y ver una **cruz** en la OLED en la posición correspondiente.

### Formato de mensaje (simple, CSV)

`cx_norm,cy_norm` con valores normalizados `[0..1]`.

### Paso A — ESP32: dibujar cruz con coordenadas normalizadas

Crea `Etapa4_ESP32_UDP_Cruz.ino`:

```cpp
#include <WiFi.h>
#include <WiFiUdp.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
#define OLED_ADDR     0x3C
const char* WIFI_SSID = "TU_SSID";
const char* WIFI_PASS = "TU_PASSWORD";
const uint16_t UDP_PORT = 5005;

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
WiFiUDP udp;

void drawCross(int x, int y) {
  display.clearDisplay();
  display.drawRect(0,0,SCREEN_WIDTH,SCREEN_HEIGHT,SSD1306_WHITE);
  display.drawFastHLine(max(0,x-4), y, 9, SSD1306_WHITE);
  display.drawFastVLine(x, max(0,y-4), 9, SSD1306_WHITE);
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(4,52);
  display.printf("(%3d,%2d)", x, y);
  display.display();
}

void setup() {
  Serial.begin(115200);
  if(!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)){ while(true) delay(500); }
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status()!=WL_CONNECTED) delay(200);
  udp.begin(UDP_PORT);
  display.clearDisplay(); display.setTextSize(1);
  display.setCursor(0,0);
  display.println("Listo Etapa 4");
  display.println("IP: " + WiFi.localIP().toString());
  display.display();
}

void loop() {
  int n = udp.parsePacket();
  if (n>0) {
    static char buf[64];
    int len = udp.read(buf, sizeof(buf)-1);
    if (len<1) return;
    buf[len]='\0';
    float cx=0, cy=0;
    if (sscanf(buf, "%f,%f", &cx, &cy) == 2) {
      cx = fminf(fmaxf(cx, 0.0f), 1.0f);
      cy = fminf(fmaxf(cy, 0.0f), 1.0f);
      int x = (int)roundf(cx * (SCREEN_WIDTH-1));
      int y = (int)roundf(cy * (SCREEN_HEIGHT-1));
      drawCross(x,y);
      Serial.printf("RX norm:(%.3f,%.3f) -> px:(%d,%d)\n", cx, cy, x, y);
    }
  }
}
```

### Paso B — PC: enviar el centro detectado desde OpenCV

Partiendo de `etapa1_cuadrado.py`, crea `etapa4_envio_centro.py`:

```python
# etapa4_envio_centro.py
import cv2 as cv
import numpy as np
import socket, argparse, time

def cuadrado_mayor(frame):
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    blur = cv.GaussianBlur(gray, (5,5), 0)
    edges = cv.Canny(blur, 60, 150)
    contours, _ = cv.findContours(edges, cv.RETR_EXTERNAL, cv.CHAIN_APPROX_SIMPLE)
    best, best_area = None, 0
    for c in contours:
        peri = cv.arcLength(c, True)
        approx = cv.approxPolyDP(c, 0.02*peri, True)
        if len(approx)==4 and cv.isContourConvex(approx):
            area = cv.contourArea(approx)
            if area>1000:
                x,y,w,h = cv.boundingRect(approx)
                r = w/float(h)
                if 0.7<r<1.3 and area>best_area:
                    best_area = area; best = approx
    return best

def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--ip", required=True)
    ap.add_argument("--port", type=int, default=5005)
    ap.add_argument("--cam", type=int, default=0)
    args = ap.parse_args()

    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    cap = cv.VideoCapture(args.cam)  # En Mac: cv.VideoCapture(args.cam, cv.CAP_AVFOUNDATION)
    if not cap.isOpened():
        print("No se pudo abrir la camara"); return

    last_send = 0
    while True:
        ok, frame = cap.read()
        if not ok: break
        H, W = frame.shape[:2]

        sq = cuadrado_mayor(frame)
        if sq is not None:
            cv.drawContours(frame, [sq], -1, (180,0,255), 2)
            M = cv.moments(sq)
            if M["m00"] > 1e-3:
                cx = int(M["m10"]/M["m00"])
                cy = int(M["m01"]/M["m00"])
                cv.drawMarker(frame, (cx,cy), (0,255,0), cv.MARKER_CROSS, 18, 2)

                cx_n = cx / float(W)
                cy_n = cy / float(H)
                msg = f"{cx_n:.3f},{cy_n:.3f}"

                now = time.time()
                if now - last_send > 0.05:  # ~20 Hz
                    sock.sendto(msg.encode("utf-8"), (args.ip, args.port))
                    last_send = now
                    cv.putText(frame, f"Enviado: {msg}", (10, 30),
                               cv.FONT_HERSHEY_SIMPLEX, 0.7, (255,255,255), 2)
        else:
            cv.putText(frame, "No se detecta cuadrado", (10, 30),
                       cv.FONT_HERSHEY_SIMPLEX, 0.7, (0,0,255), 2)

        cv.putText(frame, "q: salir", (10, 60), cv.FONT_HERSHEY_SIMPLEX, 0.7, (255,255,255), 2)
        cv.imshow("Etapa 4 - Envio centro", frame)
        if cv.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv.destroyAllWindows()

if __name__ == "__main__":
    main()
```

### Verificación final

1. Flashea `Etapa4_ESP32_UDP_Cruz.ino` y anota la IP del ESP32.
2. Ejecuta en la PC:
   `python etapa4_envio_centro.py --ip 192.168.X.Y --port 5005`
3. Mueve el cuadrado frente a la cámara: la **cruz en la OLED** debe seguir el centro.

---


# Pistas y mejoras opcionales

* Si la OLED “parpadea” mucho, dibuja solo si cambió la posición.
* Para **acentos/UTF-8** en OLED, considerar **U8g2** (más fuentes).
* Para estabilidad de detección, prueba **erosión/dilatación** suave antes de Canny.
* **Viceversa (ESP32 → PC)**: el ESP32 puede enviar cada 1 s un “heartbeat” con `RSSI` por UDP y la PC mostrarlo en consola (reutiliza sockets de Etapa 3).

