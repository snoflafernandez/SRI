## Prerrequisitos
En primer lugar debe haber instaladas 3 máquinas virtuales, un Debian y dos Proxmox (nodos).
### Debian
Esta máquina debe de tener dos tarjetas de red: 
- Una en modo "bridge" para comunicarse con la red local (con una dirección estática).
- Otra con conexión a internet conectada a la tarjeta de red (pide por DHCP).

El objetivo es hacer que esta máquina funcione como router de la red local.
Por tanto el archivo interfaces es similar al siguiente:  
~~~
auto ens33
iface ens33 inet static
address 10.10.2.132
netmask 255.255.255.0

auto ens37
iface ens37 inet dhcp
~~~
También es necesario crear una serie de reglas para configurar NAT de manera que la dirección estática de los nodos sea traducida y pueda salir al exterior. Estos comandos son: **(necesario cada vez que se inicia la máquina)**
~~~ 
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o ens37 -j MASQUERADE
~~~  

### Proxmox
En los nodos se debe configurar una dirección ip estática en la que la **puerta de enlace** será la dirección de la máquina Debian.  
Por otro lado, es importante prestar atención al nombre de las máquinas ya que si es el mismo en ambas (p.e. si ha sido clonada) al tratar de añadir el segundo nodo al cluster devolverá un fallo. Para cambiar el nombre hay que modificar dos archivos:
~~~
/etc/hosts -> Cambiar el nombre de usuario en el dominio y en el nombre.
/etc/hostname -> Cambiar el nombre de usuario.
~~~
Por último se reinicia el sistema.

## NFS
Es necesaria la existencia de un servidor NFS para compartir archivos entre las máquinas de manera que la imagen de la máquina virtual pueda estar en los distintos nodos. Para configurarlo hay que distinguir dos partes: Servidor (Debian) y Clientes (Proxmox)
### Servidor
1. Instalar las herramientas necesarias para crear un servidor NFS.
~~~ 
apt-get install nfs-kernel-server nfs-common
~~~

2. Crear el directorio compartido.
~~~
mkdir /home/compartido
chown nobody:nogroup /home/compartido
chmod 755 /home/compartido

mkdir /var/www
chown root:root /var/www
chmod 755 /var/www
~~~

3. Añadir los clientes al archivo */etc/exports* (tantas veces como clientes haya).
~~~
/home/compartido    ip_cliente(rw,sync,no_subtree_check)
/var/www    ip_cliente(rw,sync,fsid=0,crossmnt,no_subtree_check,no_root_squash)
~~~

4. Reiniciar el servicio *Kernel nfs server* para guardar los cambios.
~~~
service nfs-kernel-server restart
~~~

### Cliente
1. Instalar cliente NFS.
~~~
apt-get install nfs-common
~~~

2. Crear los directorios donde montar los archivos compartidos.
~~~
mkdir -p /mnt/nfs/home/compartido
mkdir -p /var/www
~~~

3. Montar los directorios **(necesario cada vez que se inicia la máquina)**
~~~
mount ip_servidor:/home/compartido /mnt/nfs/home/compartido
mount ip_servidor:/var/www /var/www
~~~

4. Para comprobar que se ha creado correctamente se puede crear un archivo en ese directorio y ver que existe en cualquiera de las máquinas.

### Contenedor Proxmox
1. En primer lugar es necesario descargar una plantilla (están disponibles en la documentación de Proxmox download.proxmox.com/images/system) y guardarla en el directorio compartido.
   
2. Crear el contenedor con la imagen .tar.gz
~~~
pct create 100 /mnt/nfs/home/compartido/archivo.tar.gz -storage local-lvm --password contraseña
~~~
**Los siguientes pasos son opcionales ya que no es necesario iniciar la máquina**

3. Iniciar la máquina y entrar en la consola:
~~~
pct start 100
pct console 100
~~~

4. Apagar la máquina
~~~
Acceder a la máquina vía SSH (si se ha accedido a la consola).
pct stop 100
~~~

### Crear cluster Proxmox, añadir nodo al cluster y migrar la máquina.
1. Crear el cluster:
~~~
pvecm create CLUSTERNAME
~~~

2. Añadir nodo al cluster:
~~~
Acceder vía SSH a la segunda máquina.
pvecm add IP_DE_UN_NODO_EXISTENTE
~~~

3. Listar nodos y ver estado del cluster:
~~~
Listar nodos
pvecm node

Estado del cluster
pvecm status
~~~

4. Migrar máquina de un nodo a otro:
~~~
pct migrate id_maquina USUARIO_DE_OTRO_NODO
~~~

5. Comprobar que la máquina ha sido migrada:
~~~
Ejecutar el siguiente comando en ambos nodos y ver que no está en el primero y si en el segundo
pct list
~~~