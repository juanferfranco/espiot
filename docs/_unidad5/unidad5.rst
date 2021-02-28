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

Y si el usuario, luego, quiere conectar el dispositivo a otra red WiFi 
¿Qué hacer? De esta pregunta también se ocupa este proyecto.

Por tanto, este proyecto te permitirá aprender:

* Cómo configurar y almacenar de manera permanente las credenciales de 
  tu red WiFi.
* Cómo borrar las credenciales almacenadas para configurar una nueva red.

Por lo pronto:

* Compila el proyecto.
* Ingresa el comando ``idf.py erase_flash`` para borrar por completo la memoria 
  flash del controlador.
* Graba el programa en la memoria y lanza el monitor, el resultado debe ser este:

.. code-block:: bash

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

  .. code-block:: c

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

  .. code-block:: bash

        idf.py -p /dev/ttyUSB0 -b 921600 erase_flash build flash monitor

  Debes ver en el monitor algo similar a esto:

  .. code-block:: bash

        I (989) wifi:Total power save buffer number: 16
        W (989) wifi_prov_scheme_softap: Error adding mDNS service! Check if mDNS is running
        I (999) wifi_prov_mgr: Provisioning started with service name : PROV_D38320 
        I (1009) app_main: Provisioning started

* En este momento tu ESP32 se ha convertido en un access point al cual puedes 
  comunicarte usando tu celular. Conéctate al AP que provee el ESP32, en mi caso 
  será ``PROV_D38320``

* Una vez te conectes, verás algo similar a esto en el monitor:

  .. code-block:: bash

        I (303949) wifi:new:<1,0>, old:<1,1>, ap:<1,1>, sta:<0,0>, prof:1
        I (303949) wifi:station: a8:9c:ed:d0:e8:e4 join, AID=1, bgn, 20
        I (304119) esp_netif_lwip: DHCP server assigned IP to a station, IP is: 192.168.4.2
        W (304369) httpd_uri: httpd_uri: URI '/' not found
        W (304369) httpd_txrx: httpd_resp_send_err: 404 Not Found - This URI does not exis

* Ahora descarga la APP adecuada para tu sistema operativo móvil:

    * `Android <https://play.google.com/store/apps/details?id=com.espressif.provsoftap>`__.
    * `iOS <https://apps.apple.com/in/app/esp-softap-provisioning/id1474040630>`__.

* Sigue las instrucciones en la aplicación.

* Una vez termines de provisionar la aplicación, reinicia el ESP32 y verás que ahora 
  se conecta directamente a tu red WiFi. 

Ejercicios 4: protocolo de provisionamiento
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Los dos DEMOS anteriores son posibles gracias a un mecanismo de provisionamiento que 
viene incluido en el ESP-IDF denominado Unified Provisioning. 

Nota que el código fuente de las aplicaciones móviles está disponible. Esto es un 
excelente punto de partida para el desarrollador encargado de implementar la aplicación 
móvil personalizada para tu producto.

.. warning:: La seguridad en IoT

    En `este <https://docs.espressif.com/projects/esp-jumpstart/en/latest/networkconfig.html#unified-provisioning>`__ 
    enlace puedes estudiar con más detalle cómo funcionan el mecanismo de Unified Provisioning.

    Ten presente que la seguridad es un tema de vital importancia en IoT.

Ejercicios 5: código de la aplicación
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el archivo app_driver.c. Nota que el programa se ha modificado para incluir 
la capacidad de borrar la credenciales usando el pulsador:

.. code-block:: c

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

.. code-block:: c

    iot_button_add_on_press_cb(btn_handle, 3, button_press_3sec_cb, NULL);

Esta función te permitirá detectar, como ya sabes, si el botón lleva presionado al 
menos 3 segundos. Si es es el caso observa que ocurrirán 2 cosas:

.. code-block:: c

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

Ahora vamos a concentrarnos en ``app_main.c``. Primero recordemos el código de la unidad 
anterior, pero a simplificado para entender los pasos más gruesos:

.. code-block:: c

    void app_main()
    {
        
        (1) app_driver_init();
        
        (2) esp_err_t ret = nvs_flash_init();
        (3) tcpip_adapter_init();
        (4) esp_event_loop_init(event_handler, NULL);
        (5) wifi_init_sta();
        
        while (1) 
        {
            ...
        }
    }

.. code-block:: c

    static void wifi_init_sta()
    {
        (6) wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
        (7) esp_wifi_init(&cfg);
        (8) esp_wifi_set_storage(WIFI_STORAGE_RAM);
        (9) esp_wifi_set_mode(WIFI_MODE_STA);

        (10) wifi_config_t wifi_config = {
            .sta = {
                .ssid = EXAMPLE_ESP_WIFI_SSID,
                .password = EXAMPLE_ESP_WIFI_PASS,
            },
        };
        (11) esp_wifi_set_config(ESP_IF_WIFI_STA, &wifi_config);
        (12) esp_wifi_start();
    }

Si analizas el código ``4_network_config`` verás que hasta el paso 
(7) tenemos lo mismo de la Unidad anterior. ¿Qué es la nuevo 
que debe soportar la aplicación?

Antes de configurar el WiFi en modo STATION y conectarnos a una 
red WiFi debes saber si tenemos o no las CREDENCIALES. Para 
saber si el programa está provisionado se llama esta función:

.. code-block:: c

    wifi_prov_mgr_is_provisioned(&provisioned);

La función guardará en la variable ``provisioned`` un valor booleano 
que indicará si el ESP32 tiene o no las credenciales.

Para poder llamar esta función es necesario que el MANEJADOR de 
provisionamiento de WiFi esté inicializado:


.. code-block:: c

    wifi_prov_mgr_init(config);

Si el ESP32 ya está provisionado, lo que pasemos en config realmente 
no importa, pero si no lo está, entonces pasaremos de una vez 
cómo queremos provisionar:

.. code-block:: c

    wifi_prov_mgr_config_t config = {
        .scheme = wifi_prov_scheme_ble,
        .scheme_event_handler = WIFI_PROV_SCHEME_BLE_EVENT_HANDLER_FREE_BTDM,
        .app_event_handler = {
            .event_cb = prov_event_handler,
            .user_data = NULL
        }
    };

Recuerda que tenemos los métodos BLE y SoftAP. ``prov_event_handler`` será 
el callback que utilizaremos para comunicar los eventos de ``wifi_prov_mgr`` 
con la aplicación:

Luego de llamar a ``wifi_prov_mgr_init(config)`` puede ocurrir una de dos cosas:

.. code-block:: c

    wifi_prov_mgr_init(config);

    if (!provisioned) 
    {
        ...
        (1) wifi_prov_mgr_start_provisioning(security, pop, service_name, service_key);
    }
    else
    {
        (2) wifi_prov_mgr_deinit();
        (3) wifi_init_sta();
    }

* (2) y (3): como el ESP32 ya tenía las credenciales entonces en (2) se liberan 
  la memoria reservada para el ``wifi_prov_mgr`` y en (3):

  .. code-block:: c

        esp_wifi_set_mode(WIFI_MODE_STA) );
        esp_wifi_start();

  Se configura el ESP32 en modo STATION y se intenta iniciarlo en ese modo. Recuerda 
  que luego de este momento, la comunicación con el driver WiFi se hará mediante 
  los callbacks definidos en ``event_handler``.

* (1): inicializar el proceso de provisionamiento. Recuerda que los eventos 
  producidos por ``wifi_prov_mgr`` en este caso se recibirán con ``prov_event_handler``:

Ejercicios 7: reto
^^^^^^^^^^^^^^^^^^^^^^^^^

Lee la sección `código del proyecto <https://docs.espressif.com/projects/esp-jumpstart/en/latest/networkconfig.html#the-code>`__ 
para profundizar un poco más en el funcionamiento y responder estas preguntas:

* ¿Para qué se llama la función `wifi_prov_mgr_init`?
* ¿Para qué sirve la partición NVS?
* ¿Se implementa en la aplicación algún mecanismo para borrar la información 
  en NVS?
* ¿Cómo hace la aplicación para saber si el ESP32 tiene credenciales almacenadas o no?
* ¿Cómo se selecciona el método de provisionamiento?
* Una vez el proceso de provisionamiento termina ¿Cómo se pueden liberar los recursos 
  de memoria que este utiliza?
* ¿El wifi_prov_mgr necesita conocer acerca de los eventos generados por el driver 
  de WiFi?
* ¿Con el ESP-IDF es posible brindar algún esquema de seguridad a la hora de enviar 
  el nombre de la red WiFi y la contraseña desde el celular hasta el ESP32? ¿Por qué crees 
  que esto sea importante?
* ¿Qué es la característica de prueba de posesión (``pop``) y para qué sirve? 
* Piensa cómo utilizarías la prueba de posesión en un ambiente real de producción, describe 
  cómo serían los pasos que seguiría un usuario de tu producto.
* Para implementar en producción ``pop`` ¿Qué le quedaría faltando al código de esta 
  unidad?

Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.

