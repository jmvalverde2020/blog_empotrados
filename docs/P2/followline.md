---
layout: team
---

# Sigue Lineas con conexión MQTT

Esta última práctica ha consistido en una competición, en la cual, usando el kit de elegoo smart robot car v4.0, debiamos completar un circuito hecho con cinta negra de la forma más rápida posible.

Aparte del código del sigue-lineas, contábamos con una ESP32 con conexión wifi, con la cuál debíamos enviar mensajes a un servidor a través de MQTT.

Dado que un código sigue-lineas no es excesivamente complicado, la dificultad de la prácitca residía en completar el circuito lo más rápido posible haciendo uso de todos los conocimientos adquiridos durante la asignatura y por lo tanto, se podían implementar multitud de soluciones y caracteristicas distintas.

En nuestro caso hablaremos de las implementaciones que hemos usado para completar esta práctica. Sin contar las que eran obligatorias como el envío de mensajes.

## Programación del ESP32
A la hora de implementar el ESP32 hemos necesitado crear una conexión para envío de mensajes con la placa arduino y otra conexion MQTT con la red.
### Conexión con la placa Arduino
Para conectarnos con la placa hemos usado la comunicación UART con la cual a través del puerto serie podemos recivir y enviar mensajes. Sin embargo, aunque el ESP32 cuenta con dos puertos series, la placa arduino solo cuenta con uno por lo que fue necesario crear un sistema para diferenciar los mensajes que eran para el ESP32 de los que no. El sistema fue poner entre corchetes los mensajes dirigidos a el ESP32. En el programa del ESP32, cuando se encontraba un inicio de corchete({), guardaba todos los caracteres siguientes hasta encontrar un final de corchete. 
### Comunicación MQTT
Para la conexión MQTT es necesario establecer conexión WiFi. Esto se hace al principio del programa, luego, dentro del loop, si no hay conexión con el server MQTT, tiene tres intentos para conectarse y si no se queda parado. Para el envio de mensajes hemos inplementado la QoS 2 (calidad de servicio exactly one) asegurandonos de que el mensaje se envie. Si hay un error a la hora de enviar el mensaje, se vuelve a enviar.  
### Envio del ping
Cada 4 segundos había que enviar un ping al servidor MQTT, en nuestro caso, simplemente hemos hecho un if en el loop en el que entra solo si el tiempo transcurrido desde el ultimo ping es de 4000ms o mas. 
## Implementaciones

- Controlador Proporcional.
- Búsqueda de línea.
- FreeRTOS.

### Controlador proporcional

La primera y principal implementación que hemos usado es un controlador proporcional para hacer que el robot gire acorde con la situación y ayudar a seguir la linea.

Hay cuatro situaciones claramente diferenciadas en las que buscábamos comportamientos diferentes entre ellas:

![Posibles casos](./media/casos.png)


- Linea en el centro: Este es el mejor caso posible, donde el robot ve la línea con el sensor central por lo que no necesita girar y puede ir lo más rápido posible.
- Linea entre el centro y un lateral: Esta situación se da cuando encuentra una curva y detecta la línea con el sensor central y uno lateral, en este caso debe girar proporcionalmente al valor que detecte el sensor lateral, es decir, que gire más cuanta más linea vea, ya que significará que se está saliendo del circuito.
- Linea en el lateral: Esta situación se da si el robot no ha sido capaz de girar a tiempo en una curva, la línea sólo es detectada por un sensor lateral, por lo que se está saliendo. Esta vez la velocidad de giro tiene que ser inversamente proporcional al valor del sensor ya que en caso de dejar de ver la línea, estará saliendose más del circuito. Si dejara de verla por estar centrándola, pasaría a la situacion anterior.
- Linea perdida: Esta es la peor situación, ninguno de los sensores detectan la línea y debe adoptar el comportamiento de encontrar la línea.

Para que gire más en la tercera situación que en la segunda, la asignación de velocidades es distinta.

Mientras que en la segunda la velocidad del lado que gira aumenta, y la del otro tiene un valor constante, en el otro caso, es la del lado que gira la que es constante y la del otro es la que disminuye conforme al valor del sensor.

Esto se hace para que en el caso en el que tenga que girar brusco nos aseguremos de controlar la velocidad de giro máxima y al disminuir la otra, el giro es más pronunciado.

No hemos utilizado factor D porque creemos que la entrada del sensor de color cambia drasticamente entre lectura y lectura debido a la velocidad que lleva el robot y la calidad de los sensores. Por otro lado, tampoco hemos implementado un I ya que añadirlo suele generar mayor oscilación o incluso volverlos un sistema inestable. 

### Búsqueda de línea

El algoritmo usado al perder la línea es bastante simple.
Únicamente consiste en girar hacia el lado que más velocidad tenga, normalmente el lado hacia el que estaba girando, a una velocidad predefinida para que el robot apenas avance y pueda encontrar la línea cerca del punto donde la perdió.

### FreeRTOS

Como hemos visto en clase, el uso de freeRTOS es muy útil para realizar tareas con distinta prioridad, siendo sobretodo útil a la hora de realizar tareas cuando la CPU esté libre ya que esto es algo muy complicado, si acaso posible, usando únicamente threads.

Por ello, pensamos en el uso de tiempo real desde el principio, además de por las distintas prioridades de las tareas, por el determinismo que aporta, lo cuál es muy útil para, por ejemplo leer del sensor de distancia cada cierto periodo. Aunque hacer un uso eficiente de ello es muy complicado, es cierto que conseguimos muy buenos resultados gracias a esto.

En nuestro código diferenciamos dos tareas:

- Detectar obstáculos
- Leer el sensor ITR y assignar las velocidades correspondientes a los servos.

En cuanto a la detección de obstáculos, para asegurar un buen funcionamiento del sensor, el periodo debía ser de 200ms lo cual a la hora de seguir la línea nos podía llevar a no actuar a tiempo, y dado que cortar la ejecución de esta tarea afectaba al comportamiento, decidimos asignarle mayor prioridad.

En un inicio, la lectura del sensor ITR y la asignación de las velocidades eran dos tareas distintas, sin embargo, tras muchos problemas a la hora de sincronizarlas para evitar comportamientos indeseados, decidimos juntarlas en una sola que lo hiciera de manera secuencial. Al medir el coste de la tarea decidimos ponerle un periodo de 20ms y al tener prioridad 0, actuaba cuando la CPU estaba libre, lo que a veces nos podía causar un comportamiento atrasado a altas velocidades.

En cuanto al envío de mensajes, pensamos en último momento en implementar una tercera tarea que se encargara de enviarlos quitándole coste a las otras dos, sin embargo la falta de tiempo y diversos problemas nos hicieron descartar esta idea.

## Video

La competición fue muy divertida y emocionante, y, aunque con muchos nervios al principio, el compañerismo durante el periodo de hacer la práctica y durante el examen nos dejó un muy buen sabor de boca.

A continuación un video de la mejor vuelta que hicimos con un tiempo de 8,4 segundos.

<div style="position: relative; padding-bottom: 56.25%; height: 0;"><iframe src="https://jumpshare.com/embed/l0hpFUtriFpmkP7DZdaG" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

y también un video de una vuelta en la que pierde la línea y la encuentra varias veces hasta terminar el circuito.

<div style="position: relative; padding-bottom: 56.25%; height: 0;"><iframe src="https://jumpshare.com/embed/mdSGopy96sfzCZZPoeFM" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

[Vuelve al blog](../)
