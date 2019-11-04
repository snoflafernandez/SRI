## Práctica 1. Monitorización de servicios de red
### Systemctl 
El comando systemctl es una herramienta que  sirve para controlar el sistema y sus servicios (demonios). Tiene muchas utilidades como: Ver el estado de un servicio, iniciarlo, pararlo, recargarlo...etc.  
Su sintáxis es muy sencilla pues se realiza de la siguiente manera:

~~~
Para ver el estado de un servicio (ssh en este caso)
systemctl status sshd 
~~~
~~~
Para recargar un servicio
systemctl reload sshd 
~~~
~~~
Para iniciar un servicio
systemctl start sshd  
~~~
~~~
Para parar un servicio
systemctl stop sshd
~~~

**Servicios de red del sistema**:  
![La imagen no carga](../imagenes/1.jpg)

### Journalctl
Journalctl se encarga de manejar todos los mensajes producios por: El kernel, initrd, los servicios...etc. es decir, todos los mensajes tipo log que se producen en el sistema cuando hay algún evento.  
Si no se le pasa ningún parámetro adicional nos devolverá una amplia lista con los registros del sistema, aunque lo lógico es filtrar esos resultados en función de lo que necesitemos observar. Esas opciones son: 
- -n, --lines=(lleva al número de línea deseado)
- -r, --reverse (devuelve la entrada al revés)
- -o, --output=[short|short-full|short-iso|short-precise|verbose|export|json|cat] (el formato de los mensajes)

### Netstat 
Netstat es una herramienta que permite identificar las conexiones TCP que están activas en el equipo donde se está utlizando la herramienta. Tambien muestra una lista con los puertos TCP y UDP que se encuentran abiertos.  
Para utilizar la herramienta es necesario instalar el paquete net-tools. Las opciones de la herramienta son: 
~~~
netstat [-a] [-b] [-e] [-f] [-n] [-o] [-p Protocolo] [-r] [-s] [-t] [-x] [-y] [Intervalo]
~~~

### SSH
**Cambiar puerto SSH**  
Para cambiar el puerto por el que escucha este servicio es necesario acceder a la terminal como superusuario y editar el archivo */etc/ssh/sshd_config*, buscar la línea "# Port 22" la cual se debe descomentar y cambiar por el puerto desado.  
Una vez realizados los cambios es necesario guardar y reiniciar el servicio sshd:
~~~
systemctl restart sshd
~~~  

**Crear una lista blanca de usuarios permitidos**  
Se puede crear una lista blanca de usuarios permitidos de dos maneras diferentes, una a traves de direcciones IP y otra a través de usuarios:
1. IP  
Para hacerlo a través de direcciones IP debemos modificar las iptables (cortafuegos del kernel de Linux). Debemos escribir los siguientes comandos:  
~~~
iptables -A INPUT -s [direccion IP] --dport [puerto SSH] -i eth0 -j ACCEPT  

iptables -A OUTPUT -s [direccion IP] --dport [puerto SSH] -o eth0 -j ACCEPT 
~~~
2. Usuario  
Para filtar por nombre de usuario en el archivo */etc/ssh/sshd_config* es necesario añadir el nombre de usuario permitido de la siguiente manera y después es necesario reiniciar el servicio.
~~~
AllowUsers [nombre_de_usuario]
~~~

**Permitir acceso a root solo con clave instalada en el servidor**  
Dentro del archivo */etc/ssh/sshd_config* debemos buscar la directiva "PermitRootLogin", se encuentra en el apartado *# Authentication*. Una vez encontrada la línea es necesario sustituir la opcion que trae por defecto (prohibit-password) por la palabra "no" y descomentar la línea. Posteriormente es necesario reiniciar el servicio:
~~~
PermitRootLoging no
~~~