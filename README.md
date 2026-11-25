# Rastrojero — Robot logístico autónomo

Proyecto Final de Ingeniería Mecatrónica — Facultad de Ingeniería, UNLZ.
Autor: Lucas Martín Toranzo

Vehículo autónomo para transporte y picking de cajas en almacenes. Sigue una línea con sensores IR, esquiva obstáculos con un ultrasonido montado en servo, se detiene en marcas, confirma la posición escaneando un código QR y toma la caja con un brazo robótico de 3 GDL + gripper. La carrocería homenajea al Rastrojero de IME.

## Arquitectura

```
 Panel web (Flask, PC)  ◄──── MQTT / TLS (HiveMQ Cloud) ────►  ESP32 chasis
        ▲                                                    ESP32 brazo
        │ video (navegador)
   Celular (cámara: QR + ubicación de la caja)
```

| Subsistema | Componentes |
|---|---|
| Chasis | ESP32, 2× L298N, 4 motorreductores 12 V 100 RPM, ruedas mecanum |
| Sensado | 3 sensores IR (módulo TCRT5000), HC-SR04 sobre servo SG90 |
| Brazo | NEMA 17 (base, direct drive) + A4988 a 1/8 de paso, encoder AS5600, 2× MG946R (hombro/codo), S3003 (gripper) |
| Visión | Celular transmitiendo al servidor; OpenCV para QR y ubicación de la caja |
| Energía | Pack Li-ion 21 V (5S) con BMS, fusible 10 A, bucks XL4015/XL4016 |

## Pinout

**ESP32 chasis**

| Función | GPIO |
|---|---|
| L298N izq IN1–IN4 | 27, 26, 25, 32 |
| L298N der IN1–IN4 | 22, 19, 18, 33 |
| HC-SR04 TRIG / ECHO (con divisor) | 13 / 4 |
| Servo del ultrasonido | 21 |
| IR izq / centro / der | 34 / 35 / 39 |

**ESP32 brazo**

| Función | GPIO |
|---|---|
| A4988 STEP / DIR / ENABLE | 25 / 26 / 27 |
| Servo hombro / codo / gripper | 4 / 18 / 19 |
| AS5600 SDA / SCL (I2C 0x36) | 21 / 22 |

## Estructura del repositorio

```
hardware/          CAD, STL y electrónica
software/
  firmware/        Sketches de los ESP32 (Arduino IDE)
  servidor/        Panel web de control y calibración (Python)
  legacy/          Versiones anteriores, solo como referencia
docs/              Propuesta, planificación, informe final
```

## Cómo usarlo

### Firmware (Arduino IDE)
1. Bibliotecas: `PubSubClient`, `ArduinoJson`, `AccelStepper`.
2. Copiar `software/firmware/credenciales_ejemplo.h` como `credenciales.h` dentro de la carpeta del sketch y completar WiFi y HiveMQ.
3. Placa: *ESP32 Dev Module*. Compilar y subir.

### Servidor
```bash
cd software/servidor/panel_rastrojero
python -m venv venv
venv\Scripts\activate        # Windows
pip install -r ../requirements.txt
copy ..\credenciales_ejemplo.py credenciales.py   # y completar
python panel_rastrojero.py
```
Abrir `http://localhost:5000`.

## Tópicos MQTT principales

| Tópico | Sentido | Uso |
|---|---|---|
| `robot/cmd` | panel → chasis | comandos de movimiento y modos |
| `robot/config` | panel → chasis | parámetros en vivo |
| `robot/telemetria` | chasis → panel | estado cada 250 ms |
| `robot/qr_solicitud` | chasis → servidor | pedido de escaneo en una marca |
| `robot/qr_confirmacion` | servidor → chasis | resultado del QR |
| `robot/brazo/cmd` | panel → brazo | ángulos / home / gripper |
| `robot/brazo_estado` | brazo → panel | estado del brazo |
