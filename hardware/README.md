# Hardware

| Carpeta | Contenido |
|---|---|
| `cad/` | Piezas y ensambles (SolidWorks / Onshape / SketchUp) |
| `stl/` | Archivos listos para imprimir (pisos 0 y 1, soporte en H, carrocería, eslabones del brazo, gripper, soporte del celular) |
| `electronica/` | Esquema de conexiones, placa de potencia, pinout de los ESP32 |

> GitHub rechaza archivos de más de 100 MB. Si algún ensamble los supera, subir solo las piezas o exportar a STEP.

## Alimentación
Pack Li-ion 21 V (5S) con BMS + fusible 10 A → bucks:
- XL4015 → 9,5 V: VMS de los dos L298N
- XL4015 → 5 V: lógica L298N + VIN de los ESP32
- XL4016 → rail de servos del brazo (entrada directa de batería)
