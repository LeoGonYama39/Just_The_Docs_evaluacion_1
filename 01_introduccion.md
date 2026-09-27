---
layout: default
title: Introducción
nav_order: 1
---

# Aplicación de control inteligente a Robomaster s1 para la realización de trayectorias

> Proyecto de Emilio Acuña, Joshua Barragán, Leo González y Javier Lachica

El seguimiento preciso de trayectorias en robots móviles terrestres suele apoyarse en un sensor externo de alta precisión --típicamente un sistema de captura de movimiento óptico como Vicon-- para cerrar el lazo de control en el marco de referencia del laboratorio. Sin embargo, en plataformas educativas y de bajo costo como el DJI RoboMaster S1, el acceso a esa medición en tiempo real dentro del propio lazo de control no siempre es práctico, ya sea porque el puente de comunicación introduce latencia, porque el software del fabricante restringe el acceso directo al chasis, o porque se desea evaluar hasta qué punto la sola telemetría interna del robot (odometría de ruedas, IMU y comandos aplicados) permite estimar la posición en el marco Vicon sin depender de la medición óptica durante el movimiento.

Este reporte documenta una arquitectura que resuelve ese problema en dos partes. Primero, una red neuronal lineal aprende de forma *fuera de línea* el mapeo RoboMaster -> Vicon, es decir, a partir de la telemetría del robot predice la pose que reportaría el sistema óptico. Segundo, en línea, el jacobiano de esa misma red se invierte de forma diferencial (no se reentrena durante el recorrido) para calcular, dado un error de posición y una velocidad de referencia deseados, el comando de velocidad del chasis que los produciría. El sistema se ejecuta de manera autónoma trazando cinco figuras geométricas prescritas (círculo de dos tamaños, cuadrado, figura en ocho y zigzag), y el propio sistema Vicon --usado aquí exclusivamente como instrumento de validación externo, no como realimentación del lazo-- registra la trayectoria físicamente recorrida. Este documento presenta el sistema, el modelo aprendido, la ley de control, y contrasta cuantitativamente cada trayectoria real contra su referencia ideal.

