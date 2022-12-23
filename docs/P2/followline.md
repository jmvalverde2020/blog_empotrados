---
layout: team
---

# Sigue Lineas con conexión MQTT

Esta última práctica ha consistido en una competición, en la cual, usando el kit de elegoo smart robot car v4.0, debiamos ompletar un circuito hecho con cinta negra de la forma más rápida posible.

Aparte del código del sigue-lineas, contábamos con una ESP32 con conexión wifi, con la cuál debíamos enviar mensajes a un servidor a través de MQTT.

Dado que un código sigue-lineas no es excesivamente complicado, la dificultad de la prácitca residía en completar el circuito lo más rápido posible haciendo uso de todos los conocimientos adquiridos durante la asignatura y por lo tanto, se podían implementar multitud de soluciones y caracteristicas distintas.

En nuestro caso hablaremos de las implementaciones que hemos usado para completar esta práctica. Sin contar las que eran obligatorias como el envío de mensajes.

## Implementaciones

- Controlador Proporcional.
- Búsqueda de línea.
- FreeRTOS.

### Controlador proporcional

La primera y principal implementación que hemos usado es un controlador proporcional para hacer que el robot gire acorde con la situación y ayudar a seguir la linea.

Hay cuatro situaciones claramente diferenciadas en las que buscábamos comportamientos diferentes entre ellas:

- Linea en el centro: Este es el mejor caso posible, donde el robot ve la línea con el sensor central por lo que no necesita girar y puede ir lo más rápido posible.
- Linea entre el centro y un lateral: Esta situación se da cuando encuentra una curva y detecta la línea con el sensor central y uno lateral, en este caso debe girar proporcionalmente al valor que detecte el sensor lateral, es decir, que gire más cuanta más linea vea, ya que significará que se está saliendo del circuito.
- Linea en el lateral: Esta situación se da si el robot no ha sido capaz de girar a tiempo en una curva, la línea sólo es detectada por un sensor lateral, por lo que se está saliendo. Esta vez la velocidad de giro tiene que ser inversamente proporcional al valor del sensor ya que en caso de dejar de ver la línea, estará saliendose más del circuito. Si dejara de verla por estar centrándola, pasaría a la situacion anterior.
- Linea perdida: Esta es la peor situación, ninguno de los sensores detectan la línea y debe adoptar el comportamiento de encontrar la línea.

![Posibles casos](./media/casos.png)

## Video

Aquí un video de la mejor vuelta con un tiempo de 8,4 segundos.

<div style="position: relative; padding-bottom: 56.25%; height: 0;"><iframe src="https://jumpshare.com/embed/l0hpFUtriFpmkP7DZdaG" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

y también un video de una vuelta en la que pierde la línea y la encuentra varias veces hasta terminar el circuito.

<div style="position: relative; padding-bottom: 56.25%; height: 0;"><iframe src="https://jumpshare.com/embed/mdSGopy96sfzCZZPoeFM" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

[Vuelve al blog](../)
