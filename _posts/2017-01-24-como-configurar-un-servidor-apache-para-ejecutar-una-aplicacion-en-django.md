---
layout: post
title: Cómo configurar un servidor Apache para ejecutar una aplicación en Django
published: true
---

Esta mini guía muestra de manera detallada paso a paso cómo preparar un servidor apache para que sea capaz de entender e interactuar con aplicaciones web creadas con Django (web framework de Python.), para empezar con la guía es necesario que tengas disponible un servidor Linux con permisos suficientes para ejecutar comandos en él; sin embargo, puedes también simplemente leer esta guía y enterarte un poco de qué va todo este tema.

Ya entrados en detalles, la configuración con la cual estaré trabajando en esta guía es de un servidor [Ubuntu 16.x](http://www.ubuntu.com/download/server) instalado en una [máquina virtual](https://www.virtualbox.org/wiki/Download).

### Consideraciones para la máquina virtual

Si estás empezando a trabajar con software de máquinas virtuales es importante que tengas claro que para poder seguir esta guía tu máquina virtual debe estar conectada a internet. Programas como virtual box tiene una opción dentro de la configuración de cada máquina bajo el menú de red que permite conectar dicha máquina bajo el adaptador de puente. Esto usualmente es suficiente para hacer que tu máquina virtual esté conectada a internet.

### Consideraciones para el servidor

Aunque es claro, que en la mayoría de casos cuando instalamos un programa o algún tipo de complemento en nuestras computadoras no es necesario reiniciar todo el sistema para poder empezar a usar dicho programa; personalmente recomiendo que cada determinado número de tareas reiniciemos el servidor para asegurarnos de que no tengamos futuros fallos en ejecución de servicios, por ejemplo.



Lo mínimo que necesitaremos en cuanto al servidor es asegurarnos de que tengamos instalado el servicio LAMP, si aún no tienes LAMP instalado puedes revisar [cómo instalar manualmente Apache, MySQL y PHP en ubuntu.](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu)


En cuanto a MySQL es importante tener configurado mysql_config como una variable de entorno, para ello podemos ejecutar el siguiente comando:

```bash
sudo apt-get install libmysqlclient-dev
```

Del mismo modo es recomendable que una vez el servidor se haya terminado de instalar, se ejecute el siguiente comando antes de instalar cualquier software adicional. Esto actualizará todas las dependencias del sistema.

```bash
sudo apt-get update && sudo apt-get upgrade
```

El entorno de consola de Ubuntu server puede ser muy complicado de manejar si no se tiene experiencia con el Shell Script, sin embargo considero que para facilitar el manejo del servidor se puede instalar el entorno gráfico y el manejador de usuarios.

Ubuntu GUI se instala con el comando:
```bash
sudo apt-get install ubuntu-desktop
```
Ubuntu Usuarios y Grupos
```bash
sudo apt-get install gnome-system-tools
```

Ahora que tenemos una interfaz gráfica, accedemos a un terminal y empezamos a instalar nuestras dependencias, empezaremos instalando el servidor ssh y git. Para ello solo ejecutamos los comandos.

```bash
sudo apt-get install openssh-server
sudo apt-get install git
```

Luego de esto, estamos listos para empezar a configurar el servidor para trabajar con Python, en este punto necesitamos darle a nuestro servidor la capacidad de crear entornos virtuales e instalar dependencias de Python.

```bash
sudo apt install virtualenv
sudo apt install python-pip
```

En caso de ya tener `pip` instalado, es recomendable actualizarlo.

```bash
pip install -U pip
```

Bien, en este punto ya tenemos la base lista para empezar a crear nuestro entorno, básicamente crearemos un espacio en nuestro servidor para poder almacenar una copia cifrada de nuestro repositorio y otro espacio para almacenar la información que consumirá apache.

```bash
sudo mkdir /var/www/git
sudo mkdir /var/www/public
sudo git init --bare /var/www/git/<reponame>.git
```

Ahora modificaremos el hook post update del repositorio para que cree el despliegue automático cuando hagamos push.

```bash
sudo nano /var/www/git/<reponame>.git/hooks/post-update
```

Una vez dentro hacemos que se vea algo parecido a esto:

```bash
#!/bin/bash
unset GIT_DIR
cd /var/www/public/<reponame>/
git fetch --force
git merge origin/master
if [ ! -d "env/" ]; then
  virtualenv env
fi
source env/bin/activate
pip install -r requirements.txt
python manage.py collectstatic
python manage.py makemigrations
python manage.py migrate
```

Genial, en este punto solo nos falta conectar los repositorios, para ello debemos clonar el repositorio en lo que se convertirá en nuestro directorio "publico" de apache.

```bash
cd /var/www/public/
sudo git clone /var/www/git/<reponame>.git
```

Este comando nos debió haber dejado una carpeta <reponame> dentro de public, el último paso para terminar la preparación de los directorios que recibirán nuestros archivos es darles permisos a nuestros directorios de forma recursiva. (Advertencia: si estas siguiendo esta guía en un servidor real al cual se conecten más personas, quizás debas considerar darle permiso solo a los directorios que acabamos de crear.)

```bash
sudo chmod 777 -R /var/www/
```

Perfecto, ahora solo hace falta crear el host virtual en apache para empezar a servir nuestras páginas web. Lo primero que tenemos que hacer es instalar wsgi_mod en apache para que sea capaz de comunicarse con Python.

```bash
sudo apt install libapache2-mod-wsgi
sudo service apache2 restart
```

Una vez apache se halla reiniciado tendremos el modulo instalado y activado,  a este punto solo nos queda configurar el host virtual.


```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/<reponame>.conf
sudo nano /etc/apache2/sites-available/<reponame>.conf
```

Ahora que tenemos creado el archivo del host virtual, solo es modificarlo para que se a semeje a lo siguiente:

```apacheconf
<VirtualHost *:80>  
  # Por convención personal uso para los nombres de dominio la siguiente convención
  # lenguaje.nombre_del_proyecto.rama_de_git
  ServerName python.<reponame>.dev
  ServerAdmin admin@email.com
  # El document root debe hacer referencia a la carpeta donde esta nuestro proyecto
  DocumentRoot /var/www/public/<reponame>/
  
  # El WSGIScriptAlias obtiene 2 parámetros el primero hace referencia a la ruta y el segundo al archivo wsgi.py de nuestro proyecto
  WSGIScriptAlias / /var/www/public/<reponame>/<projectname>/wsgi.py

  # El WDGIDaemonProcess recibe los siguientes parámetros
  # - ServerName
  # - El ejecutable de Python relacionado al DocumentRoot
  # - La cantidad de procesos, para lo cual es recomendable dejarlo en 2
  # - Los threads, del mismo modo se recomienda dejarlo en 15
  # - Y el nombre a mostrar
  WSGIDaemonProcess python.<reponame>.dev python-path=/var/www/public/<reponame>:/var/www/public/<reponame>/env/lib/python2.7/site-packages processes=2 threads=15 display-name=%{GROUP}
  # El WSGIProcessGroup hace referencia al ServerName
  WSGIProcessGroup python.<reponame>.dev
  
  <Directory /var/www/public/<reponame>/<projectname>/>
    <Files wsgi.py>
      Require all granted
    </Files>
  </Directory>
  
  Alias /robots.txt /var/www/public/<reponame>/static/robots.txt
  Alias /favicon.ico /var/www/public/<reponame>/static/favicon.ico
  
  # El directorio "static" será el encargado de servir los archivos css, js, etc...
  Alias /static/ /var/www/public/<reponame>/static
  
  <Directory /var/www/public/<reponame>/static>
    Require all granted
  </Directory>
  
  # El directorio "media" se encarga en este caso de servir las imágenes, videos y demás
  Alias /media/ /var/www/public/<reponame>/media/
  
  <Directory /var/www/public/<reponame>/media>
    Require all granted
  </Directory>

  # LogLevel info ssl:warn

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  # Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Con este paso terminado, aún nos falta activar nuestro host y reiniciar apache para que lo reconozca.

```bash
sudo a2ensite <reponame>.conf
sudo service apache2 restart
```

Perfecto, a este punto tenemos nuestra máquina virtual lista para servir nuestro proyecto escrito en Python. Ahora debemos hacer que nuestra computadora se comunique con el servidor, este proceso es corto y debe ir tanto en nuestro servidor como en nuestra computadora.

```bash
# UNIX
sudo nano /etc/host
# Windows (Como administrator)
notepad C:\Windows\System32\drivers\etc\hosts
```

Ahora agregamos al host el host name que asignamos en nuestro host virtual.

```bash
# Servidor
127.0.0.1   python.<reponame>.dev
# Cliente (ip del servidor)
x.x.x.x     python.<reponame>.dev
```

Con esto ya podremos acceder a nuestro proyecto desde nuestra maquina consumiendo el servidor que acabamos de configurar.

Al principio de la guía configuramos un servidor git para hacer los despliegues automáticos al hacer push, es momento de hacer nuestro primer despliegue.

En tu computadora local clona el repositorio que acabamos de crear así.

```git
git clone <serveruser>@<serverip>:<repopath>.git
```

¡Y listo, solo hace falta crear tu proyecto de Django y hacer push!
