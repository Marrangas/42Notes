# Born2beRoot
Index
- [LVM setup](#LVM setup)
- Instalation
- Scripting
- Bonus
- Concepts
- List of important comands
</br>
# LVM setup
[Back to index](#Born2beRoot)</br>
</br>
# Instalation
[Back to index](# Born2beRoot)</br>
</br>
# Scripting
[Back to index](# Born2beRoot)</br>
</br>
# Bonus
[Back to index](# Born2beRoot)</br>
</br>
# Concepts
[Back to index](# Born2beRoot)</br>
#List of important comands

-Qué es el kernell
-Arquitectura de computadores
-Qué es Linux
-Qué es la virtualzación


lsblk                               1 <- Check partitions
sudo aa-status                      2 <- AppArmor status
getent group sudo                   3 <- sudo group users
getent group user42                 4 <- user42 group users
sudo service ssh status             5 <- ssh status, yep
sudo ufw status                     6 <- ufw status
ssh username@ipadress -p 4242       7 <- connect to VM from your host (physical) machine via SSH
nano /etc/sudoers.d/<filename>      8 <- yes, sudo config file. You can $ ls /etc/sudoers.d first
nano /etc/login.defs                9 <- password expire policy
nano /etc/pam.d/common-password    10 <- password policy
sudo crontab -l                    11 <- cron schedule




$ apt-get install aptitude            <- install aptitude

$ aptitude install apparmor           <- install apparmor
$ aa-status 
$ systemctl enable apparmor

$ aptitude install ufw                <- installar UFW  
$ ufw enable
$ ufw status verbose                *    comprobar port status
# ufw default allow/deny incoming   *    Modificar puertos de entrada y salid
# ufw default allow/deny outgoing
# ufw allow/deny <port-number>
# systemctl enable ufw              *   Activar Firewal al arrancar la máquina
# ufw disable                       *   Deshabilitar Firewall
 
 
 
 
 La misma mierda con el ssh que no me funciona nunca...
 /etc/ssh/sshd_config              *     Ruta Path de la configuraciónd e ssh
 
 
 systemctl start mariadb (no mariíadb-server) no funciona....
 
 
 puede que no lo tengo aquí
 sudo vi /var/www/html/wp-config.php
 
 
