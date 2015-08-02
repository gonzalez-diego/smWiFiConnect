# SmartWiFiConnect ESP-01 *(LAN, WAN & UART)*

[TOC]

## Conexión a WiFi
### Protocolo de configuración inalámbrica
1. Comprobación periódica *(1 seg)* del estado de la *auto-conexión* *("init.lua")*.
2. En caso de timeout *(30 seg)* o fallo en la conexión *(error, pwd erróneo, wifi no encontrado...)* arrancamos el **modo StationAP** .
	1. Arrancamos el **modo StationAP** *("smConfConn.lua")*.
	2. Configuramos los ajustes del **modo StationAP** *("smAPConfData.lua")*.
	3. Escaneamos las redes disponibles y las almacenamos.
	4. En la IP *192.168.4.1* devolvemos una página web con las redes disponibles.
	5. Conectamos con la red indicada en la página web.
3. Lanzamos la *smartApp* correspondiente *(Ej:"smDimmer.lua")*.

## Aplicación Smartidea
### Conexión a Internet
#### Protocolo ESP-01 -> socketServer

1. Apertura de conexión contra el puerto 2525.
2. Procesado del saludo de servidor.
3. Envío de la ID de *smartdevice* basada en la dirección MAC.
4. Procesado de la respuesta de autenticación del servidor.
5. Procesado de los distintos datos recibidos a través del socket.

### Conexión a Cliente LAN
#### Protocolo ESP-01 <- Cliente LAN

1. Escucha en el puerto 2525.
2. Recepción del *payload (stringBuffer)* en formato `JSON` simple:
	- El formato del *payload* **`{deviceID:SMARTIDEA-XX-YY,smCommand:COMMAND,smData:DATA}`**, sin importancia del orden de los parámetros.
	- El argumento `deviceID` será en formato **`SMARTIDEA-XX-YY`**, donde *XX* e *YY* son los cuatro últimos dígitos de la MAC.
3. Envío de la respuesta por parte del smart device.
	- El smart device responderá devolviendo los datos solicitados a través del socket en un *string* en formato `JSON`.
	- En caso de error, devolverá un mensaje informativo en el campo `smData` y el comando solicitado en `smCommand`. Si el error se produce en la comprobación del `DEVICE-ID`, pero si el parámetro pasado como `DEVICE-ID` es exactamente `getdeviceid` el smart device devolverá el `DEVICE-ID` *correcto*.

##### Comandos disponibles para el *payload (stringBuffer)*
La *API REST* del smart device soporta los siguientes comandos *(smCommand)*:
- `send` - Envía *temporalmente* **el primer comando (byte)** recibido en `smData`.
- `print` - Escribe los datos recibidos en `DATA` por el puerto serie y a su vez los devuelve por el socket.
- `restart` - Reinicia el smart device y devuelve confirmación.
- `getip` - Devuelve por el socket la dirección IP del smart device.
- `getmac` - Devuelve por el socket la dirección MAC del smart device.
- `getwlancfg` - Devuelve por el socket la configuración de la WLAN actual del smart device *(SSID, PASSWORD, BSSID_SET y BSSID)*.

### Conexión UART a microcontrolador PIC
Al lanzar la App Smartidea pertinente *(EJ: smDimmer)* se configurará la *UART* para conectar con el *PIC* con los siguientes parámetros:

- `id = 0` - id de la *UART* *(NodeMCU solo soporta una aunque hay que indicarlo igualmente)*.
- `baud = 115200`
- `databits = 8`
- `parity = none`
- `stopbits = 1`
- `echo = 0`

Actualmente se registra un evento *(callback)* en la *UART* para recibir un byte y en función de lo *recibido*\* lo envía por el socket en un objeto *JSON*  según lo siguiente:

- `smDevice : SMARTIDEA-XX-YY`
- `smCommand : picdata`
- `smData : data` - *Datos recibidos del PIC*.

[Smartidea ®](http://smartidea.es)
