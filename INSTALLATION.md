## 1. Actualitza el sistema
sudo apt update && sudo apt upgrade -y
## 2. Instalar Apache
sudo apt install apache2 -y
Activa i inicia el servei:

sudo systemctl enable apache2
sudo systemctl start apache2
Verifica l'estat:

sudo systemctl status apache2
Visite http://localhostla página del defecto de Apache.

## 3. Instalar MySQL
Ubuntu 24.04 incluye el paquete mysql-serverdel repositorio oficial (versión 8.0 o superior):

sudo apt install mysql-server mysql-client -y
Inicia i habilita el servei:

sudo systemctl enable mysql
sudo systemctl start mysql
Configuración de MySQL:

Accés a la consola de MySQL
sudo mysql
Creació de la base de dades
CREATE DATABASE bbdd;
Creació de l'usuari local
CREATE USER 'usuario'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
GRANT ALL PRIVILEGES ON bbdd.* TO 'usuario'@'localhost';
FLUSH PRIVILEGES;
EXIT;

## 4. Instalar PHP y extensiones comunes
Ubuntu 24.04 incluye PHP 8.3 y el repositorio estándar:

sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-json php-cli -y
Reinicia Apache por cargar PHP:

sudo systemctl restart apache2
Verifica la versió de PHP:

php -v
Crea un fitxer de prova:

echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php

# Configuración de VirtualHost con Apache2

## Introducción
Un servidor web con Apache2 permite asignar diversos lugares web de forma independiente en una mateixa máquina. Esta funcionalidad está asociada con VirtualHosts (amfitrions virtuales). Cada VirtualHost define un lugar web único con su propio nombre de dominio, director de negocio y configuración específica.

Una vez configurado, podrá acceder a su sitio web desde un navegador utilizando el nombre de dominio que tiene definido (por ejemplo, www.domini.local).

## 1. Creación de la estructura de directores
Para organizar sus servidores web, es recomendable emmagatzemar-los dins del directori per defecto de Apache: /var/www/.

Para nuestro ejemplo, cree un directorio para el dominio domini.local:

sudo mkdir -p /var/www/domini.local
Nota: Para poder emmagatzemar los instaladores en cualquier ubicación, seguir esta convención facilitará la gestión y el mantenimiento del servidor.

## 2. Definición del VirtualHost
Cree un ajuste de configuración para su VirtualHost en el directorio /etc/apache2/sites-available/:

sudo nano /etc/apache2/sites-available/domini.local.conf
Afegiu-hi la configuració següent (sustituïu domini.localpel tu nom de domini):

<VirtualHost *:80>
    ServerAdmin admin@domini.local
    ServerName www.domini.local
    ServerAlias domini.local
    DocumentRoot /var/www/domini.local
    ErrorLog ${APACHE_LOG_DIR}/domini.local_error.log
    CustomLog ${APACHE_LOG_DIR}/domini.local_access.log combined
</VirtualHost>
Recomendación: Utilice ajustes de registro separados para cada VirtualHost para facilitar la limpieza.

## 3. Habilitar el VirtualHost
Apache2 només carga los VirtualHosts que están habilitados . Para fer-ho, use la orden a2ensite:

sudo a2ensite domini.local.conf
Esta comando crea un enllaç simbòlic des de /etc/apache2/sites-available/cap a /etc/apache2/sites-enabled/.

Nota: No cal canviar de directori abans d'executar a2ensite; funciona des de cualquier ubicación.

## 4. Reiniciar Apache2
Después de modificar la configuración, cal reinicie el servicio para aplicar los lienzos:

sudo systemctl restart apache2
Alternativa: sudo service apache2 restart (funciona, pero systemctles el estándar moderno en sistemas basados ​​en systemd).

## 5. Modificar /etc/hostsper resoldre el domini localment
Para que su sistema resuelva el nombre de dominio www.domini.localde su máquina, editeu el fitxer /etc/hosts:

sudo nano /etc/hosts
Afegiu la línia següent:

127.0.0.1   www.domini.local domini.local
Esto permite que el navegador encuentre su sitio web en el sentido de que necesita un servidor DNS externo.

## 6. Comprovar el funcionament
Abrí un navegador y accedí a:

http://www.domini.local
Si el directorio /var/www/domini.localestá instalado, Apache puede mostrar un error 403 o una lista de directorios (según la configuración). Para probar que funciona, crea un ajuste de prueba:

echo "<h1>Hola, benvingut domini.local</h1>" | sudo tee /var/www/domini.local/index.html
Torneu a cargar la página i hauríeu de veure el missatge.

## 7. Solución de problemas: Registros de Apache2
Si el bloqueo no funciona con la esperanza, consulte los registros de Apache:

Registro de errores
Conté missatges sobre errores de configuración, permisos, fitxers no trobats, etc.

sudo tail -f /var/log/apache2/domini.local_error.log
Registro de acceso
Mostra totes les peticions rebudes pel servidor.

sudo tail -f /var/log/apache2/domini.local_access.log
Consell: Useu tail -fper veure les entrades en temps real mentre proveu el lloc.

## 8. Assignació de permisos
Apache2 se ejecuta normalmente con el usuario www-data. Para evitar problemas de permisos, configure el propietario y los permisos del director de su lugar:

Canviar el propietari
Permita que su usuario pugui edite fitxers y que Apache els pugui llegir:

sudo chown -R $USER:www-data /var/www/domini.local
Establir permisos adequats
Asegúrese de que el propietario y el grupo tinguin accedan completamente, y que otros usuarios només puguin llegir:

sudo chmod -R 775 /var/www/domini.local

# Guía de instalación y configuración de plataformas en la nube (Nextcloud / ownCloud)

Dins d'un host virtual preconfigurado ( /var/www/domini.local)

Esta guía explica cómo instalar Nextcloud o ownCloud en un entorno en el que hay un host virtual activo apuntando a /var/www/domini.local(por ejemplo, domini.local). No cal configurar Apache ni el host virtual, ya que se considera operativo.

## 1. Descàrrega e instalación de la plataforma nube
### 1.1. Enllaços oficials
Nextcloud : https://www.nextcloud.com
Descàrrega directa:
https://download.nextcloud.com/server/releases/latest.zip

ownCloud : https://www.owncloud.org
Descàrrega directa (versión estable):
https://download.owncloud.com/server/stable/owncloud-complete-20240724.zip

Nota : Nextcloud es compatible con PHP 8.1+, mientras que ownCloud incluye PHP 7.4 y muchas versiones estables. Asegúrese de tener la versión de PHP adecuada después de instalarla.

### 1.2. Passos d'instal·lació
Mou't al directori del host virtual :

cd /var/www/domini.local
Neteja el contingut actual (si cal):

Assegura't que no hi ha dades importants abans d'ejecutar això.

sudo rm -rf *
Descarga el instalador.zip de la plataforma triada (Nextcloud o ownCloud) al tu sistema.

wget https://download.nextcloud.com/server/releases/latest.zip
Descomprime el archivo directamente al director :

Heu descarregat l'arxiu a una ruta qualsevol
sudo unzip /ruta/al/arxiu.zip
Si el archivo crea una carpeta interna (ej: nextcloud/o owncloud/), asegúrese de que el contingut es mogui al nivell arrel del host virtual:

sudo mv nextcloud/* . && sudo rmdir nextcloud
 o
sudo mv owncloud/* . && sudo rmdir owncloud
Podeu fer això directamente si ho teniu descomprimit a Descargas:
cp -R ~/Descargas/nextcloud/. /var/www/domini.local/.
Elimineu la carpeta nextcloudi l'arxiulatest.zip

sudo rm -rf ~/Descargas/nextcloud && sudo rm -rf ~/Descargas/latest.zip
Podeu fer això directamente si ho teniu descomprimit a /var/www/domini.local:
cp -R /var/www/domini.local/nextcloud/. /var/www/domini.local/.
Elimineu la carpeta nextcloudi l'arxiulatest.zip

sudo rm -rf /var/www/domini.local/nextcloud && sudo rm -rf /var/www/domini.local/latest.zip
Asegúrese de que los permisos sean correctos :

sudo chown -R www-data:www-data /var/www/domini.local
sudo chmod -R 755 /var/www/domini.local
Acceda a la interfaz web : Abra el navegador y visite:

http://domini.local
Siga las instrucciones de configuración asistida:

Crea un usuari administrador.
Configura la base de datos (recomendada: MariaDB/MySQL).
Verifica que todos los requisitos del sistema estén completos.
2. Recomendaciones adicionales
Directori de dades : Durant la instal·lació, es recomana no emmagatzemar les dades dins del directori web (ej: /var/www/domini.local/data). Millor usa una ruta externa com /var/ncdatao /opt/owncloud-data.
Copias de seguridad : Fes backups habituales del directori de dades i de la base de dades.
Seguridad : Desactiva el acceso a dispositivos inteligentes ( .htaccess, config.php) y considera cambiar las reglas de seguridad adicionales de Apache o Nginx.
Apéndice: Instalación de PHP 7.4 en Ubuntu 24.04 (nombres para ownCloud)
Esto no es necesario si instala ownCloud , ya que muchas versiones estables no son compatibles con PHP 8.3 (versión defectuosa en Ubuntu 24.04). Nextcloud no requiere esto .

Pasos:
Actualitza el sistema :

sudo apt update && sudo apt upgrade -y
Instalar las dependencias para afegir repositorios PPA :

sudo apt install software-properties-common -y
Afegeix el repositorio de PHP de Ondřej Surý :

LC_ALL=C.UTF-8 sudo add-apt-repository ppa:ondrej/php -y
Actualitza els repositoris :

sudo apt update
Instale PHP 7.4 y las extensiones requeridas :

sudo apt install -y php7.4 \
    libapache2-mod-php7.4 \
    php7.4-fpm \
    php7.4-common \
    php7.4-mbstring \
    php7.4-xmlrpc \
    php7.4-soap \
    php7.4-gd \
    php7.4-xml \
    php7.4-intl \
    php7.4-mysql \
    php7.4-cli \
    php7.4-ldap \
    php7.4-zip \
    php7.4-curl
(Opcional) Selecciona PHP 7.4 con una versión por defecto :

sudo update-alternatives --config php
Activa los módulos de Apache necesarios :

sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php7.4-fpm
Reinicia Apache :

sudo systemctl restart apache2
Verificación : Puedes comprobar la versión activa de PHP con:

php -v
