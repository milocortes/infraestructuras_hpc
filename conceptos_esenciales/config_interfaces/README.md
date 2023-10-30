# Configuración de interfaces de red

Se realizará la Configuración de la red mediante el administrador *NetworkManager*. El administrador permite consultar y controlar la configuración y el estado de la red. NetworkManager integra tres herramientas:

* Interfaz de usuario de texto, ```nmtui```.
* Utilería de línea de comandos, ```nmcli```.
* Las herramientas de la interfaz gráfica de usuario, GNOME GUI.

También es posible  configurar una interfaz de red usando archivos de configuración, esto es, editando manualmente los archivos de ```ifcfg``` que se encuentran en el directorio ```/etc/sysconfig/network-scripts```. Si se edita un archivo ```ifcfg```, *NetworkManager* no es automáticamente consciente del cambio y tiene que ser solicitado para notar el cambio:

1. Para cargar un nuevo archivo de configuración:
```console
#> nmcli connection load /etc/sysconfig/network-scripts/ifcfg-connection_name
```

2. Si ha actualizado un archivo de conexión que ya ha sido cargado en *NetworkManager*:
```console
#> nmcli connection up connection_name
```

Los comandos para reiniciar la red después de modificar el archivo:

```console
#> nmcli networking off; nmcli networking on
```

Si es necesario reiniciar NetworkManager:
```console
#> systemctl restart NetworkManager.service
```

## Configuración de interfaces de red con ```nmcli```

### Comandos frecuentes
Para mostrar la lista de perfiles de conexión:

```console
#> nmcli connection show
```

Para mostrar la configuración de un perfil de conexión específico:
```console
#> nmcli connection show connection_name
```

Para modificar las propiedades de una conexión:
```console
#> nmcli connection modify connection_name property value
```

Para activar una conexión:
```console
#> nmcli connection up connection_name
```
Para desactivar una conexión:
```console
#> nmcli connection down connection_name
```

### Configuración de una red Ethernet

Se describe el procedimiento para agregar una conexión estática Ethernet con las siguiente configuración:

* Dirección IPv4 estática 192.168.0.1 con una máscara de subred /24.
* Gateway por defecto IPv4 192.168.0.254.
* Un servidor DNS IPv4 192.168.0.200.
* Un dominio de búsqueda DNS example.com


1. Añade un nuevo perfil de conexión *NetworkManager* para la conexión Ethernet:
```console
#> nmcli connection add con-name enp7s0 ifname enp7s0 type ethernet
```
2. Establezca la dirección IPv4:
```console
#> nmcli connection modify enp7s0 ipv4.addresses 192.168.0.1/24
```

3. Establezca el método de conexión IPv4 en manual:
```console
#> nmcli connection modify enp7s0 ipv4.method manual
```

4. Establezca el gateway por defecto IPv4:
```console
#> nmcli connection modify enp7s0 ipv4.gateway 192.168.0.254
```

5. Establezca las direcciones de los servidores DNS IPv4:
```console
#> nmcli connection modify enp7s0 ipv4.dns "192.168.0.200"
```

6. Para establecer varios servidores DNS, especifíquelos separados por espacios y
encerrados entre comillas.


7. Establezca el dominio de búsqueda DNS para la conexión:
```console
#> nmcli connection modify enp7s0 ipv4.dns-search example.com
```

8. Activar el perfil de conexión:
```console
#> nmcli connection up enp7s0
```

#### Verificación de la configuración

1. Muestra el estado de los dispositivos y las conexiones:
```console
#> nmcli device status
```

2. Para mostrar todos los ajustes del perfil de conexión:
```console
#> nmcli connection show enp7s0
```

### Configuración de una conexión Ethernet dinámica mediante ```nmcli```

Se describe la adición de una conexión Ethernet dinámica utilizando la utilidad ```nmcli```. Con esta configuración, *NetworkManager* solicita la configuración IP para esta conexión a un servidor DHCP.

Requisitos previos: Debe haber un servidor DHCP disponible en la red.

* Añade un nuevo perfil de conexión NetworkManager para la conexión Ethernet:
```console
#> nmcli connection add con-name enp7s0 ifname enp7s0 type ethernet
```
