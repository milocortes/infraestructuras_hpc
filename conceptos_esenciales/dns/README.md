# Servicio DNS en Linux

El sistema de Nombres de Dominio (DNS) proporciona una forma de asignar nombres a números. Por ejemplo, un servidor web con dirección IPv4 93.184.216.34 puede ser asignado a un nombre example.com. El servicio DNS asigna nombres de dominio con direcciones IP, lo que facilita usar nombres de dominio fáciles de recordar.

El más común servicio DNS que levantan las organizaciones es un **servidor DNS interno** para el uso de sus propios usuarios. Este servidor probablemente tiene un archivo poblado con registros DNS para resolución de dominios interno. Este archivo puede ser poblado al editarse manualmente o automáticamente mediante el auto registro de los clientes o a partir de un sevicio DHCP (Dynamic Host Configuration Protocol).

## Instalación y configuración de un servidor DNS en CentOS 8

Prerrequisito: Agregar una nueva interfaz de red interna.

Renombramos el hostname al editar los archivos ```/etc/hostname``` y ```/etc/hosts``` por el hostname ```ns-server```.

1. Instalar el servidor DNS:
```console
[root@dns-server ~]# dnf install bind bind-utils
```

2. Configurar la interface de red interna con los siguientes parámetros:
  * IP: 192.168.1.250
  * DNS: 192.168.1.250

  ```console
  [root@dns-server ~]# nmcli connection add con-name enp0s8 ifname enp0s8 type ethernet
  [root@dns-server ~]# nmcli connection modify enp0s8 ipv4.addresses 192.168.1.250/24
  [root@dns-server ~]# nmcli connection modify enp0s8 ipv4.method manual
  [root@dns-server ~]# nmcli connection modify enp0s8 ipv4.dns "192.168.1.250"
  [root@dns-server ~]# nmcli connection up enp0s8
```

3. Editar el archivo de configuración ```/etc/named.conf```

```console

```

4. Crear un archivo de resolución por dominio. Se creará el archivo ```/var/named/diplomado.hpc```:

5. Verificar la configuración del archivo de resolución por dominio:

```console
[root@dns-server ~]# named-checkzone dns-server.local /var/named/diplomado.hpc
```

6. Crear archivo de resolución inversa:
```console
[root@dns-server ~]# vi /var/named/1.168.192.in-addr
```

7. Verificar la configuración del archivo de resolución inversa por dominio:
```console
[root@dns-server ~]# named-checkzone dns-server.local /var/named/1.168.192.in-
addr
```

8. Arrancar el servidor DNS:
```console
[root@dns-server ~]# systemctl start named.service
```

9. Habilitar el servidor DNS:
```console
[root@dns-server ~]# systemctl enable named.service
```

10. Abrir el puerto del servidor DNS:
```console
[root@dns-server ~]# firewall-cmd --zone=public --add-port=53/udp --permanent
[root@dns-server ~]# firewall-cmd --reload
```

11. Verificar el funcionamiento en el servidor DNS:

12. Probar el servidor DNS desde una máquina cuya IP sea 192.168.1.1, instalar ```bind-utils```:
