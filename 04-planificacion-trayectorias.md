---
layout: default
title: Planificación de trayectorias
nav_order: 5
---

# Planificación de trayectorias con UR5e

> Proyecto de Leo González Yamada

> Código `PlanTrayec_proceso.m` del [repositorio del proyecto](https://github.com/LeoGonYama39/JustTheDocs_Robotica).

En esta sección, se explicará y demsotrará la **planificación de trayectorias** de un **UR5e** efectuada en la aplicación industrial Pick and Place explicada en el inicio. Específicamente, usaremos el **método heurístico** y el **polinomio quíntico**

Como se mencionó el el apartado anterior, al resolver el problema por pura cinemática inversa, estamos atados a las acciones de las funciones nativas de UR, y si queremos controlar la velocidad de un movimiento, es necesario usar otros métodos como el **polinomio quíntico**, el cuál requiere del **método eurístico**.

## Contenido

- [Método eurístico](#método-eurístico)
- [Polinomio quíntico](#polinomio-quíntico)
- [Aplicación al problema](#aplicación-al-problema)
- [Demostración de la rutina](#demostración-de-la-rutina)
- [Análisis](#análisis)

---

## Método eurístico

Este método consiste en definir la pose deseada en función del tiempo con condiciones. En otras palabras, tenemos el control total de qué pose queremos que haga el robot en qué momento del tiempo, sin estar atados solamente a esperar a que el UR5e acabe de poner la pose indicada en las funciones `MoveJ` o `MoveL`.

Para este caso, usaremos las mismas poses definidas en la sección anterior (a exepción de una pose), las cuáles procesaremos con cinemática inversa para obtener las posiciones articulares y mover el robot con esas posiciones. Por lo tanto, seguiremos usando el método de inversa geométrica explorado en la sección anterior.

En el código de esta sección, pdoremos observar la siguiente secuencia (en segundos):

- **Si tiempo (t) menor a 1:** Moverse a la posición previa al agarre de la botella.
- **t de 1 a 10:** la botella se transportará por la banda.
- **t de 10 a 11:** el robot se moverá a la posición para agarrar la botella y la agarrará.
- **t de 11 a 12:** el robot volverá a la posición previa al agarre de la botella, pero con la botella agarrada por el gripper.
- **t de 12 a 13:** el robot se moverá a la posición previa a la colocacción de la botella.
- **t de 13 a 16:** el robot se moverá a la posición donde se dejará la botella con **polinomio quíntico**.
- **t de 16 a 17:** el gripper soltará el objeto y el robot volverá a la posición previa a la colocacción de la botella.

Por lo tanto, gracias al método eurístico, tenemos una rutina más controlada, donde podemos decidir qué hará el robot en cada momento de la rutina.

---

## Polinomio quíntico

Este método nos permite mover el robot con una trayectoria *suave*, de manera *segura* y *controlada*, y esto en función del tiempo, por lo que se puede convinar con el método eurístico.

Este polinomio, como lo dice su nombre, es de quinto grado, por lo que tendremos la siguiente forma:

![Forma del polinomio quíntico](/assets/img/poliQuint/poliQuint.jpg)

Esto definirá la posición del EF respecto del tiempo. Para su cálculo, tendremos el código `poliQuint.m`, al cuál se le tiene que introducir 6 parámetros: 

- **Posición inicial (d0)**
- **Posición final (df)**
- **Velocidad inicial (v0)** (generalmente 0)
- **Velocidad final (vf)** (generalmente 0)
- **Aceleración inicial (a0)** (generalmente 0)
- **Aceleración final (af)** (generalmente 0)

---

## Aplicación al problema

Para nuestra aplicación, mencionamos que, en todos los tiempos delimitados por el método eurístico, lo que se hace es poner directamente la posición articular (calculada por la cinemática inversa) al robot para moverse a esa posición, por lo que no estamos variando la posición en el tiempo, solo estamos diciendo que en ese lapso de tiempo, tenga determinada posición.

Sin embargo, si nos detenemos a ver la rutina desde 13 segundos a 16 segundos, es donde usamos el **polinomio quíntico**, donde la posición variará dependiendo del tiempo.

Este es el paso en el que el EF se moverá linealmente en su eje *z* para poder colocar la botella, por lo tanto, sólo queremos ese movimiento suave del polinomio quíntico en el eje *z*, no en otros ejes. Por lo tanto, aplicaremos únicamente el polinomio quíntico en el eje *z*.

> Si comparamos la función `getMatrizHomBotella` en el código anterior (`CI_proceso.m`) y el actual (`PlanTrayec_proceso.m`), podremos observar que en la matriz homogénea para la posición de colocación de la botella, mientras que en el código anterior hay un valor fijo, en el actual, se introduce una ecuación respecto al tiempo. 

Para el cálculo, usamos el código mencionado anteriormente (`poliQuint.m`), en donde calculamos el polinomio con z0 = 500.00 (posición en z del paso anterior) y zf = 230.00 (posición a la que quiero que se mueva en z desde z0).
Para todas las velocidades y aceleraciones, indicaremos 0, ya que la velocidad y aceleración inicial es 0, y queremos que también lo sea la velocidad y aceleración final.

---

## Demostración de la rutina

A continuación, un video de la rituna:

<iframe width="560" height="315" src="https://www.youtube.com/embed/y__e6M-BmEA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen>
</iframe>

---
## Análisis
Después de haber visto la rutina funcionando, podemos concluir que:

- Se logró implementar una **rutina más controlada**, en donde tenemos vía libre para decidir en qué momento queremos que ocurra determinada acción.

- La botella se coloca de manera **suave**, lo que permite tratar al producto con más **delizadeza**, cosa que es importante, sobre todo tomando en cuenta que es una bebida.

- Se puede concluir que esta rutina, realizada con planificación de trayectorias, es una **mejora directa** a la rutina realizada únicamente con cinemática inversa.
