# Interfaz de Keybus DSC
![dscKeybusInterface](https://user-images.githubusercontent.com/12835671/105620980-5b356380-5dc8-11eb-93c2-e813751dda8a.png)
Esta biblioteca permite la interfaz directa de microcontroladores de las series Arduino, esp8266 y esp32 con sistemas de seguridad [DSC PowerSeries](http://www.dsc.com/dsc-security-products/g/PowerSeries/4) y [Classic series](https://www.dsc.com/manual/29000203) para la integración con automatización del hogar, control remoto como un teclado virtual, notificaciones de eventos de alarma, desbloqueo de códigos de instalador y emulación de paneles DSC para usar teclados DSC como dispositivos de entrada de propósito general.

Esto permite que las instalaciones existentes del sistema de seguridad DSC conserven las características y la confiabilidad de un sistema cableado, mientras se integran con dispositivos y software modernos por menos de $5 USD en componentes.

Los ejemplos incorporados pueden usarse tal como están o como base para adaptarlos a otros usos:
* Integración con la automatización del hogar: [Home Assistant](https://www.home-assistant.io), [Apple HomeKit & Siri](https://www.apple.com/ios/home/), [Google Home](https://assistant.google.com), [OpenHAB](https://www.openhab.org), [Athom Homey](https://www.athom.com/en/)
* Control remoto: Interfaz web, aplicación móvil [Blynk](https://www.blynk.io), bot de [Telegram](https://www.telegram.org) (con armado/desarmado remoto a través del chat)
* Notificaciones: [Pushover](https://www.pushover.net), [PushBullet](https://www.pushbullet.com), [Pushsafer](https://www.pushsafer.com), [Twilio SMS](https://www.twilio.com), [TinyGSM SMS](https://github.com/vshymanskyy/TinyGSM), correo electrónico
* Interfaz de teclado: Emula un panel DSC para conectar teclados DSC como dispositivos de entrada física para cualquier propósito general, sin un panel DSC.
* Desbloqueo de código de instalador: Búsqueda automática de código para desbloquear paneles con códigos de instalador desconocidos

Integraciones de ejemplo:
* [Apple Home & Siri](https://www.apple.com/ios/home/):  
  ![HomeKit](https://user-images.githubusercontent.com/12835671/61570833-c9bb9780-aa54-11e9-9477-8e0853609e91.png)
* [Home Assistant](https://www.home-assistant.io):  
  ![HomeAssistant](https://user-images.githubusercontent.com/12835671/61985900-38f33780-afd1-11e9-9d43-ab0b681b7b03.png)
* [OpenHAB](https://www.openhab.org) MQTT:  
  ![OpenHAB](https://user-images.githubusercontent.com/12835671/61560425-daa6e180-aa31-11e9-9efe-0fcb44d2106a.png)
* Teclado virtual de la aplicación heredada de [Blynk](https://www.blynk.io):  
  ![VirtualKeypad-Blynk](https://user-images.githubusercontent.com/12835671/61568638-9fb0a800-aa49-11e9-94d0-e598431ea2ed.png)
* Teclado virtual web:  
  ![VirtualKeypad-Web](https://user-images.githubusercontent.com/12835671/61570049-e43f4200-aa4f-11e9-96bc-3448b6630990.png)
* Bot de [Telegram](https://www.telegram.org):  
  ![Telegram](https://user-images.githubusercontent.com/12835671/102932636-47fc4480-4466-11eb-844b-baa767b92157.png)


## Inicio rápido
1. Instala la biblioteca DSC Keybus Interface:
    * Arduino IDE: Busca `DSC` en el Administrador de Bibliotecas - `Sketch > Include Library > Manage Libraries`  
      ![ArduinoIDE](https://user-images.githubusercontent.com/12835671/41826133-cfa55334-77ec-11e8-8ee1-b482cdb696b2.png)
    * PlatformIO IDE: Busca `DSC` en el [Registro de Bibliotecas de PlatformIO](https://platformio.org/lib/show/5499/dscKeybusInterface)  
      ![PlatformIO](https://user-images.githubusercontent.com/12835671/41826138-d5852b62-77ec-11e8-805d-7c861a329e43.png)
    * PlatformIO CLI: `platformio lib install "dscKeybusInterface"`
    * Alternativamente, `git clone` o descarga el archivo .zip del repositorio en el directorio de bibliotecas de Arduino/PlatformIO para mantenerte al día con los últimos cambios.
2. Selecciona, configura y sube uno de los sketches de ejemplo al microcontrolador:  
      ![Examples](https://user-images.githubusercontent.com/12835671/102936214-61ed5580-446d-11eb-9d70-0eae40fe5757.png)
3. Conecta el microcontrolador al DSC Keybus según el diagrama de cableado con los resistores apropiados (y un transistor si deseas controlar el sistema).


## ¿Por qué?
**Tenía**: _Un sistema de seguridad DSC que no estaba siendo monitoreado por un servicio de terceros._  
**Quería**: _Notificación si la alarma se activaba._

Me interesaba encontrar una solución que accediera directamente al par de líneas de datos que DSC usa para su protocolo propietario Keybus para enviar datos entre el panel, los teclados y otros módulos (en lugar de usar el módulo serial DSC IT-100). Interceptar las líneas de datos es una tarea ideal para un microcontrolador y también presentaba una oportunidad para trabajar con las plataformas [Arduino](https://www.arduino.cc) y [FreeRTOS](https://www.freertos.org) (a través de [esp-open-rtos](https://github.com/SuperHouse/esp-open-rtos)).


Aunque ha habido una excelente [discusión sobre el protocolo DSC Keybus](https://www.avrfreaks.net/s/topic/a5C3l000000ULgIEAW/t097136) y varios proyectos existentes, había algunos problemas que seguían sin resolverse:
* Captura de datos Keybus propensa a errores.
* Decodificación de datos limitada: hubo buenos avances para los estados armado/desarmado y estado parcial de zonas para una sola partición, pero la mayoría de los datos no se decodificaban (notablemente faltaba el estado de alarma activada).
* Solo lectura: no era posible controlar el Keybus para actuar como un teclado virtual.
* No había implementaciones para hacer un trabajo útil con los datos.

Esta biblioteca utiliza una combinación de hardware e interrupciones de temporizador para capturar datos Keybus con precisión, y ha llevado a la ingeniería inversa de gran parte del protocolo Keybus.


## Características
* Monitoreo del estado de todas las particiones:
  - Alarma activada, armado/desarmado, retraso de entrada/salida, alarma de incendio, teclas de pánico del teclado
* Monitoreo del estado de las zonas:
  - Zonas abiertas/cerradas, zonas en alarma
* Monitoreo del estado del sistema:
  - Listo, problema, energía AC, batería
* Monitoreo del estado de las salidas PGM 1-14
* Teclado virtual:
  - Escribir teclas al panel para todas las particiones
  - Activar salidas de comando del panel
* Interfaz de teclado:
  - Emula un panel DSC para usar teclados DSC PowerSeries y Classic series como dispositivos de entrada física para cualquier propósito general, sin necesidad de un panel DSC.
* Hora del panel - recuperar la fecha/hora actual del panel y establecer una nueva fecha/hora (incluyendo un ejemplo con sincronización NTP)
* Desbloqueo del código de instalador del panel - determinar el código de instalador de 4 dígitos del panel
* Expansor de zonas virtual - la [rama expander](https://github.com/taligentx/dscKeybusInterface/tree/expander) puede emular un módulo expansor de zonas DSC para agregar zonas al sistema de seguridad que son manejadas por el microcontrolador - ¡gracias a [Dilbert66](https://github.com/Dilbert66) por esta contribución!
* Interfaz Keybus directa:
  - No requiere la [interfaz serial DSC IT-100](https://www.dsc.com/alarm-security-products/IT-100%20-%20PowerSeries%20Integration%20Module/22).
* Diseñado para una decodificación de datos y rendimiento confiables:
  - Interrupciones de pines GPIO de hardware e interrupciones de temporizador para una captura precisa de datos
  - Bufferización de datos: ayuda a prevenir la pérdida de datos Keybus si el sketch está ocupado
  - Decodificación extensa de datos: la mayoría de los datos Keybus, como se ve en la [guía del desarrollador del Data Interface DSC IT-100](https://cms.dsc.com/download.php?t=1&id=16238), ha sido ingeniería inversa y documentada en [`src/dscKeybusPrintData.cpp`](https://github.com/taligentx/dscKeybusInterface/blob/master/src/dscKeybusPrintData.cpp).
  - Código no bloqueante: Permite que los sketches se ejecuten lo más rápido posible sin usar `delay` o `delayMicroseconds`
* Sistemas de seguridad soportados:
  - [DSC PowerSeries](https://www.dsc.com/?n=enduser&o=identify) - se soportan todos los paneles, probados con: PC585, PC1555MX, PC1565, PC1565-2P, PC5005, PC5010, PC5015, PC5020, PC1616, PC1808, PC1832, PC1864
  - [DSC Classic series](https://www.dsc.com/?n=enduser&o=identify): PC1500, PC1550, PC2550
    * Requiere configurar el panel a través de la programación `*8` para habilitar el modo PC16-OUT - consulta [Configuración DSC](https://github.com/taligentx/dscKeybusInterface/tree/master?tab=readme-ov-file#dsc-configuration)
    * PC2500 y PC3000 requieren prueba, [publica un problema](https://github.com/taligentx/dscKeybusInterface/issues) si puedes probar estos paneles.
  - Los PowerSeries rebrandados de DSC (como algunos sistemas ADT) también deberían funcionar con esta interfaz.
* Sistemas de seguridad no soportados:
  - DSC Alexor (PC9155) es completamente inalámbrico y no tiene una interfaz Keybus accesible.
  - La serie DSC Neo utiliza un protocolo de datos cifrado de mayor velocidad (Corbus) que no es posible soportar actualmente.
  - Otras marcas (que no sean sistemas DSC rebrandados) utilizan diferentes protocolos y no son soportadas.
    * Para Honeywell Ademco Vista 15P/20P, consulta el proyecto [esphome-vistaECP de Dilbert66](https://github.com/Dilbert66/esphome-vistaECP)
    * Para sistemas Paradox, consulta el proyecto [paradox-esp8266 de liaan](https://github.com/liaan/paradox_esp8266)
* Microcontroladores soportados:
    - [Arduino](https://www.arduino.cc/en/Main/Products):
      * Placas: Uno, Mega, Leonardo, Mini, Micro, Nano, Pro, Pro Mini
      * Placas basadas en ATmega328P, ATmega2560 y ATmega32U4 a 16Mhz
    - esp8266:
      * Placas de desarrollo: NodeMCU v2 o v3, Wemos D1 Mini, etc.
      * Incluye [soporte del framework Arduino](https://github.com/esp8266/Arduino) y WiFi por ~$3 USD enviados.
    - esp32, esp32-s2, esp32-s3, esp32-c3:
      * Incluye [soporte del framework Arduino](https://github.com/espressif/arduino-esp32) (se requiere v2.0.2 o más reciente) por $3-5 USD enviados.
* Características posibles (¡PRs bienvenidos!):
  - Emulación del [DSC IT-100](https://cms.dsc.com/download.php?t=1&id=16238)
  - Desbloqueo de códigos de instalador de 6 dígitos

## Notas de la versión
* develop
  - Nuevo: Decodificación Keybus para programación de módulos, PC5200 y estado extendido de zonas 33-64 - ¡gracias a [kricon](https://github.com/kricon) por esta contribución!
  - Nuevo: Soporte para la serie Classic PC3000 (experimental) - con consideraciones del código PC3000 de [sfossen](https://github.com/sfossen). Las implementaciones difieren, también puedes probar su código original que ahora está preservado en la [rama pc3000](https://github.com/taligentx/dscKeybusInterface/tree/pc3000).
  - Nuevo: Funciones de biblioteca específicas de `KeybusReader` para capturar datos de módulos e imprimir datos Keybus, reduce el tamaño y el uso de memoria en todos los sketches
  - Actualizado: Soporte del sketch `Unlocker` para la serie Classic de DSC
  - Actualizado: Configuración del ejemplo `HomeAssistant-MQTT` para Home Assistant core 2022.06, soporte para cambiar modos armados mientras está armado, agregar botones de incendio/aux/pánico
  - Actualizado: `VirtualKeypad-Web` para soportar ArduinoJSON 7.x, usar LittleFS para esp8266
* 3.0
  - Nuevo: Soporte para paneles de la serie Classic de DSC: PC1500, PC1550, PC2550 - ¡gracias a [kricon](https://github.com/kricon) por probar extensamente y contribuir a esto!
  - Nuevo: Sketches de ejemplo `KeypadInterface` y `KeypadInterface-MQTT` - emula un panel DSC para conectar teclados DSC PowerSeries y Classic como dispositivos de entrada física para cualquier propósito general, sin necesidad de un panel DSC.
  - Nuevo: Sketch de ejemplo `HomeKit-HomeSpan` (esp32) - integrar directamente con Apple HomeKit como un accesorio independiente usando [HomeSpan](https://github.com/HomeSpan/HomeSpan)
  - Nuevo: Sketches de ejemplo de notificaciones push para [Pushover](https://www.pushover.net) y [Pushsafer](https://www.pushsafer.com) para esp8266/esp32
  - Nuevo: Soporte para el microcontrolador esp32-s2
  - Nuevo: Código reestructurado para soportar nuevas características desde sketches usando flags #define - habilita soporte para la serie Classic y `KeypadInterface`.
  - Actualizado: `Homebridge-MQTT` soporta cambiar modos armados mientras está armado
  - Actualizado: Se añadió el certificado raíz TLS a `Twilio-SMS`
  - Actualizado: Eliminada la función `handlePanel()` obsoleta
  - Corrección de errores: Notas actualizadas de `VirtualKeypad-Web` para cambiar a [este fork de ESPAsyncWebServer](https://github.com/arjenhiemstra/ESPAsyncWebServer) para resolver bloqueos con clientes iOS y macOS.
  - Corrección de errores: Sketch de ejemplo `Pushbullet` actualizado con el huella del certificado de seguridad TLS
  - Corrección de errores: Solución para [problema de handshake TLS en esp32](https://github.com/espressif/arduino-esp32/issues/6165) que impedía hacer una conexión TLS más de una vez.
  - Corrección de errores: Se arregló el manejo de `Homebridge-MQTT` para estados de retraso de salida mientras se están armando múltiples particiones
  - Corrección de errores: Se resolvió el problema de códigos de acceso que no se actualizaban al desarmar, cambiados los códigos de acceso para armar para actualizarse antes de que el estado armado cambie
* 2.0
  - Nuevo: Sketch de ejemplo para bot de [Telegram](https://www.telegram.org)
  - Nuevo: Sketch de ejemplo para integración con [OpenHAB](https://www.openhab.org) usando MQTT
  - Nuevo: Sketch de ejemplo `Unlocker` - determina el código de instalador del panel
  - Nuevo: Sketch de ejemplo `TimeSyncNTP` - usa NTP para establecer automáticamente la hora del panel
  - Nuevo: Ejemplo de integración con [ESPHome](https://esphome.io) (ubicado en el directorio `extras`) - ¡gracias a [Dilbert66](https://github.com/Dilbert66) por esta contribución!
  - Nuevo: Sketch de ejemplo `TinyGSM-SMS` - envía el estado por SMS con un módem GSM - ¡gracias a [jvitkauskas](https://github.com/jvitkauskas) por esta contribución!
  - Nuevo: Sketch de ejemplo `KeybusReaderIP` habilita el acceso a datos Keybus a través de IP, ¡gracias a [aboulfad](https://github.com/aboulfad) por esta contribución!
  - Nuevo: Soporte para el microcontrolador esp32 - requiere [Arduino-esp32](https://github.com/espressif/arduino-esp32) v1.0.5-rc6 o superior
  - Nuevo: Características para sketches:
      * `ready` y `disabled` rastrean el estado de las particiones
      * `setTime()` establece la fecha y hora del panel
      * `pgmOutputs[]` rastrea el estado de las salidas PGM 1-14
      * `timestampChanged` rastrea cuándo el panel envía una marca de tiempo
      * `accessCode` rastrea el código de acceso utilizado para armar/desarmar
      * `resetStatus()` desencadena una actualización completa del estado de todas las particiones y zonas - por ejemplo, después de la inicialización o una conexión de red perdida.
      * `pauseStatus` pausa las actualizaciones de estado si se establece en `true` - por ejemplo, durante una conexión de red perdida
      * `stop()` desactiva la interfaz - por ejemplo, antes de comenzar las actualizaciones OTA
      * `appendPartition()` en los sketches de ejemplo simplifica la adición de números de partición a los mensajes
      * `panelVersion` rastrea el número de versión del panel
  - Nuevo: Manejo de `*1 bypass/re-activate` usado para cambiar el modo stay/away mientras está armado
  - Actualizado: `VirtualKeypad-Blynk` y `VirtualKeypad-Web` muestran memoria de alarma, luces de zona de programación y buffer de eventos
  - Actualizado: `HomeAssistant-MQTT`, `Homebridge-MQTT`, `OpenHAB-MQTT` incluyen el estado de las salidas PGM 1-14
  - Actualizado: Escrituras en el teclado virtual
      * `write()` para múltiples teclas ahora se puede configurar para bloquear hasta que la escritura esté completa con un parámetro opcional si el array de caracteres es efímero
      * Comprobar `writeReady` generalmente ya no es necesario en el sketch, la biblioteca bloqueará si una escritura anterior está en progreso - esto se puede verificar si el sketch necesita esperar hasta que la biblioteca pueda realizar una escritura no bloqueante
  - Actualizado: Sketch `HomeAssistant-MQTT` ahora incluye armamento nocturno y para esp8266/esp32 incluye un sensor con mensajes de estado de partición
  - Actualizado: Procesamiento de estado de partición expandido para mejorar la detección del estado del panel al iniciar
  - Obsoleto: `handlePanel()` ahora es `loop()`
  - Corrección de errores: Se resolvió el estado de la tecla aux/pánico del teclado, el poder AC y el estado de la batería en PC585/PC1555MX
  - Corrección de errores: Se resolvió el problema de que el sketch `Homebridge-MQTT` no manejaba los estados objetivo de HomeKit
  - Corrección de errores: Se resolvieron problemas de sincronización al llamar consecutivamente a `write`
* 1.2
  - Nuevo: Ejemplo de interfaz web de teclado virtual, ¡gracias a [Elektrik1](https://github.com/Elektrik1) por esta contribución!
    - A partir de esp8266 Arduino Core 2.5.1, es posible que necesites [actualizar manualmente el plugin esp8266FS](https://github.com/esp8266/arduino-esp8266fs-plugin) para la carga de SPIFFS.
  - Nuevo: Soporte para CPU esp8266 funcionando a 160MHz - esto ayuda a los sketches que usan TLS a través de BearSSL
  - Actualizado: Ejemplo `HomeAssistant-MQTT` incluye estado de disponibilidad, ¡gracias a [bjrolfe](https://github.com/bjrolfe) por esta contribución!
  - Actualizado: Lista de paneles DSC probados: PC585, PC1565, PC5005, PC1808
  - Actualizado: Diagramas de cableado de energía para esp8266
  - Actualizado: Lista de módulos esp8266
* 1.1
  - Nuevo: Decodificación de manipulación y falla de zonas 33-64
  - Nuevo: Ejemplo de notificación push usando [Twilio](https://www.twilio.com), ¡gracias a [ColingNG](https://github.com/ColinNg) por esta contribución!
  - Corrección de errores: Estado incorrecto de las zonas 17-32 almacenado
* 1.0
  - Nuevo: Ejemplo de teclado virtual [Blynk](https://www.blynk.io) y ejemplos de diseño de aplicación
  - Nuevo: Soporte para teclado virtual en salidas de comando de terminales PGM 1-4
  - Nuevo: Estado `keybusConnected` para verificar si se están recibiendo datos del panel DSC
  - Nuevo: Decodificación de alarmas de entrada auxiliar
* 0.4
  - Nuevo: Soporte para teclado virtual en particiones 3-8, gracias a [jvitkauskas](https://github.com/jvitkauskas) por contribuir con los registros necesarios
  - Nuevo: Soporte para placas Arduino basadas en ATmega32U4 (cambio a Timer1 de AVR)
  - Cambiado: Nombres de ejemplos simplificados, configuraciones, números de versión añadidos
  - Corrección de errores: Escrituras en teclado virtual con particiones 5-8 habilitadas
  - Corrección de errores: Escrituras de teclas F/A/P con `processModuleData` desactivado
  - Corrección de errores: Error en `configuration.yaml` del ejemplo de HomeAssistant para `alarm_control_panel`
* 0.3
  - Nuevo: Estado para particiones 2-8, zonas 33-64
  - Nuevo: Soporte para teclado virtual en partición 2
  - Nuevo: Ejemplo de integración con [Athom Homey](https://www.athom.com/en/), gracias a [MagnusPer](https://github.com/MagnusPer) por esta contribución
  - Nuevo: Disposición de PCB, contribuida por [sjlouw](https://github.com/sj-louw)
  - Nuevo: Número configurable de particiones y zonas para personalizar el uso de memoria: `dscPartitions` y `dscZones` en `dscKeybusInterface.h`
  - Nuevo: Decodificación de comandos `0xE6` y `0xEB` en KeybusReader
  - Cambiado: Ejemplos divididos por plataforma
  - Cambiado: Los sketches de Arduino ya no usan el pin 4 para evitar conflictos con la tarjeta SD en shields Ethernet.
  - Cambiado: Ejemplos MQTT actualizados con campos de nombre de usuario y contraseña
  - Cambiado: `processRedundantData` ahora es verdadero por defecto para evitar almacenar datos repetitivos, reduce el uso de memoria.
  - Nota: Esta versión cambia los métodos de la biblioteca para acomodar múltiples particiones, los sketches existentes necesitarán ser actualizados para coincidir con los nuevos ejemplos.
* 0.2
  - Nuevo: Estado para zonas 9-32
  - Nuevo: Ejemplo de integración con [Home Assistant](https://www.home-assistant.io)
  - Nuevo: Bufferización de datos del panel, añade `dscBufferSize` a `dscKeybusInterface.h` para permitir la configuración del número de comandos del panel que se almacenan en buffer para personalizar el uso de memoria (usa 18 bytes de memoria por comando almacenado en buffer).
* 0.1 - Lanzamiento inicial
## Examples

Los ejemplos incluidos demuestran cómo utilizar la biblioteca y se pueden usar tal cual o adaptar para integrarse con otros softwares. Publica un problema/pull request si has desarrollado (y te gustaría compartir) un sketch/integración que otros puedan usar.

* **Status**: Procesa e imprime el estado del sistema de seguridad en una interfaz serial, incluyendo la lectura de datos del teclado virtual. Esto demuestra cómo determinar si el estado del sistema ha cambiado, qué ha cambiado y cómo tomar acción en base a esos cambios. Actualmente, solo se rastrea un subconjunto de todos los comandos decodificados para limitar el uso de memoria:
  * Particiones listas
  * Particiones armadas en modo away/stay/desarmado
  * Particiones en alarma
  * Retraso de salida en particiones en progreso
  * Retraso de entrada en particiones en progreso
  * Alarma de incendio en particiones
  * Zonas abiertas/cerradas
  * Zonas en alarma
  * Salidas PGM 1-14
  * Alarma de incendio/auxiliar/pánico en el teclado
  * Obtener/establecer la fecha y hora del panel
  * Número de código de acceso de usuario (1-40)
  * Energía AC del panel
  * Batería del panel
  * Problema del panel
  * Keybus conectado

* **HomeKit-HomeSpan** (esp32): Se integra directamente con Apple HomeKit como un accesorio nativo (para la app Home y Siri) usando [HomeSpan](https://github.com/HomeSpan/HomeSpan), sin necesidad de un servicio o dispositivo adicional. Demuestra el armado/desarmado de particiones, estado de zonas, alarmas de incendio, estado de salidas PGM y control de salidas de comandos del panel.
  - Para esp8266, la biblioteca [dscKeybusInterface-RTOS](https://github.com/taligentx/dscKeybusInterface-RTOS) incluye una implementación nativa de HomeKit que se ejecuta directamente en esp8266, sin requerir un dispositivo adicional que ejecute MQTT o Homebridge.

* **Homebridge-MQTT**: Interfaz con [Homebridge](https://github.com/nfarina/homebridge) a través de MQTT para integrarse con Apple HomeKit (incluyendo la app iOS Home y Siri) y [Google Home](https://github.com/oznu/homebridge-gsh). Demuestra el armado/desarmado de particiones y, para HomeKit, la visualización del estado de zonas, salidas PGM y alarmas de incendio.

* **HomeAssistant-MQTT**: Interfaz con [Home Assistant](https://www.home-assistant.io) a través de MQTT. Demuestra el armado/desarmado de particiones y la visualización del estado de zonas, salidas PGM, alarmas de incendio y problemas. Para esp8266/esp32, el estado de particiones está disponible como un mensaje de texto para su visualización.

* **OpenHAB-MQTT**: Interfaz con [OpenHAB](https://www.openhab.org) a través de MQTT. Demuestra el uso de los estados del panel y particiones como interruptores de OpenHAB y los estados de zonas como contactos de OpenHAB. Para esp8266/esp32, también se envía un mensaje de estado del panel como una cadena a OpenHAB. Consulta https://github.com/jimtng/dscalarm-mqtt para una integración usando la convención Homie para el componente Homie MQTT de OpenHAB.

* **ESPHome** (esp8266): Se integra con [ESPHome](https://esphome.io) como un componente personalizado - ten en cuenta que este ejemplo está ubicado en el directorio `extras`. ¡Gracias a [Dilbert66](https://github.com/Dilbert66) por esta contribución!

* **Homey**: Se integra con [Athom Homey](https://www.athom.com/en/) y la biblioteca [Homeyduino](https://github.com/athombv/homey-arduino-library/), incluyendo estados armados, alarmas y fuego (actualmente limitado a una partición), y estados de zonas. ¡Gracias a [MagnusPer](https://github.com/MagnusPer) por contribuir con este ejemplo!

* **Telegram** (esp8266/esp32): Demuestra el envío de actualizaciones de estado como notificaciones push y el armado/desarmado del sistema de seguridad a través de un [Telegram](https://www.telegram.org) bot. Soporta notificaciones en iOS, Android y escritorios macOS/Windows/Linux (gratuito).

* **Pushover** (esp8266/esp32): Demuestra el envío de actualizaciones de estado como notificaciones push a través de [Pushover](https://www.pushover.net). Soporta notificaciones en iOS, Android, escritorios macOS nativos y ventanas emergentes en navegadores Chrome/Firefox/Safari (4.99USD una sola vez por cliente).

* **Pushbullet** (esp8266/esp32): Demuestra el envío de actualizaciones de estado como notificaciones push a través de [Pushbullet](https://www.pushbullet.com). Soporta notificaciones en Android, Windows y ventanas emergentes en navegadores Chrome/Firefox (gratuito). Ten en cuenta que iOS ya no es compatible.

* **Pushsafer** (esp8266/esp32): Demuestra el envío de actualizaciones de estado como notificaciones push a través de [Pushsafer](https://www.pushsafer.com). Soporta notificaciones en iOS, Android, escritorios Windows y ventanas emergentes en navegadores Chrome/Firefox/Edge/Opera/Yandex (0.99EUR o menos por 1000 notificaciones).

* **Twilio-SMS** (esp8266/esp32): Demuestra el envío de actualizaciones de estado como mensajes de texto SMS a través de [Twilio](https://www.twilio.com) - ¡gracias a [ColingNG](https://github.com/ColinNg) por contribuir con este ejemplo!

* **Email** (esp8266/esp32): Demuestra el envío de actualizaciones de estado por correo electrónico. El correo se envía usando SMTPS (puerto 465) con SSL para cifrado - esto es necesario en esp8266/esp32 hasta que STARTTLS pueda ser compatible. Por ejemplo, esto funcionará con Gmail después de cambiar la configuración de la cuenta para [permitir aplicaciones menos seguras](https://support.google.com/accounts/answer/6010255).

  Esto se puede usar para enviar mensajes de texto SMS si el proveedor de servicios del número tiene una [puerta de enlace de correo a SMS](https://en.wikipedia.org/wiki/SMS_gateway#Email_clients) - ejemplos para EE. UU.:
  * T-mobile: 5558675309@tmomail.net
  * Verizon: 5558675309@vtext.com
  * AT&T: 5558675309@txt.att.net

* **VirtualKeypad-Web** (esp8266/esp32): Proporciona una interfaz web de teclado virtual, usando el esp8266/esp32 como un servidor web independiente, incluyendo la visualización de memoria de alarmas, luces de zonas de programación y el buffer de eventos. ¡Gracias a [Elektrik1](https://github.com/Elektrik1) por contribuir con este ejemplo!

* **VirtualKeypad-Blynk** (esp8266/esp32): Proporciona una interfaz de teclado virtual para la aplicación [Blynk legacy](https://blynk.io/blog/what-will-happen-to-the-legacy-blynk-platform) gratuita en iOS y Android, incluyendo la visualización de memoria de alarmas, luces de zonas de programación y el buffer de eventos. Esto usa un [servidor Blynk local](https://github.com/Peterkn2001/blynk-server) y no requiere servicios o suscripciones en la nube. Escanea uno de los siguientes códigos QR desde la aplicación Blynk legacy para ver un diseño de teclado de ejemplo:
  - [Teclado virtual con 16 zonas](https://user-images.githubusercontent.com/12835671/103719316-5f6f1d80-4f8e-11eb-8a7c-4bd7bfe3cd8a.png)
  - [Teclado virtual con 32 zonas](https://user-images.githubusercontent.com/12835671/103719459-af4de480-4f8e-11eb-8e4a-7172961e2d29.png)
  - [Teclado virtual con 8 zonas y registro de eventos](https://user-images.githubusercontent.com/12835671/103719518-cc82b300-4f8e-11eb-8b2a-97299e7be3a2.png)

  La actual aplicación Blynk IoT no es compatible - esta nueva plataforma es solo en la nube y tiene planes de precios basados en el número de elementos de datos en la interfaz. El teclado virtual usa hasta 124 elementos y requeriría el [plan de precios Pro](https://blynk.io/pricing) a un costo elevado de $99USD/mes.

* **TimeSyncNTP**: Sincroniza y mantiene el tiempo en los paneles PowerSeries a través de un servidor NTP, incluyendo ajustes de DST.

* **Unlocker**: Encuentra el código de instalador de 4 dígitos para los paneles PowerSeries y Classic probando todos los códigos posibles, incluyendo el manejo del bloqueo del teclado si está habilitado. El código válido se muestra en el serial y también se parpadea repetidamente con el LED incorporado. Arduino revisa cada código secuencialmente, pero esp8266/esp32 puede encontrar el código más rápidamente ya que revisa en el orden de los [códigos de 4 dígitos más comúnmente usados](http://www.datagenetics.com/blog/september32012).

* **KeypadInterface**: Interfaz directa con teclados DSC PowerSeries y Classic (probado con PC1500RK) (sin un panel DSC) para usar estos como entradas físicas para cualquier propósito general. Ejemplos incluidos para la interfaz a través de serial y MQTT. Nota: Esto usa una configuración de cableado diferente de la interfaz Keybus estándar, consulta el diagrama de cableado en el sketch de ejemplo.

* **KeybusReader**: Decodifica e imprime datos del Keybus en una interfaz serial, incluyendo la lectura del teclado virtual desde el serial. Esto puede ayudar a decodificar el protocolo Keybus y también es útil como herramienta de resolución de problemas para verificar que los datos se muestren sin errores. Para esp8266/esp32, `KeybusReaderIP` habilita la conectividad a través de WiFi.

  Consulta [`src/dscKeybusPrintData.cpp`](https://github.com/taligentx/dscKeybusInterface/blob/master/src/dscKeybusPrintData.cpp) para todos los comandos y mensajes del protocolo Keybus actualmente conocidos. ¡Se aceptan problemas y pull requests con adiciones/correcciones!

## Más proyectos DSC
* **[dscalarm-mqtt](https://github.com/jimtng/dscalarm-mqtt)**: implementación de la convención MQTT [Homie](https://homieiot.github.io)
* **[esphome-dsckeybus](https://github.com/Dilbert66/esphome-dsckeybus)**: implementación de esta biblioteca como un componente personalizado de ESPHome
* **[PC1500KeybusReader](https://github.com/polishedmarvin/dscKeybusInterface)**: Ejemplo de MQTT HomeKit para el PC1500 y ESP32 usando [dougkpowers/pc1550-interface](https://github.com/dougkpowers/pc1550-interface)

## Wiring

```
DSC Aux(+) ---+--- Arduino Vin pin
              |
              +--- 5v voltage regulator --- esp8266 NodeMCU / Wemos D1 Mini 5v pin
                                            esp32 development board 5v pin

DSC Aux(-) --- Arduino/esp8266/esp32 Ground

                        Arduino        +--- dscClockPin (Arduino Uno: 3)
DSC Yellow ---+--- 15k ohm resistor ---|
              |                        +--- 10k ohm resistor --- Ground
              |
              |     esp8266/esp32      +--- dscClockPin (esp8266: D1, GPIO 5 / esp32: 18)
              +--- 33k ohm resistor ---|
                                       +--- 10k ohm resistor --- Ground

                        Arduino        +--- dscReadPin (Arduino Uno: 5)
DSC Green ----+--- 15k ohm resistor ---|
              |                        +--- 10k ohm resistor --- Ground
              |
              |     esp8266/esp32      +--- dscReadPin (esp8266: D2, GPIO 4 / esp32: 19)
              +--- 33k ohm resistor ---|
                                       +--- 10k ohm resistor --- Ground

Classic series only, PGM configured for PC16-OUT mode:
DSC PGM ------+--- 1k ohm resistor --- DSC Aux(+)
              |
              |         Arduino        +--- dscPC16Pin (Arduino Uno: 4)
              +--- 15k ohm resistor ---|
              |                        +--- 10k ohm resistor --- Ground
              |
              |     esp8266/esp32      +--- dscPC16Pin (esp8266: D7, GPIO 13 / esp32: 17)
              +--- 33k ohm resistor ---|
                                       +--- 10k ohm resistor --- Ground

Virtual keypad (optional):
DSC Green ---- NPN collector --\
                                |-- NPN base --- 1k ohm resistor --- dscWritePin (Arduino Uno: 6 / esp8266: D8, GPIO 15 / esp32: 21)
      Ground --- NPN emitter --/
```

* The DSC PowerSeries Keybus operates at ~12.6v and the Classic series operates at ~13.75v, a pair of resistors per data line will bring this down to an appropriate voltage for each microcontroller.
    * Arduino:
      * The DSC yellow (clock) line connects to a [hardware interrupt pin](https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/) - for the Uno, these are pins 2 or 3.  The example sketches use dscClockPin: 3.
      * The DSC green (data) line can be connected to any of the remaining digital pins 2-12.  The examples sketches use dscReadPin: 5 and dscWritePin: 6.
    * esp8266: connect the DSC lines to GPIO pins that are normally low to avoid putting spurious data on the Keybus: D1 (GPIO5), D2 (GPIO4) and D8 (GPIO15).  The example sketches use dscClockPin: D1, dscReadPin: D2, dscWritePin: D8.  For the Classic series, dscPC16Pin: D7 (GPIO 13).
    * esp32: connect the DSC lines to GPIO pins that do not send signals at boot: 4, 13, 16-39.  For virtual keypad, use pins 4, 13, 16-33 - pins 34-39 are input only and cannot be used.  The example sketches use dscClockPin: 18, dscReadPin: 19, dscWritePin: 21.  For the Classic series, dscPC16Pin: 17.
* Virtual keypad uses an NPN transistor and a resistor to write to the Keybus.  Most small signal NPN transistors should be suitable, for example:
  * 2N3904
  * BC547, BC548, BC549
  * That random NPN at the bottom of your parts bin (my choice)
* Power:
  * Arduino boards can be powered directly from the DSC panel
  * esp8266/esp32 development boards should use an external voltage regulator set to 5v to the 5v pin:
    - LM2596-based step-down buck converter modules are reasonably efficient and commonly available for under $1USD shipped (eBay, Aliexpress, etc) - these are the modules I use.
    - MP2307-based step-down buck converter modules (aka Mini360) are also available but some versions run hot with an efficiency nearly as poor as linear regulators.
    - Linear voltage regulators (LM7805, etc) will work but are inefficient and run hot - these may need a heatsink.
  * esp8266/esp32 boards can also use an external voltage regulator set to 3.3v to the 3.3v pin - this bypasses the module's onboard voltage regulator.  For example, some Wemos D1 mini clones use [low current voltage regulators](https://www.letscontrolit.com/forum/viewtopic.php?t=6603) that can cause stability issues.  NodeMCU boards are not affected as they use the more powerful AMS1117 regulator.
* Connections should be soldered, breadboards can cause issues.

## Virtual keypad
This allows a sketch to send keys to the DSC panel to emulate the physical DSC keypads and enables full control of the panel from the sketch or other software.

Keys are sent to partition 1 by default and can be changed to a different partition.  The following keys can be sent to the panel - see the examples for usage:

* Keypad: `0-9 * #`
* Arm stay (requires access code if quick arm is disabled): `s`
* Arm away (requires access code if quick arm is disabled): `w`
* Arm with no entry delay (requires access code): `n`
* Fire alarm: `f`
* Auxiliary alarm: `a`
* Panic alarm: `p`
* Door chime enable/disable: `c`
* Fire reset: `r`
* Quick exit: `x`
* Change partition: `/` + `partition number` or set `writePartition` to the partition number.  Examples:
  * Switch to partition 2 and send keys: `/2` + `1234`
  * Switch back to partition 1: `/1`
  * Set directly in sketch: `dsc.writePartition = 8;`
* Command output 1: `[`
* Command output 2: `]`
* Command output 3: `{`
* Command output 4: `}`

## DSC Configuration
Panel options affecting this interface, configured by `*8 + installer code` - see the `Unlocker` sketch if your panel's installer code is unknown.  Refer to the DSC installation manual for your panel to configure these options:
* Swinger shutdown: By default, the panel will limit the number of alarm commands sent in a single armed cycle to 3 - for example, a zone alarm being triggered multiple times will stop reporting after 3 alerts.  This is to avoid sending alerts repeatedly to a third-party monitoring service, and also affects this interface.  As I do not use a monitoring service, I disable swinger shutdown by setting this to `000`.
  - PC1555MX/5015: section `370`
  - PC1616/PC1832/PC1864 section `377`
  - PC1500/PC1550: section `15`
  - PC2550: section `22`

* AC power failure reporting delay: The default delay is 30 minutes and can be set to `000` to immediately report a power failure.
  - PC1555MX/5015: section `370`
  - PC1616/PC1832/PC1864 section `377`
  - PC1500/PC1550: section `17`
  - PC2500: section `16`
  - PC2550: section `20`
  - PC3000: section `22`

* Classic series - requires PC16-OUT mode to get the security system status (see wiring section):
  - Communicator: Enable to support PC16-OUT mode (if not using a monitoring service, "Communicator Call Directions" options can be set to `0`)
    - PC1500/PC1550: section `12`, option `1` off
    - PC2500: section `13`, option `1` off
    - PC2550: section `15`, option `1` off
    - PC3000: section `18`, option `1` off
  - PC16-OUT: Enable to set the PGM output to PC16-OUT mode to send required panel status data on the Keybus.
    - PC1500/PC1550: section `13`, option `4` on
    - PC2500: section `14`, option `4` on
    - PC2550: section `16`, option `4` on
    - PC3000: section `19`, option `4` on
  - PGM output: Set to "Strobe output" mode to track when the alarm is tripped.
    - PC1500/PC1550: section `24`, option `08`
    - PC2500: section `22`, option `x8` (first digit is aux input zone configuration, default `2`)
    - PC2550: section `30`, option `x8`
    - PC3000: section `28`, option `x8`

## Notes
* For OTA updates on esp8266 and esp32, you may need to stop the interface using `dsc.stop();`:
  ```
  void setup() {
    ...
    ArduinoOTA.onStart([]() {
      dsc.stop();
    ...
  ```
* Memory usage can be adjusted based on the number of partitions, zones, and data buffer size specified in [`src/dscKeybus.h`](https://github.com/taligentx/dscKeybusInterface/blob/master/src/dscKeybus.h) or [`src/dscClassic.h`](https://github.com/taligentx/dscKeybusInterface/blob/master/src/dscClassic.h).   Default settings:
  * Arduino: up to 4 partitions, 32 zones, 10 buffered commands
  * esp8266/esp32: up to 8 partitions, 64 zones, 50 buffered commands

* PCB layouts are available in [`extras/PCB Layouts`](https://github.com/taligentx/dscKeybusInterface/tree/master/extras/PCB%20Layouts) - thanks to [sjlouw](https://github.com/sj-louw) for contributing these designs!

* Support for other platforms depends on adjusting the code to use their platform-specific timers.  In addition to hardware pin-change interrupts to capture the DSC clock, this library uses platform-specific timer interrupts to capture the DSC data line in a non-blocking way 250μs after the clock changes (without using `delayMicroseconds()`).  This is necessary because the clock and data are asynchronous - I've observed keypad data delayed up to 160μs after the clock falls.

* Resource utilization:
  * Arduino: 1 hardware interrupt digital pin, 2 digital pins (+1 for Classic series), Timer1 interrupt
  * esp8266: 3 digital pins (+1 for Classic series), timer1 interrupt
  * esp32 series: 3 digital pins (+1 for Classic series), timer0 interrupt

## Troubleshooting
If you are running into issues:
1. Run the `KeybusReader` example sketch and view the serial output to verify that the interface is capturing data successfully without reporting checksum errors.
    * If data is not showing up or has errors, check the clock and data line wiring, resistors, and all connections.  Breadboards can cause issues, connections should be soldered instead.
    * Power supply issues can also cause data errors or microcontroller crashes/reboots - if this occurs, test with an external power supply set to 5v (Arduino) or 3.3v (esp8266/esp32) and connected to the matching voltage pin (bypassing the voltage regulator on development boards).
2. For virtual keypad, run the `KeybusReader` example sketch and enter keys through serial and verify that the keys appear in the output and that the panel responds.
    * If keys are not displayed in the output, verify the transistor pinout, base resistor, and wiring connections.
3. Run the `Status` example sketch and view the serial output to verify that the interface displays events from the security system correctly as partitions are armed, zones opened, etc.

For general discussions, feature requests, or how-to issues, you can [post in Discussions](https://github.com/taligentx/dscKeybusInterface/discussions), or [post an Issue](https://github.com/taligentx/dscKeybusInterface/issues) if it looks like an issue with the library code itself.

## References
[AVR Freaks - DSC Keybus Protocol](https://www.avrfreaks.net/s/topic/a5C3l000000ULgIEAW/t097136): An excellent discussion on how data is sent on the Keybus.

[stagf15/DSC_Panel](https://github.com/stagf15/DSC_Panel): A library that nearly works for the PC1555MX but had timing and data errors.  Writing this library from scratch was primarily a programming exercise, otherwise it should be possible to patch the DSC_Panel library.

[dougkpowers/pc1550-interface](https://github.com/dougkpowers/pc1550-interface): An interface for the DSC Classic series.
