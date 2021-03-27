---
title: Uso_de_partici_n_encriptada
date: 2009-04-30
tags:
---
La partición encriptada de adJ permite manejar información que allí se almacene mientras haya arrancado el computador dando la llave de encripción correcta.  Por esto, un atacante que por ejemplo hurte el disco duro, pero que no conozca la llave de encripción, no podrá descifrar los archivos allí almacenados.

Este artículo describe el uso de la partición encriptada  tanto local como remotamente.

##1. INTRODUCCIÓN

En el momento de la instalación de adJ, puede elegir mantener la información de PostgreSQL en una imagen encriptada.   Al hacerlo el programa de instalación creará 2 imágenes encriptadas con claves que se le solicitará y las montará en los directorios ```/var/postgresql``` y ```/var/www/resbase``` .

La primera (i.e ```/var/postgresql```) mantendrá las bases de datos del motor PostgreSQL, mientras que la segunda (i.e ```/var/www/resbase```) quedará preparada para mantener copias de respaldo de las bases de datos y un "archivo digital".


##2. ARCHIVO DIGITAL

El directorio ```/var/www/resbase/``` incluirá un directorio ```AD``` que se ha planeado para organizar cronológicamente fuentes de información.    Basta ubicar la información en ese directorio para que quede en la imagen encriptada y contar con las facilidades de respaldo de adJ.  

Sugerimos que el nombre de cada archivo en este directorio comience con el año, mes y día seguido de un consecutivo. Se sugiere que el archivo tenga un formato estándar y abierto (e.g texto plano, HTML o formato de OpenOffice)

Puede ubicar un documento bien operando localmente el servidor adJ o bien remotamente enviando el archivo.


##3. USO DE LA PARTICIÓN ENCRIPTADA LOCALMENTE

Basta que almacene la fuente de información en ```/var/www/resbase/AD```  . La forma de ubicarlo en ese directorio depende del programa que esté usando para guardar la información por ejemplo:

* Con Mozilla  Firefox (por ejemplo si encuentra una noticia en Internet) y OpenOffice (por ejemplo si redacta un testimonio) emplee guardar como, examine el sistema de archivos hasta que ubique el directorio raíz, después ingrese a ```var```, de allí ingrese a ```www```, de allí ingrese a ```resbase``` y de allí ingrese a ```AD```.  Una vez en ese directorio provea el nombre del archivo siguiendo las convenciones descritas.
* Con el administrador de archivos ```xfe``` (por ejemplo si va a copiar un documento de una memoria USB después de montarla) ubique el archivo que desea copiar (o mejor mover) sobre este oprima el botón derecho del ratón y elija ```Copiar``` (o mejor ```Cortar```) en el panel izquierdo de directorios ubique el directorio raíz ```/```, a continuación dentro de este ```var``` dentro de este a su vez ```www```, dentro de este ```resbase``` y dentro de este ```AD```; una vez en el Archivo Digital presione el botón derecho sobre la parte blanca del panel derecho (que debe mostrar el contenido del Archivo Digital) y elija la opción ```Pegar```.  A continuación puede renombrarlo ubicando el cursor sobre el nuevo archivo y presione el botón derecho para elegir "Renombrar".
* Desde el interprete de comandos podría copiar una fuente de información del 2.Abr.2009 que está en archivo llamado ```caso.odt``` del directorio ```/mnt/usb``` al directorio ```/var/www/resbase/``` con: 
<pre>
cp /mnt/usb/caso.odt /var/www/resbase/20090402-01.odt
</pre>


##4. USO DE LA PARTICIÓN ENCRIPTADA REMOTAMENTE

Para emplear remotamente la partición encriptada debe tener su información en un archivo (en un formato y con un nombre como los descritos en la sección 2), y debe enviarlo al servidor con un programa que soporte el protocolo ssh {3} (por ejemplo FileZilla o scp).  Para poder usar/configurar el programa de transferencia debe disponer de:
* Un nombre de máquina o una dirección IP del servidor adJ
* Una cuenta en el servidor adJ y por ende una clave
* Que su cuenta tenga permiso para escribir en el directorio ```/var/www/resbase/AD```

Por ejemplo con FileZilla puede configurar un sitio (Archivos->Administrador de Sitios) con la IP/nombre del servidor, puerto 22, tipo de servidor SFTP tipo de ingreso Cuenta, su nombre de cuenta y clave.  
[http://aprendiendo.pasosdeJesus.org/archivos/conf-filezilla.png]

Una vez configurado puede iniciar una sesión con el sitio configurado, cuando logre conectarse en el  panel del computador que está empleando (por defecto el del lado izquierdo) ubique el archivo que enviará y en el panel del servidor (por defecto el del lado derecho) ubique el directorio ```/var/www/resbase/AD``` y transmita el archivo arrastrándolo del panel del computador fuente al panel del servidor (si lo requiere también podría renombrar el archivo después de transmitirlo al servidor).

Empleando scp desde una línea de comandos en un sistema tipo Unix como Linux u OpenBSD bastaría un comando similar al caso local:
<pre>
scp /mnt/usb/caso.odt jpabon@192.168.177.13:/var/www/resbase/20090402-01.odt
</pre>
el cual la primera vez que se conecte a ese servido le pedirá confirmar llave y después le solicitará su clave ---aunque puede configurarse para que no solicite clave.


##4. REFERENCIAS

* {1} Transferencia de información a y desde el servidor. http://structio.sourceforge.net/guias/basico_OpenBSD/transferir.html
* {2} telnet y ssh. http://structio.sourceforge.net/guias/AA_Linux_colegio/servicios-de-la-intranet.html#telnet-y-ssh
* {3} The Secure Shell (SSH) Protocol Architecture. RFC 4251. IETF. http://www.ietf.org/rfc/rfc4251.txt

-----

Dedicado al Creador que nos guia y pastorea. 
