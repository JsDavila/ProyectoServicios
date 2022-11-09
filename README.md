# ProyectoServicios
Monitoreo Nagios

##Pasos para la instalación de nagios


Antes de la instalación de los paquetes o plugins de la herramienta primero debemos ser SELINUX en la ruta /etc/sysconfig/selinux. En el archivo /etc/sysconfig/selinux se cambió la opción de “SELINUX=enforcing” a “SELINUX=disabled”

##Instalación de paquetes

yum install –y httpd php gcc glibc glibc-common gd gd-devel make netsnmp net-snmp-utils

Descarga de Nagios core y plugins




wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios3.5.0.tar.gz 
wget https://www.nagios-plugins.org/download/nagios-plugins-1.5.tar.gz



##Creación de usuarios y grupo


useradd nagios
groupadd nagcmd
usermod –a –G nagcmd nagios
usermod –a –G nagcmd apache



Extraemos los paquetes descargados e ingresamos al directorio “nagios” que se creó, una vez dentro del directorio debemos compilar e instalar Nagios.


tar –zxvf nagios-3.5.0.tar.gz # tar –zxvf nagios-plugins-1.5.tar.gz
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode
make install-webconf # cp –r contrib/eventhandlers/ /usr/local/nagios/libexec
/usr/local/nagios/bin/nagios –v /usr/local/nagios/etc/nagios.cfg




Iniciamos procesos de apache, nagios, snmp y creamos el usuario para ingresar a la interfaz gráfica

service httpd start
service nagios start
service snmpd start
htpasswd –c /usr/local/nagios/etc/htpasswd.users nagiosadmin


Luego entramos al directorio de Nagios-plugins-1.5 y procedemos con su instalación


service httpd start
service nagios start
service snmpd start
htpasswd –c /usr/local/nagios/etc/htpasswd.users nagiosadmin


## Configuraciones finales para poder ingresar al modo gráfico


chkconfig --add nagios
chkconfig nagios on
chkconfig --add httpd
chkconfig httpd on
chkconfig snmpd on


Para poder tener estadísticas de todos los equipos a monitorear se hace un cambio en el archivo Nagios.cfg donde cambiamos la opción “log_initial_states” con el valor de 1


Y reiniciamos el servicio


service nagios restart


## Instalación y configuración complemento NSClient++


Edite su archivo de configuración de Nagios principal (main).
vi /usr/local/nagios/etc/nagios.cfg


Quite el carácter asterisco (#) de la siguiente línea de su archivo de configuración principal:

cfg_file=/usr/local/nagios/etc/objects/windows.cfg


**Descargar e instalar el nscp.exe en la máquina de Windows**

Dentro del directorio en donde se instaló la herramienta editar el archivo NSClient.ini descomentando todos los módulos que aparecen listados en la sección [modules], excepto CheckWMI.dll y RemoteConfiguration.dll.
Asignar ip y contraseña al archivo NSCclient:


allowed_host= **IP HOST**
password=nagios


Abrir y editar el archivo windows.cfg en el Centos


vi /usr/local/nagios/etc/objects/windows.cfg



## Agregar la definición nueva de equipo para la máquina Windows que se desea monitorear



define host
{
use windows-server;
host_name winserver;
alias servidores windows;
address 10.253.70.1;
}


Definir los servicios que se desea monitorear. Ejemplo:


define service
{
use generic-service
host_name winserver
service_description CPU Load
check_command check_nt!CPULOAD!-l 5,80,90
}

Abrir y editar el archivo commands.cfg en el Centos para incluir el argumento "-s <PASSWORD>" (donde PASSWORD es la contraseña que se especificó en la máquina Windows) así:

define command
{
command_name check_nt command_line $USER1$/check_nt –H $HOSTADDRESS$ -p 12489 -s pasa.123 -v $ARG1$ $ARG



