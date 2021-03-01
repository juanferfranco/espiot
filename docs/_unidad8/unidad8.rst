Unidad 8: fabricación
=======================

Sesión 1
-----------

En esta sesión analizaremos la estructura del proyecto ``7_mfg``.


Ejercicios
-----------

Ejercicios 1: introducción al proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Finalmente hemos llegado al final de nuestro recorrido. Este proyecto permitirá 
crear una nueva partición donde serán almacenados los datos particulares 
de cada uno de los dispositivos IoT de un lote de PRODUCCIÓN.

Para comenzar, lee la descripción de este proyecto 
`aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/manufacturing.html#manufacturing>`__.

Ejercicio 2: mini reto 1
^^^^^^^^^^^^^^^^^^^^^^^^^^^

En este proyecto ¿Por qué es importante crear una nueva partición para guardar 
datos individuales de cada dispositivo IoT de un lote de producción? recuerda la funcionalidad 
de presionar el botón por 3 segundos.

Ejercicios 3: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el proyecto ``7_mfg``. No olvides configurar los puertos 
de entrada-salida correspondientes a tu hardware usando el archivo ``board_esp32_devkitc.h``.

Copia también los certificados.

Observa:

* El archivo ``partitions.csv`` se modificó para incluir una partición tipo NVS extra 
  denominada ``fctry``.

* El archivo ``cloud_aws.c``. La función ``cloud_start`` se modificó para cargar 
  desde la partición ``fctry`` los valores serial_no, cert (del dispositivo) y priv_key (
  la clave privada del dispositivo). Nota que serial_no corresponde al device_id 

¿Qué archivos se cargarían directamente en la imagen del programa y cuáles 
van en la partición ``fctry``?

Graba la aplicación en la memoria flash del micro. 

Ejercicios 4: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

En el ejercicio anterior, grabaste la aplicación en la memoria flash del microcontrolador; 
sin embargo, aún no has almacenado los archivos de fabricación en la partición ``fctry``. 
Para hacerlo debes tener la información que quieres guardar en un archivo ``.csv`` (mfg_config.csv), 
luego convertir ese archivo en una imagen binaria y finalmente grabar la partición con la 
información.

En `este <https://docs.espressif.com/projects/esp-jumpstart/en/latest/manufacturing.html#generating-the-factory-data>`__ 
enlace están los pasos para hacerlo. Ten presente que debes cambiar la variable 
``$ESPPORT`` por el puerto donde tienes conectado el microcontrolador.

Hechos los ejercicios anteriores, podrás controlar remotamente el ESP32, así como realizar 
el proceso de OTA.

Ejercicios 5: RETO
^^^^^^^^^^^^^^^^^^^^^

¿Recuerdas cuando hablamos de la Proof of Possession o pop? 

¿Qué modificaciones le harías al programa para almacenar la clave de pop en la partición de 
fabricación?

¿Te animas a realizar la implementación?

Ejercicios 6: SOLO PARA LOS MÁS CURIOSOS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Puedes profundizar un poco más en este `blog <https://medium.com/the-esp-journal/building-products-creating-unique-factory-data-images-3f642832a7a3>`__. 

Ejercicios 7: SOLO PARA LOS MÁS CURIOSOS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Puedes profundizar un poco más en los asuntos de seguridad en 
`este <https://docs.espressif.com/projects/esp-jumpstart/en/latest/security.html>`__ enlace. 

Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.