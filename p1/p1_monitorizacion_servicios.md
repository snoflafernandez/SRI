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

**Activar el registro de logs para el servicio SSH**  
A través de journalctl se pueden listar todos los logs del sistema y estos se pueden filtrar por servicios o programas: 
~~~
journalctl -u ssh
~~~
Al filtrarlos utitilizando el comando anterior se listan todos los logs emitidos por el sistema para el programa SSH.  

**Probar conexión varias veces, con fallo y de manera exitosa**  
Cuando se intenta acceder via SSH al sistema se producen dos mensajes de log, tanto si es con fallo como si es exitosa.  
Si se produce un fallo al acceder via SSH devolverá los siguientes fallos:  
- pam_unix(ssh:auth) authentication failure[...]rhost=*dirección cliente* user=*usuario*
- Failed password for usuario from *dirección cliente*  
  
Si por el contrario el acceso es exitoso se devuelven los siguientes mensajes:  
- Accepted password for usuario from *dirección cliente*
- pam_unix(ssh:session): session opened for user usuario

**Mostrar los ultimos 10 mensajes generados por le servicio sshd y, exportarlos en formato json.**
~~~
journalctl -u ssh -n 10 -o json> logs_ssh.json
~~~

**Configurar journalctl para hacer persistentes los registros, pero que no ocupen más de 100mb de espacio en disco**
En primer lugar es necesario abrir el archivo de configuración de journalctl que se encuentra en la ubicacion */etc/systemd/journald.conf*.  
En ese archivo hay que descomentar las lineas:  
- Storage=auto
- SystemMaxFiles=100  
Posteriormente creamos el directorio */var/log/journal* y reiniciamos el servicio (se perderán todos los mensajes procedentes de esa sesión).

**Mostrar los registros autorizados de acceso remoto de la últimos 5 usuarios.**
Los registros de acceso remoto de los usuarios se enumeran desde 0 hasta el numero de usuarios que acceda remotamente. De esta manera si se utiliza el comando journalctl con el usuario que se quiere listar se podrá ver sus registros:
~~~
journalctl -t sshd -b[0,1,2,3,4...]
~~~


**Revisar los intentos con error de acceso generados por el servicio sshd en las últimas 24 horas.**
Se utiliza la palabra clave *until* o la palabra *since* y se puede establecer la hora desde que lo quieres o el intervalo de tiempo atrás. Por ultimo se especifica que se buscan entradas que hayan devuelto un error:
~~~
journalctl -u ssh --until "24 hour ago" --priority=err
~~~