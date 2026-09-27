---
layout: default
title: Inicio
nav_order: 1
---

# Proyecto Pick and Place con UR5e, en RoboDK

> Proyecto de Leo González Yamada

En esta página, se documenta la simulación de un espacio de trabajo robotizado integrado en el entorno **RoboDK**. El sistema consiste en una solución industrial de Pick and Place, donde un robot **UR5e** equipado con un efector final (EF) **OnRobot SG-a Soft Gripper** realiza el ciclo de ordenamiento de diez botellas en un contenedor final (caja).

Para la ejecución de la rutina, se implementaron diversas técnicas de cinemática y planificación mediante la integración de **Matlab** y el **Add-On de RoboDK**, validando los modelos matemáticos directamente en la simulación:

- Para empezar, se hará una demsotración de **cinemática directa** del UR5e. Este paso es importante ya que posteriormente, se usará esta misma cinemática directa para la implementación del **control cinemático**.

- Se realizará la implemetnación industrial del robot, en donde hará su rutina de ordenamiento de botellas en una caja:
    - Primero se hará una demostración con **cinemática inversa**, en donde esta se usará para calcular la posición articular (**q**) del UR5e con una pose (**X**) deseada. Se variará la matriz de rotación **R**, como demostración de que se puede calcular la pose con distintas rotaciones. La q será directamente entregada al UR5e con la función `MoveJ` o `MoveL` del Add-On de RoboDK.

    - Debido a las limitaciones de controlar la velocidad del EF, se hará otra demostración de la implementación, pero usando el método de **planeación de trayectorias** heurístico y polinomio quíntico, los cuales nos permitirán mover el actuador de una forma **segura** y **controlada**.

- Por último, se hará una demsotración de **control cinemático** de solo posición cartesiana (x, y, z) del EF. Será únicamente una demostración del método fuera de la implementación industrial, en donde se demostrará con **ecuaciones paramétricas** cómo se puede dibujar un círculo con el EF.
