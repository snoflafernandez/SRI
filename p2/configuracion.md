## Prerrequisitos
En primer lugar debe haber instaladas 3 máquinas virtuales, dos Proxmox (nodos) y un Debian.
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
También es necesario crear una serie de reglas para configurar NAT de manera que la dirección estática de los nodos sea traducida y pueda salir al exterior. Estos comandos son:
~~~ 
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o ens37 -j MASQUERADE
~~~  

### Proxmox
En los nodos se debe configurar una dirección ip estática en la que la **puerta de enlace** será la dirección de la máquina Debian.  
Por otro lado es importante prestar atención al nombre de las máquinas ya que si es el mismo en ambas (p.e. si ha sido clonada) al tratar de añadir el segundo nodo al cluster devolverá un fallo. Para cambiar el nombre hay que modificar dos archivos:
~~~
/etc/hosts -> Cambiar el nombre de usuario en el dominio y en el nombre.
/etc/hostname -> Cambiar el nombre de usuario.
~~~

