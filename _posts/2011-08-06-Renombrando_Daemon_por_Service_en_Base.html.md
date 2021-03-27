---
title: Renombrando_Daemon_por_Service_en_Base
date: 2011-08-06
tags:
---
Se ha comenzado a renombrar daemon por service en el sistema base.

Se renombro la bitácora /var/log/daemon por /var/log/service, la facilidad daemon se considera obsoleta frente al nuevo nombre de facilidad service.

Este cambio se implementó en 
/etc/syslog.conf /etc/newsyslog.conf 


Para el resto del sistema base debe tenerse cuidado con:

1. Funcion daemon() de libc
2. Usuario y grupo daemon aunque mantener uid y gid ayudan


Hay comentarios por cambiar como "cursed by a Unix daemon".
En cuanto a archivos en 4.9 los únicos en /usr/src con este nombre eran:

./gnu/usr.sbin/sendmail/sendmail/daemon.c
./gnu/usr.sbin/sendmail/sendmail/daemon.h
./kerberosV/src/lib/roken/daemon.c
./lib/libc/gen/daemon.3
./lib/libc/gen/daemon.c
./usr.sbin/lpr/common_source/startdaemon.c
./xenocara/app/xdm/daemon.c
./xenocara/app/xfs/os/daemon.c

Respecto a esta cadena en medio de archivos se encuentran estos casos:


Estos puede renombrarse sin inconveniente junto con los Makefile que los compilan.
Problema con función daemon de la librería de C, implementada en /lib/libc/gen/daemon.{3,c}
pues puede ser usada por aplicaciones del sistema base (de hecho es usada por xenocara/app/xdm/daemon.c) como portes y otros programas.  Por compatibilidad mantenerse la versión obsoleta daemon() pero agregarse la función service() ---daemon() hará un llamado a service().

