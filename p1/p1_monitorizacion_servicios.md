## Práctica 1. Monitorización de servicios de red
### Systemctl 
El comando systemctl es una herramienta que 
sirve para controlar el sistema y sus servicios (demonios). 
Tiene muchas utilidades como: Ver el estado de un servicio, 
iniciarlo, pararlo, recargarlo...etc.  
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

servicios de red del sistema:  
![La imagen no carga](imagenes/1.jpg)

### Journalctl
Journalctl se encarga de manejar todos los mensajes producios por: El kernel,
initrd, los servicios...etc. es decir, todos los mensajes tipo log que se 
producen en el sistema cuando hay algún evento.  
Si no se le pasa ningún parámetro adicional nos devolverá una amplia lista
con los registros del sistema, aunque lo lógico es filtrar esos resultados en
función de lo que necesitemos observar. Esas opciones son: 
- -n, --lines=(lleva al número de línea deseado)
- -r, --reverse (devuelve la entrada al revés)
- -o, --output=[short|short-full|short-iso|short-precise|verbose|export|json|cat] (el formato de los mensajes)
