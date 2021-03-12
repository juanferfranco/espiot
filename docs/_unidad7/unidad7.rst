Unidad 7: actualización remota
================================

Sesión 1
-----------

En esta sesión analizaremos la estructura del proyecto ``6_ota``.

Ejercicios
-----------

Ejercicios 1: introducción al proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Una de las condiciones que ``NO PUEDEN`` faltar a la hora de desplegar en PRODUCCIÓN un 
dispositivo IoT es la capacidad de poder actualizar remotamente el firmware. El 
proyecto de esta unidad se enfoca en adicionarle esta característica al proyecto 
de curso.

A la posibilidad de actualizar remotamente el firmware de un dispositivo se le conoce 
como Over-The-Air update u OTA. Como te podrás imaginar el esp-idf soporta esta 
característica esencial para un producto IoT.

Ejercicio 2: particiones de la memoria flash
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Para comprender cómo funciona y cómo utilizar las opciones de OTA que ofrece el 
esp-idf, lo primero que debes hacer es entender cómo funciona el esquema de 
partición de la memoria flash del microcontrolador en el esp-idf. Para lograr 
lo anterior te voy a proponer que leas este `blog <https://medium.com/the-esp-journal/how-to-use-custom-partition-tables-on-esp32-69c0f3fa89c8>`__ 
hasta a sección que dice DEMO, pero sin incluirla.
(Si el enlace no te abre, intenta abrirlo en modo incógnito).

Ejercicio 3: mini-reto 1
^^^^^^^^^^^^^^^^^^^^^^^^^

Regresa al proyecto ``4_network_config`` y analiza el archivo ``partitions.csv``.

* ¿Cuántas particiones de datos se están definiendo?
* ¿Cuáles son los subtipos de particiones de datos que se definen?
* ¿Cuántas particiones de aplicación se definen?
* ¿Cuál es el subtipo de las particiones de aplicación?
* ¿De qué tamaño es cada partición definida?

Ejercicio 4: mini-reto 2
^^^^^^^^^^^^^^^^^^^^^^^^^

Programa de nuevo el ESP32 con el código del proyecto ``4_network_config``. Ejecuta los siguientes 
comandos, primero para leer una partición y luego para poder visualizar su contenido en la salida 
estándar:

.. code-block:: bash

    esptool.py read_flash 0x8000 0xc00 ptable.img
    gen_esp32part.py ptable.img

* ¿En qué OFFSET comienza cada partición?
* Calcula la dirección de inicio de phy_init usando el OFFSET y el SIZE de NVS.
* Repite el procedimiento anterior para calcular el inicio de factory usando el offset 
  y size de phy_init. ¿Por qué esta vez no te da? Si no recuerdas regresa a la lectura 
  del `blog <https://medium.com/the-esp-journal/how-to-use-custom-partition-tables-on-esp32-69c0f3fa89c8>`__, 
  pero esta vez salta directamente a la sección ``OFFSET``.

Ejercicio 5: mini-reto 3
^^^^^^^^^^^^^^^^^^^^^^^^^

Lee solo la sección Flash partitions `aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/firmwareupgrade.html#flash-partitions>`__.

Vuelve al proyecto ``4_network_config`` y ejecuta menuconfig:

.. code-block:: bash

    idf.py menuconfig

Ve a la sección Partition Table de menuconfig.

* ¿Cuántas tipos de tablas de partición ves disponibles?
* Para usar una tabla de partición custom o personalizada qué debes 
  hacer?
* ¿Es posible que tu primera partición esté definida en un OFFSET 
  predefinido por ti?
* ¿Cuál sería el OFFSET inicial de tu primera partición si no cambias
  el bootloader por defecto del ESP-IDF? Si no lo recuerdas, vuelve al blog.


Ejercicio 6: mini-reto 4
^^^^^^^^^^^^^^^^^^^^^^^^^

¿Para que sirve la partición Partition Table?

Vuelve a ejecutar estos comandos:

.. code-block:: bash

    esptool.py read_flash 0x8000 0xc00 ptable.img
    gen_esp32part.py ptable.img

¿Qué partición estas leyendo e interpretando?

Ejercicio 7: Over-The-Air update
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ahora vas a aprender cómo funciona el mecanismos de OTA que ofrece el esp-idf.
Lee la sección OTA Mechanism `aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/firmwareupgrade.html#ota-mechanism>`__.

* ¿Qué particiones se están definiendo?
* ¿De que tipo y subtipo es la partición OTA-Data?
* ¿Para qué sirve la partición OTA-Data?
* ¿Por qué es importante tener dos particiones OTA (ota_0, ota_1)?

Ejercicio 8: Over-The-Air update
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el proyecto ``6_ota``. 

* No olvides configurar los puertos 
  de entrada-salida correspondientes a tu hardware usando el archivo ``board_esp32_devkitc.h``.
* No olvides guardar los certificados en la carpeta clud_cfg.

Ejercicios 9: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ten PRESENTE que una vez ejecutes el proyecto, la aplicación CAMBIARÁ y ya no podrás 
cambiar de nuevo el firmware usando OTA porque la nueva aplicación no tiene considerada esta 
funcionalidad.

.. warning:: CONFIGURA correctamente el END-POINT

   No olvides configurar TU END-POINT, en el sitio del fabricante el end-point 
   utilizado es del propio fabricante. 

Una vez programes el código del proyecto, puedes abrir un cliente curl y ejecutar 
la siguiente línea, PERO editando el comando con tus datos.

.. code-block:: bash

    curl -d '{"state":{"desired":{"ota_url":"https://raw.githubusercontent.com/wiki/espressif/esp-jumpstart/images/hello-world.bin"}}}' --tlsv1.2 --cert device.cert --key device.key https://TU-ENDPOINT:8443/things/TU-DEVICEID/shadow | python -mjson.tool

Nota que estamos enviando desde CURL la URL donde estará el archivo .bin con el 
ejecutable tal como explican `aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/firmwareupgrade.html#send-firmware-upgrade-url>`__.

Ejercicio 10: SOLO PARA LOS MÁS CURIOSOS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Te dejo un enlace para que repases y profundices un poco más sobre OTA 
`aquí <https://medium.com/the-esp-journal/ota-updates-framework-ab5438e30c12>`__.
(Si no te abre el enlace intenta en modo incógnito).

Ejercicio 11: SOLO PARA LOS MÁS CURIOSOS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

¿Cómo puedes obtener el certificado del servidor donde almacenarás la imagen de tu 
programa para actualizar el firmware del ESP32?

En `este <https://docs.espressif.com/projects/esp-jumpstart/en/latest/security.html#obtaining-ca-certificates>`__ 
enlace puedes ver cómo obtener ese certificado.

Ejercicio 12: SOLO PARA LOS MÁS CURIOSOS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

¿Cómo utilizar `postman <https://www.postman.com/downloads/>`__ para realizar los request a 
AWS IoT Core y poder enviar órdenes al dispositivo?

Lo puedes ver en `este <https://youtu.be/-kLtkkD-2uo>`__ video.

Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.