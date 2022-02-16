# Born2beRoot
Virtualiza un sistema operativo sin interfaz gráfica (Debian o CentOS) y realiza auna configuración del mismo inclluyendo una conexión SSH segura.
Bonus: Configura una sistema funcional de worpress con las funcionalidades lighttpd, MariaDB, PHP y una extra de tu elección que excluyendo Apache 2.

## Indice
- [Instalación](#instalación)
- [Script](#scritp)
- [Bonus](#bonus)
- [Conceptos](#conceptos)
- [Lista de comandos básicos](#lista-de-comandos-básicos)

# Instalación
[Volver al índice](#born2beroot)

## Parte 0: CREACIÓN DE UNA MAQUINA VIRTUAL
0.	Crear carpeta en sgoinfree/nuestrousuario
1.	Descargar iso Debian de la página web propia (misma configuración de arquitectura que nuestro ordenador 64 bits en nuestro caso
2.	Virtual Box --> Nueva --> Memoria(Default) --> Disco Duro VDI (30.8Gb) #Guardo en sgoinfree
3. Configuracion-->Red-->Adaptador puente 					Para conexiones posteriores del bonus
										(En caso de que no vayasmos a hacer el bonus lo dejaremos tal y como está NAT)
4. Configuración-->Pantalla-->VBoxVGA #Evita un bug al iniciar la máquina
5. Seleccionamos la .iso como disco de arranque

PARTE 2: INSTALACION DEBIAN Y PARTICIONES

	- Install
	- Idioma, localizacion, y idioma de teclado (locale configuration)
	- Hostname: "login42", usuario: "login", y contraseña y contraseña del root
	- Zona horaria
	- Particiones: Manual
		Elegimos el disco duro como lugar donde hacer las particiones, con el tamaño completo
		Particion primaria (physical):
			Tamaño segun subject, y al principio del espacio disponible montandolo en /boot 
				#Va a ser el sda1, la particion fisica donde vamos a instalar el arranque del sistema
		Partición lógica (Logical Volume):
			Seleccionamos el resto de la partición y la seleccionamos como volumen físico para encriptacion
				#Nos piden que el volumen logico este encriptado
			Configurar volumenes encriptados-->Crear-->Seleccionamos el volumen a encriptar
			Elegimos contraseña de encriptacion
			Particiones del LVM:
				- Creamos Logical Group en la particion logica. Nombre: LVMGroup 
				- Creamos los Logical Volumes uno a uno, del tamaño correspondiente 
					#(var-log solo necesita un guion en el nombre)
				- Montamos los volumenes de acuerdo al subject, con sistema archivos Ext4
					#Aqui seleccionamos para que va a ser cada uno (/root, /home, etc). El var log
					#debemos montarlo escribiendolo manualmente (/var/log)
					#Además, el swap usa tipo de archivos especial, swap area
	- Seleccionamos el mirror desde donde descargara los paquetes el apt
	- Deseleccionamos los paquetes extra ESPECIALMENTE LA INTERFAZ GRAFICA

## Parte 1 LVM
1.	Comprobar, el procesador del ordnador, la arquitectura debe ser la misma para poder virtualizar correctamente el sistema que elijamos (Debian 64 en este caso)
2.	Crear la crear la máquina virtual en Virtual box
3.	Crear 1 partición para el /boot (arranque del sistema operativo)
4.	El resto del espacio reservado, ccrearemos una partición selecciónando la opción de do not mount it
5.	Creamos un grupo volumenes encriptados (LVM)
6.	Añadimos las particiones, según el tamaño del subject (considerar que virtual box gestióna la reserva de memoria en 1024 Bits)

##  Parte 2 LOGIN
8.	Usar password de encriptación
9.	Login: Preferible root para el setup
10.	lsblk									<- 	Comprobamos que las particiones son correctas
11.	aa-status								<- 	Comprobamos el estado de AppArmor. Deberia estar instalada y funcional por defecto

## Parte 3 SUDO
14.	apt install sudo						<- 	Debemos estar en root. Si no: "su -" --> "root password"
15.	dpkg -l | grep sudo						<- 	Comprobamos que este instalado correctamente
	systemctl status sudo
16. adduser <usuario> sudo					<- 	Añadimos el usuario al grupo sudo
14. getent group sudo						<- 	Comprobacion usuario
15.	sudo reboot								<- 	Reiniciar sudo
16.	sudo -v									<- 	Comprobación sudopowers

Configuración sudo: 
	/etc/sudoers							a) 	archivo de configuración inicial
	/etc/sudoers.d/<otras_config>			b) 	posibilidad de crear nuevos archivos de configuración de sudo
												(El archivo sin ~ ni)

17. sudo vim /etc/sudoers.d/sudo-conf		a) 	Modo de editar con el programa de la elección específico
	sudo -e /etc/sudoers.d/sudo-conf		b) 	Modo de editar (da a alegir entre los programas instalados)

	Defaults passwd_tries=3					<- 	Maximo numero de intentos para acceder a sudo
	Defaults badpass_message="error"		<- 	Mensaje de error personalizado
	Defaults log_input,log_output 			<- 	Guardado de los imputs y los okutputs
	Defaults iolog_dir="/var/log/sudo" 		<- 	Lugar de guardado de los logs (hay que crear el directorio manualmente)
	Defaults requiretty 					<- 	Obligamos a que solo se pueda usar desde terminal.
												Ningún proograma podrá usar sudo si no es desdde consola
												Ver conceptos [TTY](#tty)
	Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

18. sudo mkdir /var/log/sudo				<-	Creamos la carpeta y el archivo donde vamos a guardar los logs

## Parte 3: INSTALAR UFW: Uncomplicated Firewall. Es un firewall facil de configurar. Monitoriza los puertos

19.	apt install ufw
20.	ufw enable
21. systemctl enable ufw					<-	Activamos que se lance el ufw en cada inicio
22.	ufw status								<-	Muestra los puertos Activamos
23.	ufw allow 4242 #Permite el puerto 4242, que segun el subject es el unico debido
24.	ufw delete numeroderegla	#Elimina la regla numerada

## Parte 4 INSTALAR SSH
con autenticación y encriptacion de datos

	apt install openssh-server
	systemctl enable sshd
	nano /etc/ssh/sshd_config 
		Port 22 --> Port 4242 #Cambiamos el puerto por defecto que usa el SSH por el 4242
		PermiteRootLogin prohibit-password --> PermitRootLogin no #Elimina la opcion de conectarse como root

Desde consola externa, comprobamos la conexión remota:
	ssh user@serverIP -p 4242 #Conectamos con el usuario a la direccion ip y en el puerto 4242
	(ifconfg -a para comprobar la direccion ip)

## Parte 5: INSTALAR POLITICA DE CONTRASEÑAS

sudo nano /etc/login.defs		#Cambiamos aqui la politica básica de contraseñas.  
		PASS_MAX_DAYS    99999 -> PASS_MAX_DAYS    30		#PASS_WARN_AGE ya esta por defecto en 7
 		PASS_MIN_DAYS    0     -> PASS_MIN_DAYS    2 
En los usuarios ya creados, estos cambios no son retroactivos:
		chage -M 30 <username/root>					Max days
		chage -m 2 <username/root>	#Min days
		chage -W 7 <username/root>  #Warning days
		chage -l <username/root>	#Permite comprobar la informacion
	apt install libpam-pwquality  #Paquete para control mas extenso de las contraseñas
	sudo nano /etc/pam.d/common-password   #Cambiamos la configuracion por defecto
		Añadimos al final de la linea "password requisite pam_pwqiality.so retry=3"
				minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
				#Minima longitud, ucredit y dcredit que contenga minimo una letra mayuscula y un digito, Maximo
				#tres caracteres consecutivos iguales, que no contenga el nombre de usuario, y que se diferencie
				#en 7 caracteres de la contraseña anterior. Además, forzamos los cambios al root.
	passwd #Cambiamos la contraseña del usuario de acuerdo a estas normas
	sudo passwd #Cambiamos la de root
	sudo addgroup user42 #Creamos un nuevo grupo tal y como nos piden
	sudo adduser usuario user42 #Añadimos el usuario al grupo
	Control de usuarios:
		getent passwd | cut -d ":" -f 1 #Muestra todos los usuarios
		adduser nombreusuarionuevo #Añadiria un usuario nuevo	
		users #Muestra los usuarios logeados
		usermod #Modifica usuarios. -l nombre de usuario, -c comentario o nombre.
		userdel -r #Elimina le usuario y los archivos propios
		less /etc/group | cut -d ":" -f 1 - show list of all users on computer;

		groups nombreusuario #Muestra los grupos del usuario
		groupadd nombregrupo #Añade grupo nuevo 
		groupdel nombregrupo #Elimina el grupo
		gpasswd -a nombreusuario nombregrupo #Añade el usuario al grupo. -d lo elimina
		gpasswd -d <username> <groupname> - removes user from group;

## Parte 6: CONFIGURAR UNA TAREA CRONOMETRADA

Usaremos cron

	- sudo crontab -u root -e #Configuraremos el cron del root
		Debajo de la linea m h  dom mon dow   command añadiremos:
		*/10 * * * * sh /path/to/script #Path al script monitoring.sh que hemos creado
	- sudo crontab -l #Enseña la configuracion del cron

	*/1 * * * * /path/to/monitoring.sh              #Ejecuta la script cada 30s
| 		*/1 * * * * sleep 30s && /path/to/monitoring.sh

sleep $(bc <<< $(who -b | cut -d ":" -f 2)%10*60)

# Script
[Volver al índice](#born2beroot)

https://github.com/caroldaniel/42sp-cursus-born2beroot/blob/master/script/monitoring.sh

#!/bin/bash
archv=$(uname -vm) #Uname da informacion del sistema. -v version kernel y -m arquitectura
CPU=$(grep "physical id" /proc/cpuinfo | uniq | wc -l) #Cuento el numero de physical id distintos que aparecen en la info
														 #de procesadores, cada uno corresponde a una CPU fisica.
														 #Estas pueden tener varios nucleos, y varios procesadores virtuales cada uno.
														 #Mas proces que nucleos --> Hyperthreadimng
vCPU=$(grep "^processor" /proc/cpuinfo | wc -l) #Cuento el numero de procesadores, es decir, el numero de vcpus. grep "^ " empieza por esto
tRAM=$(free --mega | awk '$1 == "Mem:" {print $2}') #Free enseña la memoria. Awk columna 2, solo cuando columna 1 es "Mem:"
uRAM=$(free --mega | awk '$1 == "Mem:" {print $3}')
rRAM=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}') #No tengo que especificar la unidad, porque voy a dividir
																 #Utilizo printf para la precision del porcentaje (decimales)
																 #%f floating, .2 es precision de dos números.
tmdsk=$(df -Bm --total | awk '$1 == "total" {print $2}' | sed 's/M//')	 	#df enseña espacio disco, -Bm selecciona unidades a Megas
umdsk=$(df -Bm --total | awk '$1 == "total" {print $3}' | sed 's/M//')		#sed 'sustituye/estevalor/porlanada/'
rmdsk=$(df -Bm --total | awk '$1 == "total" {printf("%.2f"), $3/$2*100}')	
rCPU=$(top -bn1 | awk '$1 == "%Cpu(s):" {printf("%.2f"), $2 +$4}') #top lista los procesos en tiempo real, bn1 solo actualiza una vez
																   #Cojo las col2 y 4, que tienen cpu de usuario y de proc inactivos
lstb=$(who -b | awk '{print $3 " " $4}')	#who -b da el last reboot. Cojo fecha, espacio, y hora.
nLVM=$(lsblk | grep "lvm" | wc -l)	#Cuento el numero de particiones que tengan el lvm. Si es mayor que cero, esta activo.
LVM=$(if [$nLVM -eq 0]; then echo No; else echo Yes; fi) #Estructura ifs, terminada en fi. -eq 0 es == 0. Espacio entre if y condicion importante.
ncon=$(ss -s | awk '$1 == "TCP:" {print $4}' | sed 's/,//') #ss -s enseña conexiones. Me quedo con las established y quito la coma.
nuser=$(users | wc -w)	#Enseña usuarios en misma linea, asi que cuento palabras.
ip=$(hostname -I)
mac=$(ip link show | awk '$1 == "link/ether" {print $2}') #Enseño ips, hay varios metodos. Cojo el ether, que es la MAC.
nsudcmd=$(journalctl _COMM=sudo | grep "COMMAND" | wc -l) #Puedo leer el log, y filtro los comandos sudo. De ahi me quedo con
														  #las lineas que realmente son ejecuciones de comando y las cuento.
wall -n "Architecture: $archv			#Wall envia un mensaje por pantalla. -n elimina el banner inicial
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


# Bonus
[Volver al índice](#born2beroot)

INSTALACION DE SERVICIOS BONUS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PARTE 1: WORDPRESS CON LIGHTTPD, MariaDB y PHP

	- Lighttpd: Es un webserver sencillo y rapido. Lo vamos a usar como base para alojar el WORDPRESS.
		sudo apt install lighttpd
		sudo ufw allow 80 	#Utiliza el puerto 80 por defecto. Tambien vale "sudo ufw allow https"
		systemctl start lighttpd #Inicializamos el programa
		systemctl enable lighttpd #Hacemos que se inicialice siempre al bootear
		systemctl status lighttpd #Check

	- MariaDB: Sistema de gestión de bases de datos. El webserver va a acceder a esta base de datos para mostrar 
	las distintas páginas web que pudiesemos crear. Esta basado en SQL.
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
			Remove anonymous users? [Y/n]: Y 						#Elimina el acceso de usuarios anonimos
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

	- PHP: Es un lenguaje de scripts para servidores. Las funciones de Wordpress lo usan.
		sudo apt install php-cgi php-mysql 	#Instalo los dos modulos basicos necesarios

	- Wordpress: Instalo el propio wordpress. Es un Content Manager System (CMS). Permite crear blogs y webs.
		sudo apt install wget
		sudo wget hhttp://wordpress.org/latest.tar.gz -P /var/www/html #Descargo la ultima version de Wordpress
		sudo tar -xzvf /var/www/html/latest.tar.gz 	#Extraigo el .tar, lo extraigo, copio el contenido en 
		sudo rm /var/www/html/latest.tar.gz			#/var/www/html y borro el tar y el original extraido
		sudo cp -r wordpress/* /var/www/html
		sudo rm -rf wordpress
		sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php	#Uso la config de prueba
		sudo vi /var/www/html/wp-config.php		#Cambio la configuracion para linkearla con la database de MariaDB
			define( 'DB_NAME', 'database_name_here' );^M
			define( 'DB_USER', 'username_here' );^M
			define( 'DB_PASSWORD', 'password_here' );^M

	-Configuracion Lighttpd
		sudo lighty-enable-mod fastcgi		#Activo los modulos fastcgi. Es un protocolo para conexiones de 
		sudo lighty-enable-mod fastcgi-php  #aplicaciones a servidores web
		sudo service lighttpd force-reload

	Ahora podemos conectarnos al servidor a traves del navegador utilizando nuestra ip.
		http://10.11.200.231

----------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------

PARTE 2: INSTALAR UN SERVICIO EXTRA A TU ELECCIÓN: FAIL2BAN

	- Fail2ban: Se trata de un servicio de protección extra. Entre otras cosas, permite bloquear las ips de aquellos 
	usuarios que hayan intentado conectarse de forma fallida mediante ssh tras un numero determinado de intentos.
		sudo apt install fail2ban
		systemctl start fail2ban
		cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local		#Creamos una version local de la configuracion
			#Cambiamos la configuracion en JAILS, despues de mode=normal añadiendo:
				enabled = true
				maxretry = 3
				findtime = 10m
				bantime = 10m
				port = 4242
			#Cambiamos tambien los puertos de dropbear y selinux-ssh a 4242, ya que es la que usa nuestro ssh.
			#Hay que cambiar los puertos tambien en el jail.conf
		systemctl restart fail2ban
		systemctl status fail2ban

		fail2ban-client status				#Checkeo de los status y las ip baneadas
		fail2ban-client status sshd
		tail -f /var/log/fail2ban.log

----------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------


# Conceptos
[Volver al índice](#born2beroot)

DEBIAN VS CENTOS

	Debian es más sencillo de utilizar, fácilmente actualizable y con multitud de paquetes para personalizar
	CentOS tiene mas soporte y un mercado más grande, porque una vez instalado es más fácil de usar, personalizar
	las actualizaciones y las configuraciones son muy complejas, y los paquetes más limitados.

LOGICAL VOLUME MANAGER: LVM

	Es un sistema de manejo de la memoria del disco duro usado en Linux. Los "volumen físico", es decir,
	el hardware, se divide en particiones primarias y en particiones lógicas. Las particiones primarias 
	son fijas, y es donde se almacena la parte de arranque del sistema operativo, mientras que
	las particiones lógicas, los "volumenes lógicos", se pueden subdividir en distintas particiones que 
	pueden formar parte de distintos volumenes físicos, y que pueden incluso ser realocados. Estan agrupados
	en "grupos de volumenes", que es una colección con nombre de volumenes lógicos. Solo puede haber una
	partición lógica por ordenador, y necesita de una pequeña partición física que hace las veces de puntero.

GESTIÓN DE PAQUETES: APT-GET, APT Y APTITUDE

	La herramienta base de gestión de paquetes de linux es el DPKG. Sin embargo, es una herramienta de
	bajo nivel, y requiere de instalación de las dependencias de forma manual. Por ello, se utiliza,
	originalmente apt-get, y más actualmente apt como herramienta de instalación de paquetes.
	Por su parte, Aptitude es una aplicación alternativa a Apt con interfaz gráfica, que permite una
	gestión más intuitiva de las configuraciones posibles en las instalaciones.

APPARMOR vs SELinux

	AppArmor es el programa por defecto de Debian de control del MAC (Mandatory Acces Control), un protocolo 
	de seguridad que evita que los programas puedan afectar a elementos que previamente no tenian derecho
	de hacerlo, aislandolas unas de otras. SELinux en cambio es mucho más complejo, pero también permite
	mayor control y opciones de configuración.

CAMBIO DE HOSTNAME


COMANDOS ÚTILES




## Virtualización
## LVM
Logical Volume Manager is a system of mapping and managing hard disk memory used on Linux-kernel's based systems. Instead of the old method of partitioning disks on a single filesystem, and having it be limited to only 4 partitions, LVM allows you to work with "Logical Volumes", a more dinamically and flexible way to deal with your hardware.

There are three major concepts you must understand to fully grasp the behaviour of LVM:

Volume Group: It is a named collection of physical and logical volumes. Typical systems only need one Volume Group to contain all of the physical and logical volumes on the system;
Physical Volumes: They correspond to disks; they are block devices that provide the space to store logical volumes;
Logical Volumes: They correspond to partitions: they hold a filesystem. Unlike partitions though, logical volumes get names rather than numbers, they can span across multiple disks, and do not have to be physically contiguous.
The idea sounds simple enough: You take a disk, declare it as a Physical Volume, then you take that volume and append it to the Volume Group of your choice (usually only one per computer). At last, you may "partition" that volume into small Logical Volumes that can correspond to 1 or multiple disks, and can be reallocated in memory even if they are already in use.

LVM is a great utility to have on servers and systems that demand usage stability and great necessity of quick management of the available physical devices (it makes it way easier to add or remove memory, for instance).

## Araquitectura de computadores
## Linux kernell

## Máquinas virtuales
Una máquina virtual es...
### Virtualzación
### Emulacción

## TTY
## SUDO
## AppArmor
## SSH
 Secure Socket Shell. Protocolo de red que permite un acceso seguro a un sistema,
con autenticación y encriptacion de datos

# Lista de ccomandos básico
[Volver al índice](#born2beroot)

lsblk                                     1 <- Comprobar particines
sudo aa-status                            2 <- AppArmor estado
getent group sudo                         3 <- Usuarios en el grupo sudo
getent group user42                       4 <- user42 group users
sudo service ssh status                   5 <- ssh status, yep
sudo ufw status                           6 <- ufw status
ssh username@ipadress -p 4242             7 <- connect to VM from your host (physical) machine via SSH
nano /etc/sudoers.d/<filename>            8 <- yes, sudo config file. You can $ ls /etc/sudoers.d first
nano /etc/login.defs                      9 <- password expire policy
nano /etc/pam.d/common-password          10 <- password policy
sudo crontab -l                          11 <- cron schedule

apt-get install aptitude            <- install aptitude

aptitude install apparmor           <- install apparmor
aa-status 
systemctl enable apparmor

aptitude install ufw                <- installar UFW  
ufw enable
ufw status verbose                *    comprobar port status
ufw default allow/deny incoming   *    Modificar puertos de entrada y salid
ufw default allow/deny outgoing
ufw allow/deny <port-number>
systemctl enable ufw              *   Activar Firewal al arrancar la máquina
ufw disable                       *   Deshabilitar Firewall
 
 
	1) lsblk                              #Comprueba particiones
	2) sudo aa-status                     #AppArmor
	3) getent group sudo                  #Usuarios del grupo sudo
	4) getent group user42                #Usuarios del grupo user42
	5) sudo service ssh status            #SSH
	6) sudo ufw status                    #UFW
	7) ssh username@ipadress -p 4242      #Conectar a la VM desde la terminal
	8) nano /etc/sudoers.d/<filename>     #Configuración del sudo
	9) nano /etc/login.defs               #Política de expiracion de contraseñas
	10) nano /etc/pam.d/common-password   #Política de contraseñas
	11) sudo crontab -l       			  #Tareas programadas con cron
	12)sudo /var/log/sudo				  #Logs del sudo
	13)
 
 
 La misma mierda con el ssh que no me funciona nunca...
 /etc/ssh/sshd_config              *     Ruta Path de la configuraciónd e ssh
 
 
 systemctl start mariadb (no mariíadb-server) no funciona....
 
 
 puede que no lo tengo aquí
 sudo vi /var/www/html/wp-config.php

	hostnamectl status
	hostnamectl set-hostname <new-hostname> #Deberia cambiar el hostname. PUEDE NO FUNCIONAR!
	hostname (new -hostname)
	sudo nano /ect/hostname #Cambiamos aqui el hostname
	sudo nano /etc/hosts	#Cambiamos aqui el hostname



por orden me quedan estas guías
https://github.com/caroldaniel/42sp-cursus-born2beroot/blob/master/guides/Debian-en.md

https://baigal.medium.com/born2beroot-e6e26dfb50ac (la general con otras nociones)

Bonus
https://github.com/caroldaniel/42sp-cursus-born2beroot/blob/master/guides/Bonus-en.md

To do:
[ ] Guías
[ ] Checkear la correción
[ ] Añadir una funcionalidad extra en el wordpress??
[ ] Tengo los conceptos terminados???


info extra de administración de sistemas
https://root.cern.ch/root/htmldoc/guides/users-guide/ROOTUsersGuide.html


REFERENCIAS:

	https://github.com/Sefy-j/42Madrid/tree/master/Born2beroot
	https://github.com/caroldaniel/42sp-cursus-born2beroot
	https://github.com/hanshazairi/42-born2beroot
	https://www.youtube.com/watch?v=2w-2MX5QrQw
