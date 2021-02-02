Unidad 3: drivers
===================

Sesión 1
-----------

En esta sesión vamos a analizar el proyecto 2_drivers.

Algunos puntos ha tener presentes:

* Se incluye la función ``app_driver_init()`` al código base que ya teníamos.
* En la carpeta main se modifica el archivo CMakeLists.txt para incluir en el proceso 
  de construcción otros archivos .c y se define un macro como parte de la línea 
  de comandos del compilador: 
  
  .. code-block:: bash

        set(JUMPSTART_BOARD "board_esp32_devkitc.h") 
        component_compile_options("-DJUMPSTART_BOARD=\"${JUMPSTART_BOARD}\"")
  
  El macro definido es JUMPSTART_BOARD que apunta al archivo ``board_esp32_devkitc.h``
* La mayoría de código para este proyecto estará en el archivo ``app_driver.c`` y en el 
  componente ``button`` que está en la carpeta components de la carpeta raíz del 
  proyecto de curso.
* El componente ``button`` se añade al proyecto en el archivo CMakeLists.txt que está 
  en la raíz del directorio del proyecto.

  .. code-block:: bash

        set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/../components)

* En esta versión del proyecto ya podremos leer la entrada y escribir la salida. En el archivo 
  ``board_esp32_devkitc.h`` se deben definir en qué pines estará el pulsador y el LED y 
  cuál es el estado que reportará el pulsador cuando se presione.

  .. code-block:: c

        #define JUMPSTART_BOARD_BUTTON_GPIO          19
        #define JUMPSTART_BOARD_BUTTON_ACTIVE_LEVEL  0
        #define JUMPSTART_BOARD_OUTPUT_GPIO          5 

* El componente button es configurable utilizando ``menuconfig``.



Ejercicios
-----------

Ejercicios 1: 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sesión 2
-----------
