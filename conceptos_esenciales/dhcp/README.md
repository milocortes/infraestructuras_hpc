# Servicio DHCP en Linux

El servicio **Dynamic Host Control Protocol (DHCP)** es utilizado para proporcionar información básica que los hosts necesitan para conectarse a la red y, en algunos casos, en donde encontrar configuración adicional, lo cual lo convierte en una pieza clave de la mayoría de las infraestructuras.

DHCP permite a los administradores de sistema definir la configuración de dispositivos en un servidor de forma centralizada, de manera que cuando estos dispositivos se encienden, pueden solicitar dichos parámetros de configuración. Esta *configuración central* casi siempre incluye parámetros básicos de red como direcciones IP, máscaras de subred, gateway por default, servidor DNS, etc. Lo que significa que en la mayoría de las organizaciones, los dispositivos no tienen una dirección IP estática u otras definiciones de red, sino que dichas configuraciones son definidas por el servidor DHCP.

## Instalación y configuración de un servidor DHCP

Prerequisitos: Agregar una nueva interfaz de red interna.

Renombramos el hostname al editar los archivos ```/etc/hostname``` y ```/etc/hosts``` por el hostname ```dhcp-server```.

1. Instalar el servidor DHCP
```console
[root@dhcp-server ~]# dnf install dhcp-server
```

2. Configurar la interfaz de red interna con los siguientes parámetros:
  * IP: 192.168.1.251
  * DNS: 8.8.8.8

  ```console
    [root@dhcp-server ~]# nmcli connection add con-name enp0s8 ifname enp0s8 type ethernet
    [root@dhcp-server ~]# nmcli connection modify enp0s8 ipv4.addresses 192.168.1.251/24
    [root@dhcp-server ~]# nmcli connection modify enp0s8 ipv4.method manual
    [root@dhcp-server ~]# nmcli connection modify enp0s8 ipv4.dns 8.8.8.8
    [root@dhcp-server ~]# nmcli connection up enp0s8
  ```

3. Copiar el archivo dhcpd.service al directorio /etc/systemd/system/
```console
root@dhcp-server ~]# cp /usr/lib/systemd/system/dhcpd.service /etc/systemd/system/
```

4. Editar el archivo ```/etc/systemd/system/dhcpd.service``` y colocar el dispositivo de red, en el parámetro ```ExecStart```, en el que el servidor DHCP escuchará:

5. Editar el archivo de configuración del servidor DHCP (```/etc/dhcp/dhcpd.conf```):

6. Arrancar el servidor DHCP y habilitarlo:
```console
[root@dhcp-server ~]# systemctl start dhcpd.service
[root@dhcp-server ~]# systemctl enable dhcpd.service
```

7. Verificar el estado del servidor DHCP:
```console
[root@dhcp-server ~]# systemctl status dhcpd.service
```

8. Permitir el puerto del servidor DHCP en el firewall:
```console
[root@dhcp-server ~]# firewall-cmd --add-service=dhcp --permanent
[root@dhcp-server ~]# firewall-cmd --reload
```

9. Verificar que el puerto 67 esté abierto:
```console
[root@dhcp-server ~]# nmap -sU 192.168.1.251
```

10. Configurar una interfaz de red interna de un host cliente para que obtenga la configuración de forma automática.

11. Verificar en el cliente la configuración que le mandó el servidor DHCP:
```console
[root@centos02 ~]# ip addr
```

12. Observar en el servidor DHCP los clientes que han requerido configuración de su interface de red:
```console
[root@dhcp-server ~]# cat /var/lib/dhcpd/dhcpd.leases
```

>#### Significado de los parámetros
* starts: Hora de inicio del arrendamiento.
* ends: Finalización de arrendamiento.
* tstp: Se especifica si se utiliza el protocolo de conmutación por error e indica el tiempo de vencimiento del arrendamiento que el par ha reconocido.
* cltt: Es la hora de la última transacción del cliente.
* binding state: Cuando el servidor DHCP no está configurado para utilizar el protocolo  e conmutación por error, el estado vinculante de una concesión será active o free.
next binding state: Indica a qué estado pasará el contrato de arrendamiento cuando expire el estado actual.
* rewind binding: Es parte de una optimización para su uso con conmutación por error. Esto ayuda a un servidor a rebobinar una concesión al estado transmitido más recientemente a su par.
