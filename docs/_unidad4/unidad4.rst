Unidad 4: conexión WiFi
======================================

Sesión 1
-----------

En esta sesión haremos lo siguiente:

Analizaremos la estructura del proyecto ``3_wifi_connection``.

Ejercicios
-----------

Ejercicios 1: 
^^^^^^^^^^^^^^^^

Abre el proyecto ``3_wifi_connection``. No olvides configurar los puertos 
de entrada-salida correspondientes a tu hardware usando el archivo 
``board_esp32_devkitc.h``.

Ejercicios 2: 
^^^^^^^^^^^^^^^^

Ahora abre el archivo ``app_main.c``. Nota que ahora hay más código en este 
archivo que hará las siguientes cosas:

* Inicia la partición NVS.
* Inicia el stack de comunicación TCP/IP.
* Inicia el event loop.
* Inicia el WiFi en modo estación.

Configura las constantes 

.. code:: c 

    #define EXAMPLE_ESP_WIFI_SSID EL_NOMBRE_DE_TU_RED_WIFI
    #define EXAMPLE_ESP_WIFI_PASS LA_CLAVE_DE_TU_RED

Ejecuta el programa y verifica que el ESP32 puede conectarse a tu red WiFi.

¿Qué dirección IP le asigna tu router al ESP32?




Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.
