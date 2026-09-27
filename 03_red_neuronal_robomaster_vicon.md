---
layout: default
title: Red neuronal RoboMaster $\rightarrow$ Vicon
nav_order: 3
---

# Red neuronal RoboMaster $\rightarrow$ Vicon

## Arquitectura y entrenamiento
El bloque de estimación de pose es una capa lineal de 12 entradas y 6 salidas (78 parámetros nominales, incluido el sesgo), entrenada de forma supervisada para predecir la pose Vicon (TX, TY, TZ en metros; RX, RY, RZ del ángulo global de Vicon en radianes, que no corresponden a los ángulos de Euler yaw/pitch/roll) a partir de un vector de características del robot:

1. Posición del chasis en X, Y (m) y el tercer componente del SDK convertido a radianes.
2. Orientación: yaw (continuo), pitch y roll (rad).
3. Últimos comandos aplicados: $v_x$, $v_y$ (m/s) y velocidad angular (rad/s).
4. Velocidades odométricas en X, Y y la derivada del tercer componente del SDK.

El entrenamiento usó dos sesiones registradas de forma sincronizada por correlación cruzada de la rapidez (correlación temprana de 0.912, desfase de 2.30 s), con muestreo causal de las señales del robot a 10 Hz y filtrado FIR de 4 Hz (semiancho de 0.5 s) sobre las etiquetas Vicon. La división temporal fue 65/15/20 % (entrenamiento/validación/prueba) con guardas de 1 s entre segmentos, y el ajuste de la capa lineal se realizó con un optimizador Gauss–Newton amortiguado propio, con parada temprana a las 20 épocas sin mejora (mejor época: 8 de 28 ejecutadas). La tabla resume las métricas obtenidas.

**Tabla:** RMSE de posición 3D (TX, TY, TZ) de la red 12 $\to$ 6 usada en el lazo de control, por partición de datos.

| Partición | $n$ | RMSE 3D (cm) | Condición del jacobiano |
| :--- | :---: | :---: | :---: |
| Entrenamiento | 631 | 4.17 | 1.07 |
| Validación | 128 | 5.37 | 1.07 |
| Prueba (cronológica) | 188 | 7.15 | 1.07 |

El aumento del error de 4.17 cm en entrenamiento a 7.15 cm en la partición de prueba —que corresponde a los últimos segmentos temporales de la misma sesión, no a un segundo recorrido independiente— es la primera fuente cuantificada de la acumulación de error que se discute más adelante: el modelo aprendido tiene un sesgo de generalización no despreciable frente al tamaño típico de las figuras trazadas (entre 0.2 m y 1.0 m).

## Del modelo normalizado al jacobiano físico
Internamente la red aprende $\hat{y} = \frac{x-\mu_x}{\sigma_x}W + b$ sobre variables normalizadas; el sistema la reexpresa en unidades físicas como

$$
\hat{y} = A\,x + b_{\text{fis}}, \qquad A = \frac{W_{1:}\,\sigma_y}{\sigma_x},
$$

conservando las 12 entradas y las 6 salidas. El bloque de posición del jacobiano, $J = A_{[1:2,1:2]}^{\top}$, resultó bien condicionado (número de condición 1.07, con umbral de rechazo en 20), lo que descarta que la inversión diferencial en sí misma sea una fuente relevante de amplificación numérica del error.
