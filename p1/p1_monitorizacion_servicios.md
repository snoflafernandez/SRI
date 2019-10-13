## Práctica 1. Monitorización de servicios de red
### Systemctl 
El comando systemctl es una herramienta que 
sirve para controlar el sistema y sus servicios (demonios). 
Tiene muchas utilidades como: Ver el estado de un servicio, 
iniciarlo, pararlo, recargarlo...etc.  
Su sintáxis es muy sencilla pues se realiza de la siguiente manera:


Para ver el estado de un servicio (ssh en este caso)
~~~
systemctl status sshd  
~~~
Para recargar un servicio
~~~
systemctl reload sshd 
~~~ 
Para iniciar un servicio
~~~
systemctl start sshd  
~~~
Para parar un servicio
~~~
systemctl stop sshd
~~~