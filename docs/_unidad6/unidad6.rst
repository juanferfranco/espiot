Unidad 6: nube
========================

Sesión 1
-----------

En esta sesión analizaremos la estructura del proyecto ``5_cloud``.

Ejercicios
-----------

En este proyecto vamos a conectar a la Nube el ESP32. Específicamente nos 
vamos a conectar a un servicio de Amazon conocido como AWS IoT Core.


Ejercicios 1: ¿Qué es AWS IoT Core?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`AWS IoT Core <https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html>`__ 
es un servicio de Amazon que 
permite conectar los dispositivo embebidos de una aplicación IoT a la Nube. La ventaja 
del servicio es que no es necesario configurar servidores, solo se necesita provisionar 
el dispositivo embebido con los certificados necesarios para conectarse de manera 
SEGURA al servicio.

Una vez los embebidos estén conectados al servicio podrán enviar y recibir información 
de manera robusta y segura e interactuar con otros servicios de Amazon u otros servidores 
o clientes a través de AWS IoT Core.

El propio Amazon provee una biblioteca escrita en lenguaje C denominada 
`AWS IoT Device SDK for Embedded C <https://github.com/aws/aws-iot-device-sdk-embedded-C>`__ 
para conectar el sistema embebido a AWS IoT Core.

Para integrar la biblioteca anterior a un proyecto con el ESP32 y el esp-idf tenemos 
que adaptarla. Afortunadamente Espressif ya nos hizo el trabajo y la biblioteca está 
lista para integrar como un componente más del proyecto. En 
`este <https://github.com/espressif/esp-aws-iot>`__ enlace se puede ver el código del 
componente; sin embargo, no tenemos que descargarlo porque ya lo tenemos en la carpeta de 
componentes del proyecto de curso junto al componente ``button`` que ya analizamos.


Ejercicios 2: ¿Qué protocolo se utiliza para conectar el ESP32 con IoT Core?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

En la siguiente figura (tomada de `aquí <https://docs.aws.amazon.com/iot/latest/developerguide/aws-iot-how-it-works.html>`__) 
se puede ver una arquitectura típica de una aplicación IoT:

.. image:: ../_static/iot-universe.png
   :alt:  arquitectura IoT
   :scale: 100%
   :align: center

En nuestro caso, el ESP32 se conectará a la Nube por medio del servicio AWS IoT Core 
utilizando el protocolo `MQTT <https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html>`__. 
De nuevo, IoT Core servirá como una PASARELA entre otros servicios de Amazon u otros 
servidores. AWS IoT Core tendrá un servidor especial conocido como ``MESSAGE BROKER`` al cual 
se conectarán otros dispositivos para ``PUBLICAR`` MENSAJES PARTICULARES o para ``SUSCRIBIRSE`` 
a un mensaje en específico. Un ESP32 podría entonces conectarse al BROKER usando MQTT para 
publicar un mensaje y/o para suscribirse a otro. 

¿Cómo se identifica un mensaje? se identifica por medio de un TÓPICO. Los tópicos 
tienen nombres. Por ejemplo: ``sensor/temperatura/maquina1``. Por tanto, podrías tener 
un ESP32 midiendo la temperatura de la máquina 1 y enviando mensajes a AWS IoT Core 
con el tópico ``sensor/temperatura/maquina1``. Ahora imagina que tienes una aplicación 
móvil para monitorear la temperatura de la máquina 1. Entonces lo que tendrá que hacer 
la aplicación es suscribirse al tópico ``sensor/temperatura/maquina1`` para recibir 
por parte del MESSAGE BROKER notificaciones cada vez que se publique una actualización 
a ese tópico.

Ejercicios 3: ¿Qué necesitas para conectar un ESP32 a AWS IoT Core?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Se necesita:

* Una cuenta en Amazon.
* Habilitar el servicio AWS IoT Core en tu cuenta.
* Crear una ``Thing`` en AWS IoT Core que represente al ESP32 específico 
  que vas a conectar. 
* El ESP32 deberá autenticarse con AWS IoT Core utilizando un certificado de cliente 
  X.509. Por tanto, tendrás que generar ese certificado y asociarlo con la 
  Thing de ese ESP32.
* El ESP32 tendrá que enviar a AWS IoT Core toda la información encriptada, 
  por tanto, necesitarás generar una CLAVE privada que almacenarás en el ESP32.
* El ESP32 tendrá que asegurarse que si se está conectando 
  a un servidor VERDADERO de AWS IoT Core. Entonces necesitarás el certificado del 
  servidor para autenticarlo.
* Amazon tiene servidores por todo el mundo, entonces tendrás que saber a cuál 
  servidor debes conectar el ESP32. A esto se le conoce como ENDPOINT.

En resumen, necesitarás: 

* El certificado del servidor o AWS CA Certificate (Certificate Authority).
* El certificado del cliente y la clave privada.
* El identificador del Thing asociado con el ESP32.
* La dirección del endpoint.

Todo esto se tendrá que guardar en el dispositivo.

Ejercicios 4: autenticación 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

En el ejercicio anterior viste que para conectarse a AWS IoT Core es necesario 
que el ESP32 autentique al servidor y a su vez el servidor tendrá que autenticar 
al ESP32.

Para profundizar más al respecto te dejo un 
`EXCELENTE enlace <https://realtimelogic.com/articles/Certificate-Management-for-Embedded-Systems>`__ 
que habla sobre este asunto específicamente para sistemas embebidos.


Ejercicios ?: introducción al proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el proyecto ``5_cloud``. No olvides configurar los puertos 
de entrada-salida correspondientes a tu hardware usando el archivo ``board_esp32_devkitc.h``.

Lee la descripción de este proyecto `aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/remotecontrol.html>`__.

Ejercicios ?: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

En este proyecto el ESP32 se conectará a un servicio de Amazon en Internet conocido 
como AWS IoT Core.









Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.