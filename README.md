# Born2beRoot
## Index
- [Instalación](#instalación)
- [Script](#script)
- [Bonus](#bonus)
- [Conceptos](#conceptos)
- [Lista de comandos básicos](#lista-de-comandos-básicos)

## Instalación
[Volver al índice](#born2beroot)
- [Parte 0: creación de maquina virtual](#parte-0:-creación-de-maquina-virtual)
- [Parte 1: lvm](#parte-1:-lvm)
- [Parte 2: login](#parte-2:-login)
- [Parte 3: instalación de herramientas](#parte3:-instalación-de-herramientas)
- [Parte 4: sudo](#parte-4:-sudo)
- [Parte 5: consexión segura](#parte-4:-conexión-segura)
- [Parte 6: instalar politica de contraseñas](#parte-6:-instalar-politica-de-contraseñas)
- [Parte 7: configurar una tarea cronometrada](#parte-7:-configurar-una-tarea-cronometrada)

## Parte 0: CREACIÓN DE MAQUINA VIRTUAL
```
0.  Crear carpeta en sgoinfree/nuestrousuario
1.  Descargar iso Debian de la página web de Debian			 Comprobar, arquitectura del procesador del ordnador
							 		debe ser la misma para poder virtualizar correctamente
							 		Debian 64 en este caso (amd64)
2.  Virtual Box --> Nueva	--> Memoria(Default) --> Disco Duro VDI	Tamaño indeterminado subject especifica simillar
3.  Config		--> Red  	--> Adaptador puente	 	Para conexiones posteriores del bonus
4.  Config		--> Pantalla--> VBoxVGA				Evita un bug al iniciar la máquina
5.  Seleccionamos la .iso como disco de arranque
```

## Parte 1: LVM
```
6.  Arrancar la máquina virtual
7.  Idioma, localizacion, y idioma de teclado (locale configuration)
8.  Hostname: "login42", usuario: "login", y contraseña y contraseña del root
9. Zona horaria
10. Particiones: Manual 						Elegimos el disco duro par la partició con el tamaño completo

	Particion primaria (physical)
		al principio del espacio disponible
		montandolo en /boot 					Será sda1, la partición donde dónde  el arranque del sistema
	
	Partición lógica (Logical Volume):
		Seleciomamos el reto de la partición (volumen físico par encriptación)
		Configurar volumenes encriptados-->Crear-->Seleccionamos el volumen a encriptar
		Elegimos contraseña de encriptacion
		Particiones del LVM:
			- Creamos Logical Group en la particion logica. Nombre: LVMGroup 
			- Creamos los Logical Volumes uno a uno, del tamaño correspondiente 
				(var-log solo necesita un guion en el nombre)
			- Montamos los volumenes de acuerdo al subject, con sistema archivos Ext4
				Aqui seleccionamos para que va a ser cada uno (/root, /home, etc).
				El var log debemos montarlo escribiendolo manualmente (/var/log)
				Además, el swap usa tipo de archivos especial, swap area
		Seleccionamos el mirror desde donde descargara los paquetes el apt
		Deseleccionamos los paquetes extra ESPECIALMENTE LA INTERFAZ GRAFICA
```

## Parte 2: LOGIN
```
11.	Usar password de encriptación
12.	Login: Preferible root para el setup
13.	lsblk					<- 	Comprobamos que las particiones son correctas
14.	aa-status				<- 	Comprobamos el estado de AppArmor. Deberia estar instalada y funcional por defecto
15. aptitude install apparmor			<-  En caso de no estar instalado lo instalamos	
16. systemctl enable apparmor			<- Configuramos para que funcione al inicio
```

## Parte 3: INSTALACIÓN DE HERRAMIENTAS
```
17. su - 					<-  Para instalar sudo debemos estar en root. Si no: "su -" --> "root password"
18. apt-get update -y				<-  Actualizamos apt
19. apt-get upgrade -y
20. apt-get install aptitude
21. apt-get install git -y 		
22. apt-get install wget
23. apt-get install vim
```

## Parte 4: SUDO
```
24.	apt install sudo			<-  Instalación de sudo
25.	dpkg -l | grep sudo			<-  Comprobamos que este instalado correctamente
	systemctl status sudo
26. adduser <usuario> sudo			<-  Añadimos el usuario al grupo sudo
    gpasswd -a hcastanh sudo			<-  Opción (2) para añadir -a añadir -d borrar
	usermod -aG sudo your_username		<-  Opción (3) debemos ser root
27. getent group sudo				<-  Comprobacion usuario
28.	sudo reboot				<-  Reiniciar sudo
29.	sudo -v					<-  Comprobación sudopowers

	Configuración sudo: 
		/etc/sudoers			a)  archivo de configuración inicial
		/etc/sudoers.d/<otras_config>	b)  posibilidad de crear nuevos archivos de configuración de sudo
						    (El archivo sin ~ ni)
		Añadir la línea
		tuusuario_42 	ALL=(ALL) ALL NOPASSWD: /usr/local/bin/monitoring.sh
		

30. sudo vim /etc/sudoers.d/sudo-conf		a) Modo de editar con el programa de la elección específico
	sudo -e /etc/sudoers.d/sudo-conf	b) Modo de editar (da a alegir entre los programas instalados)

	Defaults passwd_tries=3			<- Maximo numero de intentos para acceder a sudo
	Defaults badpass_message="error"	<- Mensaje de error personalizado
	Defaults log_input,log_output 		<- Guardado de los imputs y los okutputs
	Defaults iolog_dir="/var/log/sudo"	<- Lugar de guardado de los logs 
						   (hay que crear el directorio manualmente)
	Defaults logfile="/var/log/sudo/sudo.log"  Log contínuo de los sudo ejecutados
	Defaults requiretty 			<- Obligamos a que solo se pueda usar desde terminal.
						   Ningún proograma podrá usar sudo si no es desdde consola
						   Ver conceptos [TTY](#tty)
	Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

31. sudo mkdir /var/log/sudo			<- Creamos la carpeta y el archivo donde vamos a guardar los logs
						   Estando dentro de la carpeta, ejecutar el comando:
						    sudo touch /var/log/sudo/sudo.log
32. less /etc/passwd				<- Ver todos los usuarios en el equipo
33. less /etc/group				<- Ver todos los grupoes en el equipo
```

## Parte 5: CONEXIÓN SEGURA
### SSH
con autenticación y encriptacion de datos
```
34. apt install openssh-server
35. systemctl enable sshd
36. vim /etc/ssh/sshd_config 
línea13	Port 22 --> Port 4242 			<- Cambiamos el puerto por defecto que usa el SSH por el 4242
línea32	PermiteRootLogin prohibit-password  	<- Cambiar por -> PermitRootLogin
```

### UFW
Uncomplicated Firewall. Es un firewall facil de configurar. Monitoriza los puertos
```
37.	apt install ufw
dpkg -l | grep ufw				<- comprobar si está correctamente instalado
38.	ufw enable
39. systemctl enable ufw			<- Activamos que se lance el ufw en cada inicio
40.	ufw status				<- Muestra los puertos Activamos
41.	ufw allow 4242 				<- Permite el puerto 4242, que segun el subject es el unico debido
42.	ufw delete <numeroderegla>		<- Elimina la regla numerada
43. sudo ufw allow ssh				<- Permitir la encriptación ssh en los puertos
44. sudo ufw allow 4242				<- Permitir el puerto 4242 con el que haremos la conexión remota
	Desde consola externa, comprobamos la conexión remota:
		ssh user@serverIP -p 4242 	<- Conectamos con usuario a la ip y en el puerto 4242
		scp -P <puerto> <archivoe> <user>@<Tu IP>:<nueva ruta> <- Manadr archivo por SSH
		(ifconfg -a para comprobar la direccion ip)
45. ss -tunlp 					<- comporbar que solo hay un socket abierto
	*Debemos comprobar que la conexión SSH es el único socket disponible.
 	 Un socket: concepto abstracto por el cual dos procesos (posiblemente situados en computadoras distintas)
	 pueden intercambiar cualquier flujo de datos:
	 Un Socket es específico de un protocolo, máquina y puerto

46. apt install net-tools
47. ifconfig -a
48. route -n					<- Compprueba los so
	vim /etc/network/interfaces		<- Aadir en la priimary network interface:
			Comentar la línea que pone iface enp0s3 inet dhcp (añadir un # al principio)
			iface enp0s3 inet static
			address (la adress de tu equipo (ejecutar ifconfig -a)
			netmask (la adress de tu equipo (ejecutar ifconfig -a)
			gateway (aparece al hacer route -n)
			iface enp0s3 inet6 auto <- Añade una conexión IPv6 automáticamente
```

## Parte 6: INSTALAR POLITICA DE CONTRASEÑAS
```
49. sudo vim /etc/login.defs			<- Cambiamos aqui la politica básica de contraseñas.  
Línea 160	PASS_MAX_DAYS   99999        	-> PASS_MAX_DAYS    30
Línea 161 	PASS_MIN_DAYS   0           	-> PASS_MIN_DAYS    2 
Línea 162	PASS_WRAN_AGE   7     
50. Configurar cambios para los usuarios ya creados
	En los usuarios ya creados, estos cambios no son retroactivos:
	chage -M 30 <username/root>		<- Max days
	chage -m 2 	<username/root>		<- Min days
	chage -W 7 	<username/root>  	<- Warning days
	chage -l 	<username/root>	 	<- Permite comprobar la informacion

51.	apt install libpam-pwquality  		<- Paquete para control mas extenso de las contraseñas
	sudo vim /etc/pam.d/common-password   	<- Cambiamos la configuracion por defecto
	Línea 25	Añadimos al final de la linea "password requisite pam_pwqiality.so retry=3"
				minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root

				Minima longitud, ucredit y dcredit que contenga minimo una letra mayuscula y un digito, Maximo
				tres caracteres consecutivos iguales, que no contenga el nombre de usuario, y que se diferencie
				en 7 caracteres de la contraseña anterior. Además, forzamos los cambios al root.

	passwd 					<- Cambiamos la contraseña del usuario de acuerdo a estas normas
	sudo passwd 				<- Cambiamos la de root si no cumplía antes
52. sudo addgroup user42			<- Creamos un nuevo grupo (lo ide el subject)
	sudo adduser usuario user42		<- Añadimos el usuario al grupo 
	hostnamectl status			<- ¿Cómo se llama el host?
	hostnamectl set-hostname <otro_nombre>	<- Cambio del nombre del host

	Control de usuarios:
		getent passwd | cut -d ":" -f 1	<- Muestra todos los usuarios
		adduser nombreusuarionuevo	<- Añadiria un usuario nuevo	
		users 				<- Muestra los usuarios logeados
		usermod				<- Modifica usuarios. -l nombre de usuario, -c comentario o nombre.
		userdel -r 			<- Elimina le usuario y los archivos propios
		id -g <username> 		<- ID del grupo ppal del usuario
		less /etc/group | cut -d ":" -f 1 <- lista de todos los usuarios en el PC

		groups <nombreusuario>		<- Muestra los grupos del usuario
		groupadd <nombregrupo>		<- Añade grupo nuevo 
		groupdel <nombregrupo>		<- Elimina el grupo
		gpasswd -a nombreusuario nombregrupo	<- Añade el usuario al grupo. -d lo elimina
		gpasswd -d <username> <groupname> 	<- Borra el user del group
```

## Parte 7: CONFIGURAR UNA TAREA CRONOMETRADA
```
53. apt install cron				<- Instalaación de cron
54. systemctl enable crond			<- Activar cron al iniicio
	sudo crontab -u root -e 		<- Configuraremos el cron del root
	Debajo de la linea m h  dom mon dow   command 	   añadiremos:
	*/10 * * * * sh /path/to/script 	<- Path al script monitoring.sh que hemos creado

	sudo crontab -l 			<- Enseña la configuracion del cron
	*/1 * * * * /path/to/monitoring.sh  	<- Ejecuta la script cada 30s
	*/1 * * * * sleep 30s && /path/to/monitoring.sh <- En nuestro caso usr/local/bin
```

# Script
[Volver al índice](#born2beroot)
```
#!/bin/bash
archv=$(uname -vm) 														
#Uname da informacion del sistema. -v version kernel y -m arquitectura

CPU=$(grep "physical id" /proc/cpuinfo | uniq | wc -l) 					
#Cuento el numero de physical id (cada uno corresponde a una CPU fisica
#pueden tener varios nucleos, y varios procesadores virtuales cada uno.
#Mas proces que nucleos --> Hyperthreadimng

vCPU=$(grep "^processor" /proc/cpuinfo | wc -l) 						
#nº de procesadores = nº vcpus. grep "^ "

tRAM=$(free --mega | awk '$1 == "Mem:" {print $2}') 					
#Free muestra memoria. Awk columna 2, solo cuando columna 1 es "Mem:"

uRAM=$(free --mega | awk '$1 == "Mem:" {print $3}')
rRAM=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}') 			

tmdsk=$(df -Bm --total | awk '$1 == "total" {print $2}' | sed 's/M//')	
#df = espacio disco, -Bm = unidades a Megas

umdsk=$(df -Bm --total | awk '$1 == "total" {print $3}' | sed 's/M//')	
#sed 'sustituye/estevalor/porlanada/'

rmdsk=$(df -Bm --total | awk '$1 == "total" {printf("%.2f"), $3/$2*100}')	
rCPU=$(top -bn1 | awk '$1 == "%Cpu(s):" {printf("%.2f"), $2 +$4}') 		
#top lista los procesos en tiempo real, bn1 solo actualiza una vez
#Cojo columnas 2  y 4, que tienen cpu de usuario y de proc inactivos

lstb=$(who -b | awk '{print $3 " " $4}')								
#who -b da el last reboot. Cojo fecha, espacio, y hora.

nLVM=$(lsblk | grep "lvm" | wc -l)	
#nº de particiones que tengan el lvm. Si > 0, esta activo.

LVM=$(if [$nLVM -eq 0]; then echo No; else echo Yes; fi) 
#Estructura ifs, terminada en fi. -eq 0 es == 0. Espacio entre if y condicion importante.

ncon=$(ss -s | awk '$1 == "TCP:" {print $4}' | sed 's/,//') 			
#ss -s enseña conexiones. Me quedo con las established y quito la coma.

nuser=$(users | wc -w)	
#Enseña usuarios en misma linea, asi que cuento palabras.

ip=$(hostname -I)

mac=$(ip link show | awk '$1 == "link/ether" {print $2}') 
#Enseño ips, hay varios metodos. Cojo el ether, que es la MAC.

nsudcmd=$(journalctl _COMM=sudo | grep "COMMAND" | wc -l) 
#Puedo leer el log, y filtro los comandos sudo. De ahi me quedo con
#las lineas que realmente son ejecuciones de comando y las cuento.

wall -n "Architecture: $archv											
#Wall envia un mensaje por pantalla. -n elimina el banner inicial

#CPU Physical: $CPU
#vCPU: $vCPU
#Memory Usage: $uRAM/${tRAM}MB ($rRAM%)
#Disk Usage: %umdsk/${tmdsk}MB ($rmdsk%)
#CPU Load: $rCPU%
#Last Boot_ $lstb
#LVM Use: $LVM
#Connexions TCP: $mcpm ESTABLISHED
#User Log: $nuser
#Network: IPv4 $ip // MAC $mac
#Sudo: $nsudcmd commands executed"
```

# Bonus
[Volver al índice](#born2beroot)

## Parte 11:LIGHTTPD
Es un webserver sencillo y rapido. Lo vamos a usar como base para alojar el WORDPRESS.
```
sudo apt install lighttpd
sudo ufw allow 80 				<- Utiliza el puerto 80 por defecto. Tambien vale "sudo ufw allow https"
systemctl start lighttpd 			<- Inicializamos el programa
systemctl enable lighttpd 			<- Hacemos que se inicialice siempre al bootear
systemctl status lighttpd 			<- Check
```

## Parte 12: MARIADB
MariaDB: Sistema de gestión de bases de datos. El webserver va a acceder a esta base de datos para mostrar 
las distintas páginas web que pudiesemos crear. Esta basado en SQL.
```
sudo apt install mariadb-server
systemctl start mariadb-server #Inicializamos el programa
systemctl enable mariadb-server #Hacemos que se inicialice siempre al bootear
systemctl status mariadb-server #Check

sudo mysql_secure_installation #Configuramos la instalacion, eliminando ciertas opciones inseguras
	Switch to unix_socket authentication [Y/n]: Y			#Permite conectar con el unix socket, que maneja
									con seguridad las conexiones entre procesos.
	Enter current password for root (enter for none): Enter
	Set root password? [Y/n]: Y
	New password: Password
	Re-enter new password: Password
	Remove anonymous users? [Y/n]: Y 				#Elimina el acceso de usuarios anonimos
	Disallow root login remotely? [Y/n]: Y				
	Remove test database and access to it? [Y/n]:  Y		#Elimina la base de datos de prueba
	Reload privilege tables now? [Y/n]:  Y				#Guarda los datos

systemctl restart mariadb	#Restart para aplicar cambios
sudo mariadb 				#Entro en la consola del mariaDB para configurarlo
	CREATE DATABASE nombre; #Importante terminar en ;
	CREATE USER 'nombreusuario'@'nombrehost(termina42)' IDENTIFIED BY 'contraseña';
	GRANT ALL ON nombrebasededatos.* TO 'nombreusuario'@'nombrehost(termina42)' 
	IDENTIFIED BY 'contraseña' WITH GRANT OPTION; 		#El .* da acceso a todas las
								#tablas de la base de datos.
	FLUSH PRIVILEGES; 	#Aplico los cambios
	EXIT
	sudo mariadb -u nombreusuario -p 	#Check para ver si esta bien creado
	SHOW DATABASES;		#Check	
```

## Parte 13: PHP Y WORDPRESS
PHP: Es un lenguaje de scripts para servidores. Las funciones de Wordpress lo usan.
sudo apt install php-cgi php-mysql 	#Instalo los dos modulos basicos necesarios

Wordpress: Instalo el propio wordpress. Es un Content Manager System (CMS). Permite crear blogs y webs.
```
sudo apt install wget
sudo wget hhttp://wordpress.org/latest.tar.gz -P /var/www/html #Descargo la ultima version de Wordpress
sudo tar -xzvf /var/www/html/latest.tar.gz 	#Extraigo el .tar, lo extraigo, copio el contenido en 
sudo rm /var/www/html/latest.tar.gz		#/var/www/html y borro el tar y el original extraido
sudo cp -r wordpress/* /var/www/html					*
sudo rm -rf wordpress
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php	#Uso la config de prueba
sudo vi /var/www/html/wp-config.php		#Cambio la configuracion para linkearla con la database de MariaDB
	define( 'DB_NAME', 'database_name_here' );^M
	define( 'DB_USER', 'username_here' );^M
	define( 'DB_PASSWORD', 'password_here' );^M

Configuracion Lighttpd
sudo lighty-enable-mod fastcgi			#Activo los modulos fastcgi. Es un protocolo para conexiones de 
sudo lighty-enable-mod fastcgi-php  		#aplicaciones a servidores web
sudo service lighttpd force-reload

Ahora podemos conectarnos al servidor a traves del navegador utilizando nuestra ip.
 http://10.11.200.231
```

# Parte 2: INSTALAR UN SERVICIO EXTRA A TU ELECCIÓN: FAIL2BAN
Fail2ban: Se trata de un servicio de protección extra. Entre otras cosas, permite bloquear las ips de aquellos 
usuarios que hayan intentado conectarse de forma fallida mediante ssh tras un numero determinado de intentos.
```
sudo apt install fail2ban
systemctl start fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local		Creamos una version local de la configuracion
						Línea 285	#Cambiamos la configuracion en JAILS, despues de mode=normal añadiendo:
								Cambiamos tambien los puertos de dropbear y selinux-ssh a 4242, ya que es la 
								que usa nuestro ssh
	enabled = true
	maxretry = 3
	findtime = 10m
	bantime = 10m
	port = 4242

		systemctl restart fail2ban
		systemctl status fail2ban

		fail2ban-client status				status y las ip baneadas
		fail2ban-client status sshd
		tail -f /var/log/fail2ban.log
```

# Conceptos
[Volver al índice](#born2beroot)
- [Centos](#centos)
- [Debian](#debian)
- [Debian vs Centos](#debian-vs-centos)
- [LVM](#lvm)
- [Virtualización](#virtualización)
- [Máquina Virtual](#maquina-virtual)
- [Apt Aptitude](#apt-aptitude)
- [Apparmor](#apparmor)
- [SELinux](#selinux)
- [Apparmor vs SELinux](#apparmor-vs-selinux)
- [TTY](#tty)
- [SUDO](#sudo)
- [AppArmor](#apparmor)
- [SSH](#ssh)

## Centos
https://www.centos.org/

## Debian
https://www.debian.org/intro/why_debian

## Debian vs Centos
Debian es más sencillo de utilizar, fácilmente actualizable, paquetes más actualizados y con multitud de paquetes para personalizar
CentOS tiene mas soporte y un mercado más grande, porque una vez instalado es más fácil de usar, personalizar las actualizaciones y las configuraciones son muy complejas, y los paquetes más limitados pero es posible gestionar los recursos de una manera específica si la utilidad es específica.

## LVM
La gestión de volúmenes lógicos proporciona una vista de alto nivel sobre el almacenamiento en un ordenador, en vez de la tradicional vista de discos y particiones.
Los volúmenes de almacenamiento bajo el control de LVM pueden ser redimensionados y movidos a voluntad, aunque esto quizá necesite actualizar las herramientas del sistema.
LVM también permite la administración en grupos definidos por el usuario, permitiendo al administrador del sistema tratar con volúmenes llamados, por ejemplo, "ventas" o "desarrollo", en vez de nombres de dispositivos físicos, como "sda" o "sdb".

https://es.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)

## Virtualización
La virtualización utiliza el software para imitar las características del hardware y crear un sistema informático virtual. Esto permite a las organizaciones de TI ejecutar más de un sistema virtual, y múltiples sistemas operativos y aplicaciones, en un solo servidor1​ Por tanto este software tiene la función de simular la existencia del recurso tecnológico que se quiere virtualizar. 

https://es.wikipedia.org/wiki/Virtualizaci%C3%B3n

## Máquina Virtual
En informática, una máquina virtual es un software que simula un sistema de computación y puede ejecutar programas como si fuese una computadora real. 
Virtualización completa de un sistema operativo dentro de otro sistema operativo con la misma arquitectura
Las máquinas virtuales pueden ser emuladas, virtualizadas

## Apt Aptitude
Aptitude gestiona los paquetes y es de alto nivel (usuario "monkey" aptitude gestiona los paquetes _inteligentemente_)
Apt es una herramienta de bajo nive (el usuario teie todo el poder)

## Apparmor
AppArmor ("Application Armor") es un módulo de seguridad del kernel Linux que permite al administrador del sistema restringir las capacidades de un programa. Para definir las restricciones asocia a cada programa un perfil de seguridad. Este perfil puede ser creado manual o automáticamente. Para restringi las capacidades de un programa, existen limitaciónes de acceso a las rutas de los archivos.

https://es.wikipedia.org/wiki/AppArmor


## TTY
indica que sudo puede usarse incluso si no hay una shell/sesión interactiva. El impacto de seguridad depende de la configuración del sistema. Si el usuario de apache pudiera escribir en /etc/init. d/httpd o cambiar el comportamiento del guión de inicio cambiando algunos parámetros en /etc/defaults, un atacante podría hacer cualquier cosa.

(en esencia es una forma segura de evitar que se pueda ejecutar sudo desde una consola externa)

Hay una ventaja de seguridad muy limitada en tener requiretty en un servidor. Si se explota algún código que no sea raíz (un script PHP, por ejemplo), la opción requiretty significa que el código de explotación no podrá actualizar directamente sus privilegios ejecutando sudo.
Puede haber otra forma para que el atacante obtenga la raíz y, por supuesto, el atacante aún podrá desfigurar su sitio, pero no permitir que el atacante obtenga la raíz significa que otros servicios que se ejecutan como usuarios diferentes seguirán funcionando normalmente y el atacante ganará. No ser capaz de borrar los registros del sistema. Si ninguna de sus reglas de sudo hace nada peligroso como crear un directorio, esto no es una preocupación.
Además, y lo que es más condenatorio para los requisitos, no se necesita ningún privilegio para crear un tty, p. con esperar Por lo tanto, también podría desactivar requiretty: por sí solo, no proporciona una ventaja de seguridad. Proporciona una ligera ventaja de auditabilidad cuando lo ejecutan los usuarios (porque los registros dan una mejor idea de quién invocó sudo y de dónde provienen), pero no cuando se ejecuta desde un trabajo en segundo plano.

## SUDO
El programa Sudo (super user do, en Inglés) es una utilidad de los sistemas operativos tipo Unix, como Linux, BSD, o Mac OS X, que permite a los usuarios ejecutar programas con los privilegios de seguridad de otro usuario (normalmente el usuario root) de manera segura, convirtiéndose así temporalmente en súper usuario.
Se instala por defecto en /usr/bin.

https://es.wikipedia.org/wiki/Sudo

## SSH
SSH (o Secure SHell) es el nombre de un protocolo y del programa que lo implementa cuya principal función es el acceso remoto a un servidor por medio de un canal seguro en el que toda la información está cifrada. Además de la conexión a otros dispositivos, SSH permite copiar datos de forma segura (tanto archivos sueltos como simular sesiones FTP cifradas), gestionar y crear claves RSA para no escribir contraseñas al conectar a los dispositivos y pasar los datos de cualquier otra aplicación por un canal seguro tunelizado mediante SSH y también puede redirigir el tráfico del (Sistema de Ventanas X) para poder ejecutar programas gráficos remotamente. El puerto TCP asignado es el 22.

https://es.wikipedia.org/wiki/Secure_Shell


# Lista de ccomandos básicos
[Volver al índice](#born2beroot)
```
lsblk 									configuración disco
sudo visudo 								configuración de sudo
sudo apt install openssh-server
sudo systemctl status ssh
sudo service ssh status
service ssh restart
sudo –i 								aplica sudo bastante tiempo
ssh <tutuser>42@<tuIP> -p 4242 
sudo nano /etc/ssh/sshd_config indicar el puerto a usar + en máquina virtual
sudo nano /etc/sudoers 
sudo ufw enable
sudo ufw allow ssh
sudo ufw status numbered 
sudo ufw allow 4242
sudo ufw delete “numero del puerto”
sudo ufw status verbose 
sudo ufw default deny autgoing cierra los puertos de salida
sudo nano /etc/pam.d/common-password 
sudo nano /etc/login.defs 
chage -m 2 -M 30 root
chage –l rperez-a muestra
chage rperez-a cambia
cut -d: -f1 /etc/passwd muestra todos los usuarios
sudo adduser prueba1 
sudo userdel prueba1
sudo groupadd evaluating
sudo groupdel evaluating
sudo usermod -aG sudo your_username añadir usuario
groups rperez-a todos los grupos del usuario
getent group evaluating
getent group muestra todos los grupos
hostnamectl
hostnamectl set-hostname new_hostname + cambiar  sudo nano /etc/hosts + reboot
sudo nano /var/log/sudo/sudo.log
Sudo replay + “numero que aparece en el código ID abajo a la izquierda del log (segunda fila)”
bash /usr/local/bin/monitoring.sh
sudo crontab -u root –e + reboot
shasum sgoinfree/students/<tu_usuario>/Born2beroot.vdi > correcion.txt
```


REFERENCIAS:
	https://github.com/Sefy-j/Born2beroot
	https://github.com/caroldaniel/42sp-cursus-born2beroot
	https://github.com/hanshazairi/42-born2beroot
	https://www.youtube.com/watch?v=2w-2MX5QrQw (particiones para el Bonus)
