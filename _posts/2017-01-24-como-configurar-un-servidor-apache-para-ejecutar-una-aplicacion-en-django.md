---
layout: post
title: Como configurar un servidor Apache para ejecutar una aplicación en Django
published: true
---

Este tutorial te guiara paso a paso hacia la configuración de un servidor apache que ejecute aplicaciones creadas con Django el web framework de Python. Este archivo está pensado para que cualquier persona con algunos conocimientos básicos pueda crear su propio servidor capaz de servir páginas web creadas con Django, ya sea para desarrollo en vivo o para crear un servidor virtual en el propio equipo.

Para esta guía se trabajará con una máquina virtual corriendo bajo [virtualbox](https://www.virtualbox.org/wiki/Downloads) la cual tiene instalado el servidor [Ubuntu Server 14.04](http://www.ubuntu.com/download/server)

## Índice
- [Consideraciones para el VirtualBox](#consideraciones-para-el-virtualbox)
- [Consideraciones para el servidor](#consideraciones-para-el-servidor)
- [Instalando y configurando GIT en el servidor](#instalando-y-configurando-git-en-el-servidor)
- [Configuración de Apache y WSGI_MOD](#configuracion-de-apache-y-wgsi_mod)

### Consideraciones para el VirtualBox

Lo más importante para este proyecto es, una vez se tenga la máquina virtual configurada asegurarse que en la pestaña de 'Red' la opción 'Cable Conectado' bajo el menú de avanzadas este seleccionada. Una vez, se halla verificado esta configuración se puede proceder a iniciar la máquina virtual o a instalar el servidor.

### Consideraciones para el servidor

Pese a que no es necesario, personalmente recomiendo reiniciar el servidor o la máquina virtual bien sea después de cada instalación o después de un grupo de instalaciones para asegurarse de que las instalaciones se completen y los servicios de ser el caso se inicien.

Lo más importante al momento de estar instalando el servidor en nuestra máquina virtual es asegurarnos de seleccionar el servicio LAMP; sin embargo, si por error no se ha instalado se puede [instalar manualmente apache, mysql y php](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu).

En cuanto a MySQL es importante tener configurado mysql_config como una variable de entorno, para ello podemos ejecutar el siguiente comando:

```sh
sudo apt-get install libmysqlclient-dev
```

Del mismo modo es recomendable una ves el servidor se halla terminado de instalar ejecutar el siguiente comando antes de instalar cualquier software adicional

```sh
sudo apt-get update && sudo apt-get upgrade
```

El entorno de consola de Ubuntu server puede ser muy complicado de manejar si no se tiene experiencia con el Shell Script sin embargo considero que para facilitar el manejo del servidor se puede instalar el entorno grafico y el manejador de usuarios.

Ubuntu GUI se instala con el comando:
```sh
sudo apt-get install ubuntu-desktop
```
Ubuntu Usuarios y Grupos
```sh
sudo apt-get install gnome-system-tools
```
Luego de esto estamos listos para empezar a configurar el servidor para trabajar con Python

### Instalando y Configurando Git en el servidor

Antes de instalar Git dentro de nuestros servidores es importante instalar openSSH el cual nos permitira conectarnos al servidor de manera remota para poder asi ejecutar comandos como el de clonar el repositorio en git desde nuestro servirdor a nuestro equipo, para ello ejecutamos el comando:

```sh
sudo apt-get install openssh-server 
```
Luego de tener el servidor ssh activo es momento de instalar el git en nuestro servidor el cual nos permitira hacer deploy de nuestras aplicaciones, asi.
```sh
sudo apt-get install git
```
En este pundo ya tenemos git instalado dentro de nuestro servidor, el siguiente paso es configurar una carpeta para que sea la que manejara el repositorio y otra que sera hacia donde apuntara nuestro servidor Apache para poder servir las paginas.

Si en algun momento no se puede hacer ninguna de las acciones que siguen de aqui en adelante puede ser que el directorio o archivo no tiene suficientes permisos, para ello se puede ejecutar el comando `sudo chmod -R 777 /path/to/folder/file` y podria ser suficiente para continuar.

```sh
sudo mkdir /var/www/git (Directorio donde se guardaran los repositorios)
sudo mkdir /var/www/public (Directorio donde se haran los despliegues)
```
Luego de tener esto listo lo siguiente es crear el repositorio que almacenara nuestro proyecto y luego de ello lo configuramos para hacer despliegues automaticos a nuestra carpeta publica la cual estará observando nuestro Apache.

```sh
sudo git init --bare /var/www/git/djangoProject.git

#Ahora activamos el hook "post-update" de git y lo editamos con lo siguiente
sudo nano /var/www/git/djangoProject.git/hooks/post-update

#Este debe ser el contenido final de nuestro hook
---
#!/bin/sh
unset GIT_DIR
cd /var/www/public/djangoProject/
git fetch --force
git merge origin/master

#Es momento de clonar nuestro projecto en la carpeta publica y configurarlo para recivir el projecto de Django
sudo mkdir /var/www/public/djangoProject
cd /var/www/public/djangoProject
sudo git clone /var/www/git/djangoProject.git .

#Una ves clonado el repositorio editamos el archivo "post-merge"
sudo echo >> /var/www/public/djangoProject/.git/hooks/post-merge
sudo nano /var/www/public/djangoProject/.git/hooks/post-merge

#El archivo final debe se algo como esto
---
#!/bin/sh
cd /var/www/public/djangoProject/
if [ ! -d "env/" ]; then
  virtualenv env
fi
source env/bin/activate
pip install -r requirements.txt
python manage.py collectstatic
python manage.py makemigrations
python manage.py migrate
```

### Configuracion de Apache y WGSI_MOD

Ya en este punto solo hace falta instalar el wsgi_mod y crear el virtual host para servir nuestra aplicacion. Para instalar el wsgi_mod lo descargamos por aptitude y luego reiniciamos el servidor apache asi:

```sh
sudo aptitude install libapache2-mod-wsgi (v14.x)
sudo aptitude install libapache2-mod-wsgi (v16.x)
sudo service apache2 restart
```
En este momento ya tenemos la libreria wsgi_mod instalada y ejecutandose desde nuestro servidor es momento de crear el archivo de configuración para nuestro virtual host.

```sh
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/djangoProject.conf
#Es importante tener en cuenta que la extencion del archivo sea .conf de lo contrario no funcionará

#Luego de haber copiado el archivo lo editamos
sudo nano /etc/apache2/sites-available/djangoProject.conf

#El archivo final debe quedar asi:
---
<VirtualHost *:80>
  #Por convencion personal uso para los nombres de dominio la siguiente convencion
  #lenguaje.nombre_del_proyecto.rama_de_git
  ServerName python.djangoproject.dev
  ServerAdmin admin@email.com
  #El document root debe hacer referencia a la carpeta donde esta nuestro projecto
  DocumentRoot /var/www/public/djangoProject/
  
  #El WSGIScriptAlias optiene 2 parametros el primero hace referencia a la ruta y el segundo al archivo wsgi.py de nuestro projecto
  WSGIScriptAlias / /var/www/public/djangoProject/djangoProject/wsgi.py
  #El WDGIDaemonProcess recibe los siguientes parametros
  # - ServerName
  # - El ejecutable de python relacionado al DocumentRoot
  # - La cantidad de procesos, para lo cual es recomendable dejarlo en 2
  # - Los threads, del mismo modo se recomienda dejarlo en 15
  # - Y el nombre a mostrar
  WSGIDaemonProcess python.djangoproject.dev python-path=/var/www/public/djangoProject:/var/www/public/djangoProject/venv/lib/python2.7/site-packages processes=2 threads=15 display-name=%{GROUP}
  # El WSGIProcessGroup hace referencia al ServerName
  WSGIProcessGroup python.djangoProject.dev
  
  <Directory /var/www/public/djangoProject/djangoProject/>
  	<Files wsgi.py>
    		Require all granted
	</Files>
  </Directory>
	
	Alias /robots.txt /var/www/public/djangoProject/static/robots.txt
	Alias /favicon.ico /var/www/public/djangoProject/static/favicon.ico
	
	#El directorio "static" sera el encargado de servir los archivos css, js, etc...
  Alias /static/ /var/www/public/djangoProject/static
	
	<Directory /var/www/public/djangoProject/static>
	  Require all granted
	</Directory>
	
	#El directorio "media" se encarga en este caso de servir las imagenes, videos y demas
	Alias /media/ /var/www/public/djangoProject/media/
	
	<Directory /var/www/public/djangoProject/media>
	  Require all granted
	</Directory>

	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
---

#Una vez tengamos nuestro la configuracion del vhost creada debemos activar el host y reiniciar el servicio asi:
sudo a2ensite djangoProject.conf
sudo service apache2 restart

#Para terminar la configuracion del servidor solo falta agregar el ServerName a nuestro host agregando lo siguiente:
sudo nano /etc/hosts

#Y agregamos la siguiente linea
127.0.0.1	python.djangoproject.dev
```
