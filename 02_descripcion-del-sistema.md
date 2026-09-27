---
layout: default
title: Descripción del sistema
nav_order: 2
---

# Descripción del sistema

## Arquitectura general
El sistema consta de tres componentes que se comunican mediante el SDK oficial de DJI para el RoboMaster S1: (i) un módulo de adquisición que se suscribe a la posición y actitud del chasis, (ii) un módulo de inferencia/control que estima la pose en el marco Vicon y calcula el comando de velocidad, y (iii) un registrador que guarda, por cada ciclo de control, el comando aplicado, la telemetría cruda, la posición estimada, la referencia de trayectoria y —cuando está disponible— la medición óptica recibida por un puente UDP opcional. La captura Vicon utilizada para la validación de este reporte proviene de un objeto rígido (*equipo_juan*) muestreado a 100 Hz, con salidas de traslación (TX, TY, TZ) en milímetros y de ángulo global (RX, RY, RZ) en radianes.

## Límite de frecuencia de telemetría y de comando
El firmware/SDK del RoboMaster S1 impone límites de frecuencia que acotan el desempeño alcanzable del lazo cerrado, independientemente de la calidad del modelo o del controlador:

- **Suscripción de pose (posición y actitud del chasis):** 50 Hz. Ésta es, en la práctica, la tasa máxima a la que el controlador recibe una nueva muestra de odometría/IMU para corregir su estimación de posición.
- **Bucle de control y envío de comando (`drive_speed`):** se ejecuta a un periodo objetivo de 20 ms (50 Hz), limitado por el mismo ritmo de llegada de telemetría.
- **Vigilancia de comando (*command timeout*) del SDK:** 0.3 s. Si no llega un nuevo comando de velocidad dentro de esa ventana, el robot se detiene por seguridad; el bucle de 50 Hz deja un margen amplio frente a este límite.
- **Actualización de las características "lentas" de la red (yaw acumulado, últimos comandos, velocidades odométricas):** 10 Hz. Entre una actualización y la siguiente, únicamente se refresca la corrección posicional rápida (posición XY relativa), no el resto del vector de entrada de 12 componentes.
- **Vigilancias de abandono del lazo:** pérdida de telemetría por más de 0.5 s, ausencia de latido de interfaz por más de 0.8 s, o —en el modo con medición óptica en vivo— pérdida de la muestra Vicon por más de 0.3 s, detienen el recorrido.

Estos límites conviven con la frecuencia de 100 Hz del propio sistema Vicon usado para la validación: el sistema óptico resuelve la trayectoria a un ritmo el doble de rápido que el lazo interno del robot, lo cual es deseable para que la validación no esté limitada por el instrumento de medición.