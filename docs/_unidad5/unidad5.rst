Unidad 5: configuración de la red WiFi
======================================

Sesión 1
-----------

En esta sesión analizaremos la estructura del proyecto 
``4_network_config``.

Ejercicios
-----------

Ejercicios 1: introducción al proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el proyecto ``4_network_config``. No olvides configurar los puertos 
de entrada-salida correspondientes a tu hardware usando el archivo 
``board_esp32_devkitc.h``.

Lee la descripción de este proyecto `aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/networkconfig.html>`__. 

Ejercicios 2: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Para que este proyecto se pueda conectar a tu red WiFi vas a tener que configurar las 
credenciales utilizando un método de ``PRODUCCIÓN``, es decir, como lo haría un usuario 
que reciba tu producto IoT.

Y si el usuario quiere conectar el dispositivo a otra red WiFi ¿Qué hacer? de esta 
pregunta también se ocupa este proyecto.

Por tanto, este proyecto te permitirá aprender:

* Cómo configurar y almacenar de manera permanente las credenciales de 
  tu red WiFi.
* Cómo borrar las credenciales almacenadas para configurar una nueva red.

Por lo pronto:

* Compila el proyecto.
* Ingresa el comando ``idf.py erase_flash`` para borrar por completo la memoria 
  flash del controlador.
* Graba el programa en la memoria y lanza el monitor, el resultado debe ser este:

.. code:: bash

    ...
    I (858) wifi_init: rx ba win: 6
    I (858) wifi_init: tcpip mbox: 32
    I (868) wifi_init: udp mbox: 6
    I (868) wifi_init: tcp mbox: 6
    I (868) wifi_init: tcp tx win: 5744
    I (878) wifi_init: tcp rx win: 5744
    I (878) wifi_init: tcp mss: 1440
    I (888) wifi_init: WiFi IRAM OP enabled
    I (888) wifi_init: WiFi RX IRAM OP enabled
    I (898) wifi_prov_scheme_ble: BT memory released
    I (898) app_main: Starting provisioning
    I (998) phy: phy_version: 4500, 0cd6843, Sep 17 2020, 15:37:07, 0, 0
    I (998) wifi:mode : sta (30:ae:a4:d3:83:20)
    I (998) BTDM_INIT: BT controller compile version [d886f91]
    I (1358) wifi_prov_mgr: Provisioning started with service name : PROV_D38320 
    I (1358) app_main: Provisioning started

En este punto el ESP32 se quedará esperando a que lo provisiones o configures 
las credenciales WiFi. Por defecto el programa iniciará el provisionamiento 
utilizando `Bluetooth Low Energy <https://docs.espressif.com/projects/esp-jumpstart/en/latest/networkconfig.html#ble>`__.

Para provisionar la aplicación con las credenciales de tu WiFi vas a necesitar 
descargar una aplicación móvil. En `este <https://docs.espressif.com/projects/esp-jumpstart/en/latest/networkconfig.html#demo>`__ 
enlace puedes descargar una versión para Android o para iOS. Sigue las instrucciones de la 
sección ``DEMO``.


Ejercicios 3: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ahora vamos a probar el otro método: SoftAP.

* Modifica el código fuente en app_main.c en la función app_main() así:

.. code:: c 

    wifi_prov_mgr_config_t config = {
        /* What is the Provisioning Scheme that we want ?
         * wifi_prov_scheme_softap or wifi_prov_scheme_ble */

        //.scheme = wifi_prov_scheme_ble,
        .scheme = wifi_prov_scheme_softap,

        /* Any default scheme specific event handler that you would
         * like to choose. Since our example application requires
         * neither BT nor BLE, we can choose to release the associated
         * memory once provisioning is complete, or not needed
         * (in case when device is already provisioned). Choosing
         * appropriate scheme specific event handler allows the manager
         * to take care of this automatically. This can be set to
         * WIFI_PROV_EVENT_HANDLER_NONE when using wifi_prov_scheme_softap*/

        //.scheme_event_handler = WIFI_PROV_SCHEME_BLE_EVENT_HANDLER_FREE_BTDM,
        .scheme_event_handler = WIFI_PROV_EVENT_HANDLER_NONE,

        /* Do we want an application specific handler be executed on
         * various provisioning related events */
        .app_event_handler = {
            .event_cb = prov_event_handler,
            .user_data = NULL
        }
    };

* No olvides borrar por completo la memoria flash del microcontrolador 
  para garantizar que las credenciales que guardamos usando el método anterior 
  se borren:

.. code:: bash

    idf.py -p /dev/ttyUSB0 -b 921600 erase_flash build flash monitor

Debes ver en el monitor algo similar a esto:

.. code:: bash

    I (989) wifi:Total power save buffer number: 16
    W (989) wifi_prov_scheme_softap: Error adding mDNS service! Check if mDNS is running
    I (999) wifi_prov_mgr: Provisioning started with service name : PROV_D38320 
    I (1009) app_main: Provisioning started

* En este momento tu ESP32 se ha convertido en un access point al cual puedes 
  comunicarte usando tu celular. Conéctate al AP que provee el ESP32, en mi caso 
  será ``PROV_D38320``

* Una vez te conectes, verás algo similar a esto en el monitor:

.. code:: bash

    I (303949) wifi:new:<1,0>, old:<1,1>, ap:<1,1>, sta:<0,0>, prof:1
    I (303949) wifi:station: a8:9c:ed:d0:e8:e4 join, AID=1, bgn, 20
    I (304119) esp_netif_lwip: DHCP server assigned IP to a station, IP is: 192.168.4.2
    W (304369) httpd_uri: httpd_uri: URI '/' not found
    W (304369) httpd_txrx: httpd_resp_send_err: 404 Not Found - This URI does not exis

* Ahora descarga la APP adecuada para tu sistema operativo móvil:

    * `Android <https://play.google.com/store/apps/details?id=com.espressif.provsoftap>`__.
    * `iOS <https://apps.apple.com/in/app/esp-softap-provisioning/id1474040630>`__.

* Sigue las instrucciones en la aplicación.

* Una vez termines de provisionar la aplicación reinicia el ESP32 y verás que ahora 
  se conecta directamente a tu red WiFi:

.. code:: bash 

    ...
    I (749) app_main: Already provisioned, starting Wi-Fi STA
    I (839) phy: phy_version: 4500, 0cd6843, Sep 17 2020, 15:37:07, 0, 0
    I (839) wifi:mode : sta (30:ae:a4:d3:83:20)
    I (969) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1
    I (969) wifi:state: init -> auth (b0)
    I (979) wifi:state: auth -> assoc (0)
    I (979) wifi:state: assoc -> run (10)
    I (1089) wifi:connected with funcholandia, aid = 1, channel 6, BW20, bssid = c0:89:ab:e5:8f:61
    I (1089) wifi:security: WPA2-PSK, phy: bgn, rssi: -39
    I (1089) wifi:pm start, type: 1

    I (1109) wifi:AP's beacon interval = 102400 us, DTIM period = 1
    I (2129) esp_netif_handlers: sta ip: 192.168.1.1, mask: 255.255.255.0, gw: 192.168.1.254
    I (2129) app_main: Connected with IP Address:192.168.1.1

Nota algo interesante en el código:

.. code:: c

         *      - WIFI_PROV_SECURITY_1 is secure communication which consists of secure handshake
         *          using X25519 key exchange and proof of possession (pop) and AES-CTR
         *          for encryption/decryption of messages.
         */
        wifi_prov_security_t security = WIFI_PROV_SECURITY_1;

La comunicación entre el ESP32 en modo AP y la aplicación móvil se asegura. Recuerda que todos 
estos elementos están pensados para ``PRODUCCIÓN``.

Ejercicios 4: protocolo de provisionamiento
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Los dos DEMOS anteriores son posibles gracias a un mecanismo de provisionamiento que 
viene incluido en el ESP-IDF: 
`Unified Provisioning <https://docs.espressif.com/projects/esp-jumpstart/en/latest/networkconfig.html#unified-provisioning>`__. 

Nota, que el código fuente de las aplicaciones móvil está disponible. Esto es un 
excelente punto de partida para el ingeniería encargado de desarrollar la aplicación 
móvil personalizada para tu producto.

Ejercicios 5: código de la aplicación
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el archivo app_driver.c. Nota que el programa se ha modificado para incluir 
la capacidad de borrar la credenciales usando el pulsador:

.. code:: c 

    static void button_press_3sec_cb(void *arg)
    {
        nvs_flash_erase();
        esp_restart();
    }

    static void configure_push_button(int gpio_num, void (*btn_cb)(void *))
    {
        button_handle_t btn_handle = iot_button_create(JUMPSTART_BOARD_BUTTON_GPIO, JUMPSTART_BOARD_BUTTON_ACTIVE_LEVEL);
        if (btn_handle) {
            iot_button_set_evt_cb(btn_handle, BUTTON_CB_RELEASE, btn_cb, "RELEASE");
            iot_button_add_on_press_cb(btn_handle, 3, button_press_3sec_cb, NULL);
        }
    }

Nota la función:

.. code:: c 

    iot_button_add_on_press_cb(btn_handle, 3, button_press_3sec_cb, NULL);

Esta función te permitirá detectar, como ya sabes, si el botón lleva presionado al 
menos 3 segundos. Si es es el caso observa que ocurrirán 2 cosas:

.. code:: c

    static void button_press_3sec_cb(void *arg)
    {
        nvs_flash_erase();
        esp_restart();
    }

Primero, se borrará la zona de memoria flash que contiene las credenciales WiFi. Esta 
zona es la partición NVS. Más adelanta discutiremos sobre las particiones de la 
memoria flash cuando actualicemos el firmware del ESP32 de manera remota. Segundo, 
se reiniciará el ESP32 y entrará de nuevo a modo provisioning.

Ejercicios 6: código de la aplicación
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.