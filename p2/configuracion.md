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
/home/compartido    ip_cliente(rw,sync,no_subtree_cheeck)
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

1. Para comprobar que se ha creado correctamente se puede crear un archivo en ese directorio y ver que existe en cualquiera de las máquinas.