Unidad 4: conexión WiFi
======================================

Sesión 1
-----------

En esta sesión haremos lo siguiente:

Analizaremos la estructura del proyecto ``3_wifi_connection``.

Ejercicios
-----------

Ejercicios 1: introducción al proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el proyecto ``3_wifi_connection``. No olvides configurar los puertos 
de entrada-salida correspondientes a tu hardware usando el archivo 
``board_esp32_devkitc.h``.

Lee la descripción de este proyecto `aquí <https://docs.espressif.com/projects/esp-jumpstart/en/latest/wifi.html>`__. 

Ejercicios 2: ejecución del proyecto
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Abre el archivo ``app_main.c``. Nota que ahora hay más código en este 
archivo que antes. El código modela los siguientes procesos:

* Inicia la partición NVS.
* Inicia el stack de comunicación TCP/IP.
* Inicia el event loop.
* Inicia el WiFi en modo estación.

Configura las constantes 

.. code:: c 

    #define EXAMPLE_ESP_WIFI_SSID EL_NOMBRE_DE_TU_RED_WIFI
    #define EXAMPLE_ESP_WIFI_PASS LA_CLAVE_DE_TU_RED

Ejecuta el programa y verifica que el ESP32 puede conectarse a tu red WiFi.

¿Qué dirección IP le asigna tu access point al ESP32?

Ejercicio 3: nvs
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Observa de nuevo el siguiente código en ``app_main.c``:

.. code:: c 

    ...
    /* Initialize NVS partition */
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        /* NVS partition was truncated
         * and needs to be erased */
        ret = nvs_flash_erase();

        /* Retry nvs_flash_init */
        ret |= nvs_flash_init();
    }
    if (ret != ESP_OK) {
        ESP_LOGE(TAG, "Failed to init NVS");
        return;
    }
    ...

Este código llama la función ``nvs_flash_init``. ¿Para qué? Para inicializar la biblioteca 
Non-volatile storage (`NVS <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/storage/nvs_flash.html>`__). 
Este biblioteca sirve para guardar información de tipo CLAVE-VALOR en la memoria flash. Esta 
funcionalidad es utilizada por el `driver de WiFi <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/network/esp_wifi.html>`__. 

Ejercicio 4: modelo de programación del WiFi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

La siguiente figura, tomada de la documentación del fabricante, muestra el modelo de 
programación al usar el WiFi del ESP32:

.. image:: ../_static/wifi-prog-model.png
   :alt:  modelo de programación del WiFi del ESP32
   :scale: 100%
   :align: center


Nota que uno de los bloques es el stack de TCP/IP. Por tanto en ``app_main.c`` 
se inicializa:

.. code:: c

    /* Initialize TCP/IP and the event loop */
    tcpip_adapter_init();

Ahora mira en el diagrama que la biblioteca produce EVENTOS. Son precisamente los eventos 
la manera con la cual la biblioteca se comunica con tu aplicación.

De regreso a ``app_main.c``, la siguiente línea de código le dice a la biblioteca 
cuál será el callback de tu aplicación que deberá llamar cuando se produzcan los eventos:

.. code:: c

    ESP_ERROR_CHECK(esp_event_loop_init(event_handler, NULL) );

Mira el callback:

.. code:: c

    static esp_err_t event_handler(void *ctx, system_event_t *event)
    {
        switch (event->event_id) {
        case SYSTEM_EVENT_STA_START:
            esp_wifi_connect();
            break;
        case SYSTEM_EVENT_STA_GOT_IP:
            ESP_LOGI(TAG, "Connected with IP Address:%s", ip4addr_ntoa(&event->event_info.got_ip.ip_info.ip));
            break;
        case SYSTEM_EVENT_STA_DISCONNECTED:
            ESP_LOGI(TAG, "Disconnected. Connecting to the AP again...\n");
            esp_wifi_connect();
            break;
        default:
            break;
        }
        return ESP_OK;
    }

Nota que en este caso solo vamos a procesar estos eventos: SYSTEM_EVENT_STA_START, 
SYSTEM_EVENT_STA_GOT_IP, SYSTEM_EVENT_STA_DISCONNECTED; sin embargo, se podrían 
recibir `otros <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/wifi.html#esp32-wi-fi-event-description>`__ 
más.

Finalmente, observa que estamos iniciando el ESP32 en modo ``Station`` para poder conectarnos 
a un ``access point`` (el que tienes en tu casa).

.. code:: c

    /* Start the station */
    wifi_init_sta();

La función inicializa el WiFi con valores por defecto (``WIFI_INIT_CONFIG_DEFAULT``)
y en modo STATION. Configura la red a la cual se conectará el ESP32 y dará la 
orden iniciar en modo STATION. De este punto en adelante, la interacción 
con la biblioteca se realizará por medio del administrador de eventos: ``event_handler``.
Una vez la biblioteca configure correctamente el drive WiFi y el ESP32 en modo STATION,
se llamará un evento que permitirá finalmente conectarse a la red WiFi.

.. code:: c

    static void wifi_init_sta()
    {
        wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
        ESP_ERROR_CHECK(esp_wifi_init(&cfg));
        ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
        ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA) );

        wifi_config_t wifi_config = {
            .sta = {
                .ssid = EXAMPLE_ESP_WIFI_SSID,
                .password = EXAMPLE_ESP_WIFI_PASS,
            },
        };
        ESP_ERROR_CHECK(esp_wifi_set_config(ESP_IF_WIFI_STA, &wifi_config) );

        ESP_ERROR_CHECK(esp_wifi_start() );

        ESP_LOGI(TAG, "connect to ap SSID:%s password:%s",
                EXAMPLE_ESP_WIFI_SSID, EXAMPLE_ESP_WIFI_PASS);
    }

Ejercicio 5: EVENTOS
^^^^^^^^^^^^^^^^^^^^^^^^^^^

En el proyecto se están manejando 3 eventos, pero hay 
`más eventos <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/esp_event_legacy.html?highlight=system_event_sta_start#enumerations>`__ 
para informarle a la aplicación:

* SYSTEM_EVENT_STA_START: el ESP32 ya inició en modo STATION. Y ahora si puede 
  conectarse al ACCESS POINT (AP): ``esp_wifi_connect();``
* SYSTEM_EVENT_STA_GOT_IP: el ESP32 ya se conectó a un Access Point y este le asignó una dirección 
  ip.
* SYSTEM_EVENT_STA_DISCONNECTED: indica que el ESP32 se desconectó del AP.

Ejercicio 6: reto 1
^^^^^^^^^^^^^^^^^^^^

Adicionar un evento más que informe por el puerto serial si el ESP32 ya se 
conectó al AP.

Ejercicio 7: reto 2
^^^^^^^^^^^^^^^^^^^^

* Verifica que al desconectar tu AP de la energía, el ESP32 reporta que se desconectó.
* Vuelve a energizar tu AP y ahora verifica que el ESP32 reporte que está conectado 
  de nuevo.

Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.
