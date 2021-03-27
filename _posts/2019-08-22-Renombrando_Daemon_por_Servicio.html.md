---
title: Renombrando_Daemon_por_Servicio
date: 2019-08-22
tags:
---
En el contexto de tecnología las alusiones al demonio han resultado ofensivas para algunos cristianos (por ejemplo el logo del proyecto FreeBSD ver {5}, {6} y {7}), llegando eventualmente a prohibiciones (ver {8}).  Esto también ocurre con el término ```daemon``` (o demonio) para referirse a cierto tipo de programas en ambientes tipo Unix (ver constante ```LOG_DAEMON``` en ```/usr/include/syslog.h``` y en manual de ```syslog```).  

En la distribución adJ (desde 4.7) hemos venido renombrando paulatinamente daemon por servicio en el kernel y en el sistema base.

Entonces en nuestro contexto un __servicio__ se refiere a un programa que opera en segundo plano de manera permanente desde que se inicia un servidor a la espera de conexiones por parte de programas clientes que lo requieran, son ejemplos de servicios:  correo, resolución de nombres DNS, cola de impresión.   

A continuación se presentan detalles del uso del término y de la implementación del cambio.


## Historia

El nombre daemon según {1} fue acuñado por el proyecto MAC del MIT iniciado en 1963 del cual Unix lo habría tomado, los integrantes del proyecto MAC se basarían a su vez en el demonio de Maxwell, un experimento para explicar termodinámica.  Respecto a tal descripción hemos revisado fuentes de Unix en diversas versiones mantenidas por The Unix Heritage Society y otras fuentes:

* En el libro de 1891 donde Maxwell describió su experimento, no uso la palabra demon (ver {11}) tal nombre lo encontramos referenciado por Lord Kelvin (ver {12})
* Se encuentra comentarios con este términos en las fuentes del CTSS (Compatible Time-Sharing System) disponibles en {20}, desarrollado por el proyecto MAC y demostrado en 1961, según {9} el término comenzó a usarse en el CTSS , y en este se basó Multics en el cual a su vez se basó Unix.
* En la versión 2 de Unix de 1972 se encuentra ```daemon``` como etiqueta entre los fragmentos de fuentes en ensamblador públicamente conocidos (ver en {13})
* La versión 3 de 1973 en el título de las fuentes de ensamblador de ```lpd``` dice "```lpd -- Line Printer daemon```" y así continuó ampliandose su uso en versiones posteriores para otros programas y una cuenta de usuario en la versión 5.
* Con respecto a su uso en ```syslog.h```, este archivo aparece la primera vez en 4.2BSD que es de 1983 (ver {14}), aunque esta versión no incluye la constante ```LOG_DAEMON```. Como se explica en {4} BSD fue la distribución de Berkley, que se basó en fuentes donadas de Unix, y que empezó a realizar más innovaciones que la versión V de AT&T.
* En 4.3 aparece ```/etc/syslog.conf``` con facilidad ```daemon``` y ```syslog.h``` es referenciado en fuentes de ```syslogd``` (ver {15} )
* La facilidad no era un nombre estándar en POSIX 2001 (ver {16}) pero fue introducida posteriormente y fue estándar desde la revisión de 2004 de POSIX (ver {17}).
* En las fuentes de OpenBSD rastreables está desde 1995 (ver {18}) y en las fuentes rastreables de FreeBSD desde  1994 (ver {19}). Según la página del manual de ```syslog``` de OpenBSD 4.8 provienen de 4.2BSD


## Implementación del cambio

Se ha implementado en este orden:  kernel, archivos de configuración de ```/etc```,  documentación, programas del sistema base

!1. Kernel
Desde adJ 4.7 en general se cambiaron comentarios a lo largo de todas  las fuentes.  Otros pequeños cambios fueron:

* Nombre de la llamada 153 en capa de compatibilidad de FreeBSD que no está implementada (```sys/compat/freebsd```)
* En ```dev/raidframe``` la función ```rf_GetSpareTableFromDaemon``` declarada en ```dev/raidframe/rf_kintf.h```, implementada en ```dev/raidframe```/rf_decluster.c y sólo usada en ```dev/raidframe``` se renombró por ```rf_GetSpareTableFromServicio```
* En ```nfs/nfs.h``` se cambio el símbolo ```NFS_MAXASYNCDAEMON``` por ```NFS_MAXASYNCSERVICIO``` usado solo en esta parte  del kernel. 
* En ```sys/sys/buf.h``` se renombró el símbolo ```B_PDAEMON``` por ```B_PSERVICIO```, la función ```buf_daemon``` por ```buf_servicio```, y una cadena de ```B_BITS``` la cual sólo es usado para imprimir en ```sys/kern/vfs_subr.c```

La mayoría de cambios en el kernel se concentran y conectan con ```sys/uvm/```:
* Se renombró el archivo ```pdaemon.c``` por ```pservicio.c```, así como la función ```uvm_aiodone_daemon``` por ```uvm_aiodone_servicio```
* Se renombraron las variables globales ```pagedaemon``` y ```pagedaemon_proc``` por ```pageservicio``` y ```pageservicio_proc``` (declaradas en ```sys/uvm/uvm.h```)
* Se renombraron miembros de  la estructura ```struct uvmexp``` declarada en ```sys/uvm/uvm_extern.h``` y empleada en diversos archivos de ```sys/uvm```, así como las fuentes del ejecutable ```/usr/bin/vmstat``` que las usa.
* Se renombraron varios hilos de ejecución el kernel relacionados con memoria virtual y en particular relacionados con ```sys/uvm```, por ejemplo en ```sys/kern/init_main.c```, ```kern/vfs_bio.c```


Con respecto a cambios que afectan programas fuera del kernel sólo están:
* Cambio al ejecutable ```vmstat``` que ahora en lugar de reportar ```pagedaemon``` reporta ```pageservicio```
* En ```syslog``` el nombre de la facilidad ```daemon``` y su símbolo asociado ```LOG_DAEMON``` (ver ```sys/sys/syslog.h```).  Para no quebrar aplicaciones se dejó y se aumentó ```LOG_SERVICIO``` con el mismo valor de ```LOG_DAEMON``` y se añadió el nombre de facilidad ```servicio``` asociado al mismo número de ```daemon```. 

!2. Archivos de configuración de /etc

* Desde adJ 4.9 Se renombró la bitácora ```/var/log/daemon``` por ```/var/log/service``` y se cambió su uso a lo largo del sistema base.
* Desde adJ 5.1 se renombró ```daemon``` por ```service``` en todo archivo, se trataron de forma especial:
** Servicios iniciados en ```/etc/rc.d```, en adJ emplean las variables ```servicio_user```, ```servicio_flags``` y ```servicio_class```.  Se soportan pero se consideran obsoletas las variables ```daemon_user```, ```daemon_flags``` y ```daemon_class```
** En ```login.conf``` se dejó la clase ```daemon``` para asegurar compatibilidad de programas que la empleen, pero se agregó la clase ```service``` que es la recomendada.
** Se dejó el grupo ```daemon``` en ```/etc/groups``` por compatibilidad, pero se considera obsoleto y se recomienda emplear el grupo ```servicio``` que tiene el mismo gid de ```daemon```.  Se recomienda ubicar el grupo ```servicio``` antes del grupo ```daemon``` para que se vea este grupo al listar archivos cuyo dueño tenga gid 1.
** Se mantuvo el usuario ```daemon``` en ```/etc/master.passwd``` por compatibilidad, pero se considera obsoleto. Se recomienda el nuevo usuario ```servicio``` que tiene el mismo uid del usuario ```daemon```.  Se recomienda ubicar el usuario ```servicio``` antes de ```daemon``` en ```/etc/master.passwd``` para que al listar archivos con ```ls``` se presente ```servicio``` como dueño de archivos con uid 1 (este es el resultado que hemos evidenciado aún cuando ```man master.passwd``` dice "While it is possible to have multiple entries with identical login names and/or identical user IDs, it is usually a mistake to do so.  Routines that manipulate these files will often return only one of the multiple entries, and that one by random selection.")


!3. Documentación

Desde adJ 5.2 toda página del manual donde aparecía ```daemon``` se renombró por ```servicio```.

!4. Librerías

Desde adJ 5.2
* En ```libc``` se cambió la función ```daemon``` por ```servicio``` (declarada en ```/usr/include/stdlib.h```), aunque se dejó la primera por compatibilidad se considera obsoleta.
* En ```libwrapper``` se cambió ```eval_daemon``` por ```eval_servicio``` y ```RQ_DAEMON``` por ```RQ_SERVICIO``` (declarados en ```/usr/include/tcpd.h```).  Los primeros nombres se dejaron por compatibilidad pero se consideran obsoletos.  La estructura request_info tenía un campo daemon que se cambió por servicio --según la documentación en tcpd.h los programas que los usan no deberían acceder directamente ese campo sino mediante eval_daemon (ahora eval_servicio).

!5. Directorios con programas
Desde adJ 5.2 se cambiaron las referencias en los programas de los directorios ```/bin``` ```/game``` ```gnu``` (excepto sendmail y perl) ```include``` ```kerberosIV``` ```regress``` y ```share``` 




##Referencias

* {1} http://en.wikipedia.org/wiki/Daemon_%28computer_software%29
* {2} http://cm.bell-labs.com/cm/cs/who/dmr/hist.html
* {3} Twenty Years of Berkeley Unix. From AT&T-Owned to Freely Redistributable. Marshall Kirk McKusick. http://oreilly.com/catalog/opensources/book/kirkmck.html 
* {4} 
* {5} http://www.politechbot.com/2005/04/01/religious-leaders-warn/
* {6} http://seclists.org/politech/2005/Apr/6
* {7} http://fe.pasosdejesus.org/?id=Mascota+FreeBSD
* {8} http://www.guardian.co.uk/science/the-lay-scientist/2010/nov/15/3
* {9} http://en.wikipedia.org/wiki/Compatible_Time-Sharing_System
* {10} http://people.csail.mit.edu/psz/LCS-75/Brochure.html
* {11} http://openlibrary.org/books/OL7243600M/Theory_of_heat
* {12} http://zapatopi.net/kelvin/papers/kinetic_theory.html#fn1
* {13} http://minnie.tuhs.org/Archive/PDP-11/Distributions/research/1972_stuff/s1-bits.gz
* {14} http://minnie.tuhs.org/cgi-bin/utree.pl?file=4.2BSD/usr/include/syslog.h
* {15} http://minnie.tuhs.org/cgi-bin/utree.pl?file=4.3BSD/usr/src/etc/syslogd.c
* {16} http://www.opengroup.org/onlinepubs/009695399/functions/syslog.html
* {17} http://www.opengroup.org/onlinepubs/009695399/basedefs/syslog.h.html
* {18} http://www.openbsd.org/cgi-bin/cvsweb/src/sys/sys/syslog.h?rev=1.1;content-type=text%2Fplain 
* {19} http://www.freebsd.org/cgi/cvsweb.cgi/src/sys/sys/syslog.h?rev=1.1;content-type=text%2Fplain 
* {20} http://www.piercefuller.com/data/ctss-listings.zip?id=ctss
