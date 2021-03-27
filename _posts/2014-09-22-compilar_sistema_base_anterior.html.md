---
title: compilar_sistema_base_anterior
date: 2014-09-22
tags:
---
No es fácil devolverse a una versión anterior del sistema a partir de las fuentes, sin embargo nos ha servido:


# Compilar librerias basicas para la version anterior, quedarán en /usr/lib pero no serán usadas por los binarios 
# Compilar y poner kernel anterior. Reiniciar con ese
# Remplazar librerías mínimas de /usr/lib por versiones anteriorees compiladas en paso 1  libc.so, libpthread y libiberty.   (por ejemplo cp /usr/lib/libc.so.77.0 /usr/lib/libc.so.77.2
# Compilar e instalar todo base con make -j4 build (seguirá referenciando versiones nuevas de libc)
# Reiniciar pero en modo single borrar librerías con version reciente libc y libpthread (libiberty no puede borrarse aun), al continuar ld.so se verá forzado a usar versión con numeración un poco anterior pero que corresponde a la compilada en paso 1. Borrar otras librerías recientes (e.g libressl, libsqlite3, libcrypt).
# Volver a compilar todo base, el encadenador tendrá que usar las librerías disponibles.  
# Reiniciar, ya deben usarse librerías compiladas en punto 1. 
# Editar /usr/bin/{as,ld,ar,ranlib}  y modificar versión de libiberty para que corresponda a la que queda.
# Borrar libiberty con numeración reciente.
# Desde ```/usr/src/gnu/usr.bin/``` ejecutar ```make -j4 clean all install```
# Compilar xenocara. En 5.6
## make build
## Falla una librería por falta de un archivo que borra, pero esta en repositorio CVS. Después de cvs -z3 co, compilar esa
## Compilar todas las librerias cd ..; make -j4
## En ocasiones sirve pasar al directorio que falla, dar ```gmake distclean``` y después ```make -f Makefile.bsd-wrapper```
## Compilar todo: cd ..; make -j4
## Para compiladr kdrive borrar antes xserver/config.status y compilar desde directorio kdrive
## Tras compilar kdrive continuar cd ..; make -j4



