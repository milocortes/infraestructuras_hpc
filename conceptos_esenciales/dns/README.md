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
  [root@dns-server ~]# nmcli connection modify enp0s8 ipv4.dns 192.168.1.250
  [root@dns-server ~]# nmcli connection up enp0s8
```

3. Editar el archivo de configuración ```/etc/named.conf```

```properties
[...]
options {
	listen-on port 53 { 127.0.0.1; 192.168.1.250; }; /* IP del servidor DNS*/
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost; 192.168.1.0/24;};/*Segmento de red que puede consultar al DNS*/

[...]

zone "." IN {
	type hint;
	file "named.ca";
};

zone"diplomado.hpc" IN { /* Dominio que se usa para la resolucion */
 type master; /* Tipo de DNS */
 file "/var/named/diplomado.hpc"; /* Archivo de resolucion por dominio */
 allow-update { none; };
};
zone"1.168.192.in-addr.arpa" IN { /* Archivo para asociar de forma inversa las IPs con el dominio */
 type master; /* Tipo de DNS */
 file "/var/named/1.168.192.in-addr"; /* Archivo de registros de resolución inversa */
 allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

4. Crear un archivo de resolución por dominio. Se creará el archivo ```/var/named/diplomado.hpc```:
```properties
$TTL 86400
@ IN SOA dns-server.diplomado.hpc. root.diplomado.hpc. (
 2011071001 ; Serial
 3600 ; Refresh
 1800 ; Retry
 604800 ; Expire
 86400 ; Minimum TTL
)
@ IN NS dns-server.diplomado.hpc.
@ IN A 192.168.1.250
@ IN A 192.168.1.1
@ IN A 192.168.1.2
@ IN A 192.168.1.3
dns-server IN A 192.168.1.250
n1 IN A 192.168.1.1
n2 IN A 192.168.1.2
n3 IN A 192.168.1.3
```
>#### Significado de los parámetros
* Serial: El número de serie de la zona. Si un servidor de nombres secundario esclavo de este observa un aumento en este número, el esclavo asumirá que la zona ha sido actualizada e iniciará una transferencia de zona.
* Refresh: Es el número de segundos entre solicitudes de actualización de los servidores de nombres secundarios y esclavos.
* Retry: Es el número de segundos que el secundario o esclavo esperará antes de volver a intentarlo cuando el último intento haya fallado.
*Expire: Es el número de segundos que un maestro o esclavo esperará antes de considerar los datos obsoletos si no pueden llegar al servidor de nombres principal.
* Minimum TTL: utilizado anteriormente para determinar el TTL mínimo, se utiliza para el almacenamiento en caché. Este es el TTL predeterminado si el dominio no especifica un TTL.

5. Verificar la configuración del archivo de resolución por dominio:

```console
[root@dns-server ~]# named-checkzone dns-server.local /var/named/diplomado.hpc
```

6. Crear archivo de resolución inversa ```/var/named/1.168.192.in-addr```:
```properties
$TTL 86400
@ IN SOA dns-server.diplomado.hpc. root.diplomado.hpc. (
 2011071001 ; Serial
 3600 ; Refresh
 1800 ; Retry
 604800 ; Expire
 86400 ; Minimum TTL
)
@ IN NS dns-server.diplomado.hpc.
@ IN PTR diplomado.hpc.
dns-server IN A 192.168.1.250
n1 IN A 192.168.1.1
n2 IN A 192.168.1.2
n3 IN A 192.168.1.3
250 IN PTR dns-server.diplomado.hpc.
1 IN PTR n1.diplomado.hpc.
2 IN PTR n2.diplomado.hpc.
3 IN PTR n3.diplomado.hpc.
```

7. Verificar la configuración del archivo de resolución inversa por dominio:
```console
[root@dns-server ~]# named-checkzone dns-server.local /var/named/1.168.192.in-addr
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
