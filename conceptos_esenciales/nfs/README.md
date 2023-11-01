# Network File System (NFS)
## Configuración de un servidor NFS

Configurar un servidor NFS es relativamente sencillo. Esencialmente, lo que se necesita hacer es instalar los paquetes necesarios, crear el archivo ```/etc/exports``` y asegurarse que los demonios (servicios) requeridos están corriendo.

En esta sección configuraremos un servidor NFS y nos conectaremos a este desde un nodo cliente. El servidor NFS será un sistema Ubuntu 20.04 para el cual realizaremos las siguientes tareas:

* Instalar NFS server.
* Permitir el montaje del directorio ```/export```

El nodo cliente será un sistema CentOS 8, en el que se realizarán las siguientes tareas:

* Instalar NFS como cliente.
* Montar el directorio export del servidor.

Dando por hecho que los dos nodos se encuentran en un ambiente protegido, un usuario con el mismo nombre e id en el cliente tendrá derecho de escribir en la partición NFS. El usuario root del cliente no tendrá ninguna prerrogativa en cuanto la gestión de los datos de la partición NFS.   

## Instalación de NFS server
Instalaremos NFS server en el sistema Ubuntu 20.04. Instalamos los paquetes necesarios:

```console
#> nfs-kernel-server
```
En caso que el sistema Ubuntu 20.04 fuera un cliente NFS, el paquete a instalar es el siguiente:

```console
#> nfs-common
```

A su vez, si el sistema CentOS fuera el servidor NFS, se instalaría el paquete:
```console
#> yum install nfs-utils
```

Necesitamos estar seguros que los servicios relacionados con NFS están disponibles de manera que al arranque del servidor:

```console
#> systemctl enable nfs-server
```

El siguiente paso es determinar que directorios de nuestro servidor estarán disponibles en nuestra red. Cualquier directorio del sistema de archivos de Linux es un candidato para un NFS export. Sin embargo, se considera una buena práctica crear un único directorio para albergar todos los recursos compartidos para posteriormente crear subdirectorios que se se compartirán con los clientes. Una convención es crear el directorio ```exports``` en la raíz del sistema de archivos. Utilizaremos esta convención, de manera que ejecutamos la siguiente instrucción:

```console
#> mkdir /exports
```

El proceso de agregar una exportación se realiza al editar el archivo ```/etc/exports``` al agregar una linea de configuración para cada exportación. Todos los directorios que se desean compartir deben ser colocados en este archivo. El formato de la línea de configuración es la siguiente:

```properties
export client(option) client2(option) client3(option) ... clientn(option)
```  

Donde:
* ```export```: indica el directorio que se comparte.
* ```client```: el cliente (máquina o red) con la cual se comparte el export.
* ```option```: opciones que utilizará el servidor.

El siguiente es un ejemplo del archivo ```/etc/exports```:
```properties
/exports/docs 10.10.10.0/24(ro,no_subtree_check)
/exports/images 10.10.10.0/24(rw,no_subtree_check)
/exports/downloads 10.10.10.0/24(rw,no_subtree_check)
```

La primera sección contiene el directorio que estamos exportando a los otros nodos de la red. En la segunda sección estamos limitando el acceso a los nodos dentro de la red 10.10.10.0/24. La tercera sección indica los permisos a los recursos compartidos:

* ```ro```: indica que el directorio compartido es de sólo lectura, es decir los nodos pueden montar el export y acceder-leer los archivos dentro de él, pero no pueden hacer cambios.
* ```rw```: indica que el directorio permite lectura-escritura, de manera que los nodos pueden montar el export, crear nuevos archivos y modificar los existentes.
* ```no_subtree_check```: Esta opción en particular controla si el servidor escanea o no el sistema de archivos subyacente cuando procesa acciones dentro de las exportaciones, lo que puede aumentar un poco la seguridad pero reducir la confiabilidad.
* ```no_root_squash```: al definir esta opción se permite al usuario root del nodo cliente tener acceso de root a los archivos contenidos dentro del export.
* ```root_squash``` : es el contrario a ```no_root_squash```, que asigna el usuario root a nadie.
* ```sync```: Contesta las peticiones únicamente cuando el archivo esté escrito en el disco. Su contraria es ```async``` (con riesgo de pérdida de datos en caso de reinicio del servidor)

## Montar el directorio compartido en el nodo cliente

El nodo cliente monta el directorio compartido a su sistema de archivos local con la siguiente instrucción:

```console
#> mount 10.10.10.100:/exports/downloads /mnt/downloads
```

Donde la dirección IPv4 10.10.10.100 es la del servidor NFS.

## Asegurando NFS

La seguridad por default en NFS versión 4 está basada en AUTH_SYS, esto es, en la correspondencia entre UID/GID de los usuarios servidor/cliente.

>NFS versión 4 permite utilizar un control más estricto basado en el protocolo de autenticación Kerberos.

Cuando en un sistema Linux se crea un usuario o un grupo, se asigna un **UID (User ID)** and **GID (Group ID)**. A menos que exista coincidencia entre las cuentas de usuario en todos los sistemas en el mismo orden, lo más probable es que los UID y GID difieran en cada nodo. El archivo ```idmapd``` nos ayuda a mapear esos UIDs de un sistema a otro.

## Iniciar el servicio NFS

En Ubuntu 20.04 iniciamos el servicio NFS al ejecutar la siguiente instrucción:

```console
#> systemctl start nfs-kernel-server
```  

En el caso de CentOS, ejecutamos la siguiente instrucción:
```console
#> systemctl start nfs-server
```
