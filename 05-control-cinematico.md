---
layout: default
title: Control cinemático
nav_order: 6
---

# Control cinemático con UR5e

> Proyecto de Leo González Yamada

> Código `CC_UR5e.m` del [repositorio del proyecto](https://github.com/LeoGonYama39/JustTheDocs_Robotica).

En esta sección, se demostrará el control cinemático del UR5e. Se controlará únicamente las posiciones cartesianas *x*, *y* y *z* del EF, sin controlar la orientación.

También, se usará el método de **ecuaciones paramétricas**, combinado nuevamente con el **método eurístico**, para dibujar un círculo con el EF.

## Contenido

- [Ecuaciones paramétricas](#ecuaciones-paramétricas)
- [Control cinemático](#control-cinemático)
- [Aplicación](#aplicación)
- [Demostración de la rutina](#demostración-de-la-rutina)
- [Análisis](#análisis)

---

## Ecuaciones paramétricas

Parecido al polinomio quíntico, en este método moveremos al EF en función dle tiempo, pero con la diferencia de que vamos a describir cuerpos geométricos a partir de ecuaciones.

Como ejemplo, la ecuación de un círculo, la cuál es:

```text
x = Cx + rcos(ft)
y = Cy + rsen(ft)
```

Donde:
- **x:** posición en eje x.
- **y:** posición en eje y.
- **Cx:** centro del círculo en eje x.
- **Cy:** centro del círculo en eje y.
- **r:** radio del círculo.
- **f:** frecuencia.
- **t:** tiempo.

Esta ecuación es la que usaremos para esta demostración, pero el método es capaz de describir cualquier figura geométrica, siemrpe y cuando se cuente con su ecuación paramétrica.

---

## Control cinemático

Es una técnica de control que permite satisfacer una pose deseada calculando las velocidades angulares necesarias para alcanzar la pose.

La fórmula es la siguiente:

![Fórmula de control cinemático](/assets/img/control/formula_CC.jpg)

Donde:

- **q.:** perfiles de velocidad
- **J:** Matriz Jacobiana
    - Esta matriz jacobiana se forma tomando en cuenta las ecuaciones de cinemática directa como funciones, y cada articulación de *q* como un estado.
- **Xd.:** Vector de velocidades cartesianas deseadas.
- **K:** Vector de ganancias.
- **X:** Vector de posiciones cartesianas actuales.
- **Xd:** Vector de posiciones cartecianas deseadas.

---

## Aplicación

Para satisfacer la fórmula, veamos cómo se realizó en el código de esta sección:

En la función `CC_UR5e.m`, se definen una por una cada variable necesaria para el cálculo del perfil de velocidades.

1. Definir la **cinemática directa** de *x*, *y* y *z*, pero a diferencia de la sección de cinemática directa visto en este proyecto, definiremos la cinemática en función de las posiciones articulares.

2. Definir la **matriz jacobiana**. Para la primera fila, se deriva la primera función (cinemática directa de *x*) por cada posición articular. Por lo tanto, haremos 6 derivadas, y la misma lógica aplica para la segunda columna con *y* y tercera con *z*.

3. Se calcula la **inversa** de la matriz **Jacobiana**, que es la matriz que se terminará usando para el cálculo de perfiles de velocidad.

    - Hay que tener en cuenta que, la inversa de una matriz solamente se puede calcular con una matriz cuadrada nxn, y como en este caso, nuestra matriz jacobiana es de 3x6, se usará la **pseudoinversa** de Moore-Penrose.

4. Se **fijan** las **posiciones deseadas**. Este paso es importante, ya que nuevamente usaremos **planificación de trayectorias** con el método **Heurístico**. Como se hizo en la sección anterior, fijaremos posiciones específicas para un determinado tiempo. 
    
    - Para el intervalo de 10 a 20 segundos, se introduce en *x* y *y*, las ecuaciones paramétricas de un círculo de radio 0.1, con su centro en 0.2 tanto para *x* y *y*. Esto dibujará un circulo en el eje *x* y *y* en la altura de *z* = 0.543.

5. **Definir** las **velocidades deseadas**. Para este punto, es importante entender que la velocidad es la derivada de la posición, por lo que simplemente vamos a derivar las posiciones deseadas fijadas en el punto anterior respecto a la variable de tiempo *t*. 

    - Para todos los casos, la derivada sería de 0, devido a que los puntos que fijamos en el paso 4 son puntos fijos en el espacio, **a excepción** del intervalo 10 a 20 segundos. En este caso, derivamos las ecuaciones paramétricas del círculo, y para el mismo intervalo de 10 a 20 segundos, fijamos las velocidades de *x* y *y* como la derivada calculada.

6. **Fijar** las **ganancias del control**. Es una ganancia por estado (*x*, *y* y *z*). Normalmente se fija 1.


Esta función se pasará por un **método numérico iterativo** con el tiempo de simulación fijado, que en este caso es de 0 a 25 segundos, con intervalos de 0.1 segundos. 

Esto nos arrojará un arreglo con todos los vectores **q** para cumplir nuestra trayectoria planificada. Esto se introducirá en un ciclo for, en donde mandaremos el cada vector **q** a nuestro robot UR5e en RoboDK, para comprobar el control cinemático.

> Como en lo puntos anteriores, al trabajar con RoboDK, se tienen que pasar los ángulos en radianes a grados con la función `rad2deg`.

---

## Demostración de la rutina

A continuación, un video del control cinemático en RoboDK:

<iframe width="560" height="315" src="https://www.youtube.com/embed/UPhmXkkg74Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen>
</iframe>

---

## Análisis

Después de ver el control cinemático en acción, podemos concluir varios puntos:

- El control cinemático es otra forma de resolver el problema de la definición de las articulaciones de un robot para **satisfacer una posición**. Y este modelo se puede extrapolar a un control no solamente cartesiano, si no también rotacional, simplemente calculando la cinemática directa de las 3 rotaciones *rx*, *ry* y *rz* (con todo lo que implicaría este cambio, que serían más componentes a la matriz jacobiana, más ganacias K, etc.)

- A diferencia de la planificación de trayectorias con cinemática inversa, al usar el control cinemático, tenemos algo destacable, que es el hecho de que este tiene un movimiento más suave. Mientras que en la planeación con inversa, tenemos el movimiento en manos de las funciones de UR, en este caso, tenemos un **mayor control del movimiento**.

- Sin embargo, como se muestra en el video, en algunos fragmentos, el programa se pausa justo en el momento en el que ya debería de estar cumplida la posición deseada para comprobar la posición, y podemos observar que esto no pasa en todas las posiciones. Esto tiene una explicación y es que **no lo estamos dando el tiempo suficiente** al robot para moverse a esa posición. Esto se puede resolver **aumentando las ganancias**, pero esto también influirá en las posiciones que ya se cumplen a tiempo, y sin mencionar que podemos llegar a provocar un oscilamiento prolongado en la respuesta o hasta la desestabilización del sistema.

- Este problema se puede resolver con algo ya conocido para este punto: **polinomio quíntico**. Como ya sabemos, este método nos permite modelar a detalle la respuesta que queremos ver en nuestro EF, incluyendo el **tiempo de inicio y final** de un movimiento. Por lo tanto, resolveríamos el problema que tiene el control cinemático, agregando un método que asegura el movimiento de punto A a B en un determinado tiempo. Una ventaja es que el polinomio quíntico, como vimos en la sección anterior, se puede implementar a una planificación de trayectorias por método Heurístico.