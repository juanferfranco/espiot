Unidad 3: drivers
===================

Sesión 1
-----------

En esta sesión vamos a analizar el proyecto 2_drivers que hace parte del proyecto de curso.

Algunos puntos ha tener presentes:

* Se incluye la función ``app_driver_init()`` al código base que ya teníamos.
* En la carpeta main se modifica el archivo CMakeLists.txt para incluir en el proceso 
  de construcción otros archivos .c y se define un macro como parte de la línea 
  de comandos del compilador: 
  
  .. code-block:: bash

        set(JUMPSTART_BOARD "board_esp32_devkitc.h") 
        component_compile_options("-DJUMPSTART_BOARD=\"${JUMPSTART_BOARD}\"")
  
  El macro definido es JUMPSTART_BOARD que apunta al archivo ``board_esp32_devkitc.h``
* La mayor parte del código para este proyecto estará en el archivo ``app_driver.c`` y en el 
  componente ``button`` que está en la carpeta components de la carpeta raíz del 
  proyecto de curso.
* El componente ``button`` se añade al proyecto en el archivo CMakeLists.txt que está 
  en la raíz del directorio del proyecto.

  .. code-block:: bash

        set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/../components)

* En esta versión del proyecto ya podremos leer la entrada y escribir la salida. En el archivo 
  ``board_esp32_devkitc.h`` se deben definir en qué pines estará el pulsador y el LED y 
  cuál es el nivel del pulsador al ser presionado, es decir, cuál es el nivel activo.

  .. code-block:: c

        #define JUMPSTART_BOARD_BUTTON_GPIO          19
        #define JUMPSTART_BOARD_BUTTON_ACTIVE_LEVEL  0
        #define JUMPSTART_BOARD_OUTPUT_GPIO          5 

* El componente button es configurable utilizando ``menuconfig``.

* El código en ``app_driver.c`` asume que el LED se activa con la salida en alto. Si tu 
  LED se activa con bajo debes modificar el código que inicializa el LED para que comience 
  efectivamente apagado.


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
    :scale: 100%
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


Sesión 2
-----------
