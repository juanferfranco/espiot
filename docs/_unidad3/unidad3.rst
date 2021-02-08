Unidad 3: drivers
===================

Sesión 1
-----------

En esta sesión haremos lo siguiente: 


* Vamos a comenzar con el análisis de la segunda parte del proyecto de curso: ``2_drivers``.

  * Ajustes para ejecutarlo.
  * Estructura del proyecto.

* Introducción a la técnica de modelado utilizando máquinas de estado.


Ejercicios
-----------

Ejercicios 1: ejercicio 6 de la unidad 2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Te dejo una posible solución al ejercicio 6 de la unidad 2 para que estudies con detenimiento:

.. code-block:: c
    :linenos:

      #include <stdio.h>
      #include "freertos/FreeRTOS.h"
      #include "freertos/task.h"
      #include "driver/gpio.h"
      #include "string.h"
      #include "esp_system.h"
      #include "esp_spi_flash.h"


      #define LED_PIN 5
      #define BUTTON_PIN 19


      void app_main(void)
      {

          uint8_t portLevel = 1;
          uint8_t mac[6];

          gpio_pad_select_gpio(LED_PIN);
          gpio_set_level(LED_PIN, portLevel);
          gpio_set_direction(LED_PIN, GPIO_MODE_OUTPUT);


          gpio_pad_select_gpio(BUTTON_PIN);
          gpio_set_direction(BUTTON_PIN, GPIO_MODE_INPUT);
          gpio_pullup_en(BUTTON_PIN);

          esp_chip_info_t chip_info;
          esp_chip_info(&chip_info);
          esp_efuse_mac_get_default(mac);

          int c;
          char message[16];
          uint8_t counter = 0; 

          while(1)
          {
              c = getchar();
              if(c != EOF)
              {
                  if( '\n' == c)
                  {
                      message[counter] = 0;
                      if ( strncmp (message, "ledOn", strlen("ledOn") ) == 0)
                      {
                          portLevel = 0;
                          gpio_set_level(LED_PIN, portLevel);
                          
                      }
                      else if( strncmp (message, "ledOff", strlen("ledOff") ) == 0 )
                      {
                          portLevel = 1;
                          gpio_set_level(LED_PIN, portLevel);
                      }
                      else if( strncmp (message, "readButton", strlen("readButton") ) == 0 )
                      {
                          printf("Button state is: %s\n", gpio_get_level(BUTTON_PIN) ? "UP" : "DOWN" );

                      }
                      else if( strncmp (message, "rev", strlen("rev") ) == 0 )
                      {
                          printf("Chip revision: %d\n", chip_info.revision);
                      }
                      else if( strncmp (message, "idf", strlen("idf") ) == 0 )
                      {
                          printf("idf version: %s\n", esp_get_idf_version());
                      }
                      else if( strncmp (message, "bt", strlen("bt") ) == 0 )
                      {
                          printf("Bluetooth support: %s\n", chip_info.features & CHIP_FEATURE_BT ? "si":"no");
                      }
                      else if( strncmp (message, "ble", strlen("ble") ) == 0 )
                      {
                          printf("BLE support: %s\n", chip_info.features & CHIP_FEATURE_BLE ? "si":"no");
                      }
                      else if( strncmp (message, "wifi", strlen("wifi") ) == 0 )
                      {
                          printf("Wifi support: %s\n", chip_info.features & CHIP_FEATURE_WIFI_BGN ? "si":"no");
                      }
                      else if( strncmp (message, "flash", strlen("flash") ) == 0 )
                      {
                          printf("flash size %dMB\n", spi_flash_get_chip_size() / (1024 * 1024)) ;
                      }
                      else if( strncmp (message, "mac", strlen("mac") ) == 0 )
                      {

                          printf("mac add: %02x:%02x:%02x:%02x:%02x:%02x\n", mac[0], mac[1],mac[2],mac[3],mac[4],mac[5]);
                      }


                      counter = 0;
                  }
                  else
                  {
                      if( counter < ( sizeof(message) - 1 ) )
                      {
                          message[counter] = c;
                          counter++;
                      }
                  }
              }
              vTaskDelay(100/portTICK_PERIOD_MS);
          }
      }

Ejercicio 2: 23 de la unidad 2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Estudia con mucho cuidado esta solución al ejercicio 23 de la unidad 2. Aquí te presento 
a manera de introducción una técnica de modelado de software conocida como máquinas de 
estado.

La siguiente figura muestra un posible modelo de la solución al problema:

.. image:: ../_static/u2-ej23-state.png
    :scale: 75%
    :align: center
    :alt: diagrama de estados de una solución

Y una posible implementación de la máquina de estados es esta:

.. code-block:: c
    :linenos:

      #include <stdio.h>
      #include "freertos/FreeRTOS.h"
      #include "freertos/task.h"
      #include "driver/gpio.h"
      #include "string.h"
      #include "esp_system.h"
      #include "esp_spi_flash.h"

      int suma(int a, int b);
      int resta(int a, int b);
      int multi(int a, int b);
      char * readSerialString(void);


      typedef enum {
          INIT = 0,
          WAITING_OP1,
          WAITING_OP2,
          WAITING_FUNC,
      } app_state_t;


      void app_main(void)
      {
          static app_state_t appState = INIT;
          static int op1;
          static int op2;

          while(1)
          {
              switch(appState)
              {
                  case INIT: 
                  {
                      printf("Enter op1 as integer: \n");
                      appState = WAITING_OP1;
                      break;
                  }

                  case WAITING_OP1:
                  {
                      char *pString = readSerialString();
                      if(pString != NULL)
                      {
                        uint8_t status = sscanf(pString,"%d",&op1);
                        if(status == 1)
                        {
                            appState = WAITING_OP2;
                            printf("%d\n", op1);
                            printf("Enter op2 as integer: \n");
                        }
                        else
                        {
                            printf("Bad op1. Enter an int\n");
                            printf("Enter op1 as integer: \n");
                        }
                      }
                      
                      break;
                  }

                  case WAITING_OP2:
                  {
                      char *pString = readSerialString();
                      if(pString != NULL)
                      {
                        uint8_t status = sscanf(pString,"%d",&op2);
                        if(status == 1)
                        {
                            appState = WAITING_FUNC;
                            printf("%d\n", op2);
                            printf("Enter +,-,*: \n");
                        }
                        else
                        {
                            printf("Bad op2. Enter an int\n");
                            printf("Enter op2 as integer: \n");
                        }
                      }
                      
                      break;
                  }

                  case WAITING_FUNC:
                  {
                      char *pString = readSerialString();
                      if(pString != NULL)
                      {   
                        char func;
                        uint8_t status = sscanf(pString,"%c",&func);
                        if(status == 1)
                        {
                            printf("%c\n", func);

                            int (*pfunction)(int, int) = NULL;

                            if('+' == func)
                            {
                                pfunction = suma;
                            }
                            else if('-' == func)
                            {
                                pfunction = resta; 
                            }
                            else if ('*' == func)
                            {
                                pfunction = multi;
                            }

                            if(pfunction != NULL)
                            {
                                  printf("Resultado: %d %c %d = %d\n", op1,func, op2, pfunction(op1,op2));  
                                  printf("Enter op1 as integer: \n");
                                  appState = WAITING_OP1;  
                            } 
                            else 
                            {
                                  printf("Bad function\n");
                                  printf("Enter +,-,*: \n");
                            }
                        }
                      }

                      break;
                  }

                  default:
                  {
                      printf("State machine error \n");
                      break;
                  }
        
              }

              vTaskDelay(100/portTICK_PERIOD_MS);
          }
      }

      int suma(int a, int b)
      {
          return a + b;
      }

      int resta(int a, int b)
      {
          return a - b;
      }

      int multi(int a, int b)
      {
          return a*b;
      }

      char * readSerialString(void)
      {
          static char message[16];
          static uint8_t counter = 0;
          char *returnValue = NULL;

          int c = getchar();

          if (c != EOF)
          {
              if ('\n' == c)
              {
                  message[counter] = 0;
                  returnValue = message;
                  counter = 0;
              }
              else
              {
                  if (counter < (sizeof(message) - 1))
                  {
                      message[counter] = c;
                      counter++;
                  }
              }
          }
          return returnValue;
      }


Ejercicio 3: estructura de ``2_drivers``: CMakeLists.txt
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Hagamos una exploración de partes del proyecto ``2_drivers``: 

* En la carpeta main se modifica el archivo CMakeLists.txt para incluir en el proceso 
  de construcción otros archivos .c 
  
  .. code-block:: bash

      set(COMPONENT_SRCS "app_main.c"
              "app_driver.c"
      )

  Ten en cuenta que la propia carpeta main es UN COMPONENTE, no olvides que una aplicación 
  para el ESP32 utilizando el esp-idf no es más que una colección de componentes con los 
  cuales se genera el ejecutable que grabaremos en la memoria del microcontrolador.

  En este caso, en el CMakeLists.txt de main estás indicando que el componente 
  tiene dos archivos .c: ``app_main.c`` y ``app_driver.c``

* ¿Cómo se transforman los archivos .c de una aplicación a un ejecutable que será almacenado 
  en la memoria del microcontrolador? Esta es una pregunta a la que podrías dedicarle un bueno rato; 
  sin embargo, te cuento rápidamente cómo es el proceso para que podamos seguir avanzando. Mira 
  con detenimiento la siguiente figura que muestra los pasos:

  .. image:: ../_static/c-build-pipe.png
      :scale: 75%
      :align: center
      :alt: flujo de compilación en c

|

  Como puedes ver, el proceso se compone de 4 pasos. Primero, el preprocesador procesa todas 
  las DIRECTIVAS. En la figura, el archivo ``archivo.c`` tiene la directiva ``#include``. Nota 
  que el preprocesador simplemente genera un nuevo archivo intermedio que contiene la información 
  de ``archivo.c`` y el contenido ``archivo2.h``. Segundo, el archivo de salida del preprocesador 
  es compilado y se genera código ensamblador, que no es más que una representación simbólica 
  del lenguaje de máquina. Tercero, el archivo se ensambla, es decir, se transforma de lenguaje 
  de máquina simbólico a lo que usualmente denominamos unos y ceros. Mira en la figura de nuevo 
  el contenido del archivo de salida de la fase de ensamblado. Observa esta línea: 

  .. code-block:: bash

      017d  mov.n a7, a1

  ``017d`` es la representación binaria de la instrucción en lenguaje ensamblador ``mov.n a7, a1``. Finalmente, 
  el cuarto paso es el enlazado. El enlazador toma TODOS los archivo ensamblados del proyecto, los combina 
  y genera el ``archivo_ejecutable`` que grabaremos en la memoria del microcontrolador.

* Volvamos al archivo ``CMakeLists.txt`` del componente ``main``. Nota las siguientes líneas:

  .. code-block:: bash

        set(JUMPSTART_BOARD "board_esp32_devkitc.h") 
        component_compile_options("-DJUMPSTART_BOARD=\"${JUMPSTART_BOARD}\"")

  ``set(JUMPSTART_BOARD "board_esp32_devkitc.h")`` crea la constante ``JUMPSTART_BOARD`` de tal manera 
  que en otras partes del archivo ``CMakeLists.txt`` podamos usar ``JUMPSTART_BOARD`` en vez de ``board_esp32_devkitc.h``.
  Observa que ``board_esp32_devkitc.h`` está en la carpeta main y contiene información específica del sistema 
  de desarrollo que estamos utilizando como los puertos del pulsador y del LED y 
  cuál es el nivel lógico que produce el pulsador al ser presionado, es decir, cuál es el nivel lógico del pulsador 
  al activarse. En mi caso el LED estará en el pin 5, el pulsador en el pin 19 y el estado activo del pulsador será 0.

  .. code-block:: c

        #define JUMPSTART_BOARD_BUTTON_GPIO          19
        #define JUMPSTART_BOARD_BUTTON_ACTIVE_LEVEL  0
        #define JUMPSTART_BOARD_OUTPUT_GPIO          5 

  Nota también la línea ``component_compile_options("-DJUMPSTART_BOARD=\"${JUMPSTART_BOARD}\"")``. Esta información 
  se la pasaremos al COMPILADOR cuando compile el componente ``main``.

  Para que entiendas mucho mejor lo anterior te voy a explicar con un ejemplo sencillo. Considera este código:

  .. code-block:: c
      :linenos:

      #include INCLUDE

      void app_main()
      {
          int c = suma(VALOR1,VALOR2);
      }
      
  Nota que no estamos indicando en el propio código qué es ``INCLUDE``, ``VALOR1`` y ``VALOR2``. Cuando compilemos 
  este programa tendremos un error. Sin embargo, es posible indicarle al compilador qué valor tendrán esas constantes. 
  Si estuviéramos llamando explícitamente al preprocesador haríamos esto:

  .. code-block:: bash

      xtensa-esp32-elf-gcc -DVALOR1=2 -DVALOR2=3 -DINCLUDE=\"archivo2.h\" -E archivo.c

  Con este comando le decimos qué valores tendrán ``INCLUDE``, ``VALOR1`` y ``VALOR2``. Una vez preprocesado el 
  archivo tendremos esto:

  .. code-block:: c
      :linenos:

      int suma(int a, int b);

      void app_main()
      {
          int c = suma(2,3);
      }

  Ten presente que al construir el código del componente no tenemos que llamar manualmente al preprocesador porque 
  al hacer ``idf.py build`` todo el proceso ocurre de manera automática por nosotros ¡HERMOSO! 
  ¿Ahora vez lo que estamos haciendo con ``component_compile_options("-DJUMPSTART_BOARD=\"${JUMPSTART_BOARD}\"")``?

* Ahora vamos para el archivo CMakeLists.txt en el directorio principal del proyecto:

  .. code-block:: c

      # The following lines of boilerplate have to be in your project's
      # CMakeLists in this exact order for cmake to work correctly
      cmake_minimum_required(VERSION 3.5)

      set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/../components)

      include($ENV{IDF_PATH}/tools/cmake/project.cmake)
      project(2_drivers)

  Nota que solo hay una novedad con respecto al proyecto de la unidad anterior: 
  ``set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/../components)``. Este comando 
  permite adicionar componentes extra al proyecto. En este caso, estamos utilizando 
  el componete button que está en la carpeta ``components`` ubicada en el directorio 
  padre del directorio del proyecto.


Ejercicio 4: configuración de componentes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Recuerda que una aplicación para el ESP32 basada en el esp-idf es una combinación de 
`componentes <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/build-system.html#overview>`__. En 
el proyecto de esta unidad estamos utilizando varios componentes, entre ellos el componente button. ¿Qué es un 
componente? Según Espressif, un componente es una pieza modular de código independiente que se compila como una biblioteca 
estática y es enlazada, en el proceso de ENLACE, en la aplicación. ¿Recuerdas la figura del ejercicio anterior donde 
se ven los pasos de transformación del código fuente al archivo ejecutable? Pues bien, en el último paso correspondiente al enlazado, 
además de los archivos ``.o`` el enlazador puede tomar también archivos ``.a``. Los archivos ``.a`` son colecciones 
de archivos ``.o`` denominados bibliotecas estáticas. Ahora, los componentes SE PUEDEN CONFIGURAR. Veamos cómo funciona.


Considera el siguiente código del componente button definido en ``button.c``:

.. code-block:: c
    :linenos:

    #define BUTTON_GLITCH_FILTER_TIME_MS   CONFIG_IO_GLITCH_FILTER_TIME_MS
    static const char *TAG = "button";

La constante ``#define BUTTON_GLITCH_FILTER_TIME_MS`` está definida como ``CONFIG_IO_GLITCH_FILTER_TIME_MS``; sin embargo,
¿En dónde está definido ``CONFIG_IO_GLITCH_FILTER_TIME_MS``? Esa constante se GENERA en el proceso de construcción 
del ejecutable y se da incluso antes del proceso de PREPROCESADO. Esto se consigue gracias al archivo ``Kconfig`` que tienen 
aquellos componentes configurables. Por ejemplo, para el caso del componente button este es el archivo  ``Kconfig``:

.. code-block:: bash

    menu "Button"
        config IO_GLITCH_FILTER_TIME_MS
            int "IO glitch filter timer ms (10~100)"
            range 10 100
            default 50
    endmenu

Al realizar el proceso de construcción del ejecutable, las constantes definidas en los archivos ``Kconfig`` de todos 
los componentes son generadas y agrupadas en el archivo ``sdkconfig`` que queda en el directorio raíz del proyecto. 
Esto permite que el desarrollador pueda verificar la configuración de los componentes. Adicionalmente, en la carpeta ``build/config``  
se generará el archivo sdkconfig.h con las constantes ya listas para ingresar a la fase de PREPROCESADO. Para configurar 
cada componente se ejecuta el siguiente comando:

.. code-block:: bash

    idf.py menuconfig

Volviendo al componente button. Su archivo ``Kconfig`` define varios asuntos:

* ``menu "Button"``: crea una entrada para configurar el componente button con ``menuconfig``.
* ``config IO_GLITCH_FILTER_TIME_MS``: define la constante a configurar del componente.
* ``int "IO glitch filter timer ms (10~100)"``: la constante será tipo ``int`` y da una descripción.
* ``range 10 100``: los posibles valores que puede tomar la constante.
* ``default 50``: es el valor que tendrá por defecto la constante si no se configura.

Luego de configurar así quedan parte de los archivos.

``sdkconfig``:

.. code-block:: bash

    ...
    #
    # Button
    #
    CONFIG_IO_GLITCH_FILTER_TIME_MS=50
    # end of Button
  

``sdkconfig.h``:

.. code-block:: bash

    ...

    #define CONFIG_WIFI_PROV_AUTOSTOP_TIMEOUT 30
    #define CONFIG_WPA_MBEDTLS_CRYPTO 1
    
    #define CONFIG_IO_GLITCH_FILTER_TIME_MS 50
    
    #define CONFIG_AWS_IOT_MQTT_HOST ""
    #define CONFIG_AWS_IOT_MQTT_PORT 8883
    ...


* El código en ``app_driver.c`` asume que el LED se activa con la salida en alto. Si tu 
  LED se activa con bajo debes modificar el código que inicializa el LED para que comience 
  efectivamente apagado.

* En mi caso, la siguiente figura muestra el montaje que utilizaré para el proyecto de curso:

  .. image:: ../_static/unit3-circuit.png
      :scale: 75%
      :align: center
      :alt: montaje para el curso

* El archivo ``app_priv.h`` tiene el API (application programming interface) del DRIVER. El driver 
  es la parte de código específica de la aplicación que interactúa con los puertos de 
  entrada/salida del ESP32. Como nota personal, la palabra ``priv`` en ``app_priv.h`` resulta 
  infortunada porque realmente debería ``pub``, de pública, ya que las funciones que están allí 
  son las que podemos usar desde otros archivos.

  .. code-block:: c
      :linenos:

      void app_driver_init(void);
      int app_driver_set_state(bool state);
      bool app_driver_get_state(void);
  
* En el archivo ``app_priv.h`` nota la directiva del preprocesador ``#pragma once`` de la que 
  puedes leer `aquí <https://en.wikipedia.org/wiki/Pragma_once>`__.

* Observa el código ``app_main.c``:

  .. code-block:: c
      :linenos:

        #include <stdio.h>
        #include <freertos/FreeRTOS.h>
        #include <freertos/task.h>
        #include "app_priv.h"


        void app_main()
        {
            int i = 0;
            app_driver_init();
            while (1) {
                printf("[%d] Hello world!\n", i);
                i++;
                vTaskDelay(5000 / portTICK_PERIOD_MS);
            }
        }

  Nota la línea ``#include "app_priv.h"``. Esta línea te permitirá utilizar las tres funciones públicas 
  declaradas en ``app_priv.h``. En este caso, la aplicación solo está llamando una de las tres 
  ``app_driver_init()``.

* ``app_driver.c`` utiliza el componente button. Para ello incluye el archivo ``#include <iot_button.h>``. 
  De nuevo, ``iot_button.h`` tiene el API pública del componente button.



Sesión 2
-----------

En esta sesión vamos a resolver dudas sobre los ejercicios y escuchar aportes, 
comentarios y/o experiencias de todos.