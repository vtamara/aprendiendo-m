---
title: Actualizacion_3_8
date: 2013-11-01
tags:
---
Actualización a Aprendiendo De Jesus 3.8r4
******************************************

Los pasos de este archivo pueden omitirse si actualiza con el script
inst-sivel.sh


Detalles finales de actualización al sistema base
-------------------------------------------------

Basado en http://openbsd.org/faq/upgrade38.html

useradd -u86 -g=uid -c"HostAP Daemon" -d/var/empty -s/sbin/nologin _hostapd
cd /tmp
ftp $PKG_PATH/../etc38.tgz
tar xzpf etc38.tgz
cd /tmp/etc
cp hostapd.conf netstart pf.os rc services /etc
cp mtree/* /etc/mtree/


Compare diferencias y mezcle manualmente algunos archivos:
diff ftpusers /etc/ftpusers;
diff inetd.conf /etc/inetd.conf;
diff login.conf /etc/login.conf;
diff rc.conf /etc/rc.conf;
diff sysctl.conf /etc/sysctl.conf;
diff syslog.conf /etc/syslog.conf;
diff mail/aliases /etc/mail/aliases;

Después complete árbol de directorios:
mtree -qdef /etc/mtree/4.4BSD.dist -p / -u

Note que para que ciertos paquetes funcionen (e.g docbook-xsl) es necesario
aumentar los límites de memoria listados en /etc/login.conf, en particular la
clase default debe tener:

        :datasize-max=512M:\
        :datasize-cur=512M:\

y la clase staff:
        
	:datasize-cur=512M:\


Actualización de sendmail
````````````````````````````````````=

Si su sendmail tiene una configuración especial que requirió
recompilar, actualice fuentes y recompile:

cd /us/src
cvs -z3 update -Pd -rOPENBSD_3_8
cd gnu/usr.sbin
make clean
make
make install
cd cf/cf
make openbsd-proto-local.cf 


Actualización de paquetes
````````````````````````````````````=

Nueva forma de actualizar paquetes con los flags -u y -r de
pkg_add, que permite por ejemplo actualizar casi automáticamente todos
los paquetes instalados.  

Con algunos paquetes puede requerir precauciones adicionales antes de
actualizar, por ejemplo con PostgreSQL es indispensable hacer primero 
un volcado:
# su - _postgresql
$ pg_dumpall -h /var/www/tmp > /var/postgresql/po37.dump

Detenga el servidor (siguiendo instrucciones de /etc/rc.shutdown):
# su -l _postgresql -c "/usr/local/bin/pg_ctl stop -m fast \
    -D /var/postgresql/data"
Mueva temporalmente /var/postgresql por ejemplo:
# mv /var/postgresql /var/pos37
Después actualice los paquetes (postgresql-server y postgresql-client) --puede
ser junto con los demás paquetes del sistema como se presenta más
adelente-- y posteriormente recupere la configuración editando
/var/postgresql/data/postgresql.conf, tras comparar con la
configuración anterior (posiblemente deba cambiar unix_socket_directory):
# diff /var/postgresql/data/postgresql.conf /var/pos37/data/postgresql.conf
inicie el motor de la nueva versión siguiendo las instrucciones que
debe tener en /etc/rc.local, posiblemente:
# su -l _postgresql -c "/usr/local/bin/pg_ctl start \
        -D /var/postgresql/data -l /var/postgresql/logfile \
        -o '-D /var/postgresql/data'"
y finalmente recupere el volcado con:
$ su -l _postgresql -c "psql -h /var/www/tmp -f /var/pos37/po37.dump template1"


Para actualizar todos los paquetes instalados:

sudo su -
export PKG_PATH=ftp://uvirtual.ean.edu.co/pub/OpenBSD/3.8/packages/i386/
cd /var/db/pkg/
pkg_add -u * > /tmp/x
grep pkg_add /tmp/x | \
	sed -e "s/.*pkg_add/pkg_add -F installed -F update -F updatedepends/g" > /tmp/y  

Ejecutelo con: 
. /tmp/y  

Si el proceso se detiene antes de cocluir, edite ese archivo para
borrar los paquetes ya instalados, sálvelo y ejecutelo.

Después revise en /tmp/x otros paquetes que eventualmente no fueron
actualizados

Posteriormente reconfigure los paquetes que lo requieran, por ejemplo 
si actualiza de PHP4 a PHP5, elimine de /var/www/conf/httpd.conf la línea
LoadModule php4_module ...
y verifique diferencias entre los archivos de configuración (eventualmente
también tendrá que activar nuevas extensiones) :
diff /var/www/conf/php.ini /usr/local/share/examples/php5/php.ini-recommended
