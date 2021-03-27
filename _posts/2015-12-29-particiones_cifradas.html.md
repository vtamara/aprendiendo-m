---
title: particiones_cifradas
date: 2015-12-29
tags:
---
El instalador/actualizador de adJ pregunta que si desea tener particiones cifradas para bases de PostgreSQL y sus respaldos, en caso de aceptar se crean los contenedores ```/var/post.img``` y ```/var/resbase.img``` que se montan con la clave que el usuario especifique en ```/var/postgresql``` y ```/var/www/resbase``` respectivamente.

* En ```/var/postgresql``` quedan bases de datos de PostgreSQL
* En ```/var/www/resbase``` quedan copias de respaldo.

Los cifrados son los contenedores, pero en su operación típica un usuario debe dar claves para estos contenedores durante el arranque y los monta en esos directorios SIN CIFRAR.  

Un atacante que por ejemplo robe un computador o robe los contenedores cifrados no podrá tener acceso a la información de los contenedores si no cuenta con la clave.  Aunque podría intentar ataques de fuerza bruta que con suficiente tiempo podrán encontrar la clave (el algoritmo blowfish usado para el cifrado se cree que le hará bien difícil esta labor).

Un atacante que logre ingresar al sistema como un usuario mientras esté operando con contenedores ya montados podrá leer la información de esos directorios de acuerdo al usuario que logre suplantar.   Si logra suplantar la cuenta root o un usuario administrador podrá ver todos los archivos del sistema, pero si suplanta a otro usuario puede disminuirse lo que podrá leer teniendo en cuenta:

# Emplear todas las medidas de seguridad de PostgreSQL: usuarios con claves y sistema de claves (por lo menos md5 y no trust como deja por defecto el instalador de adJ).
# Emplear sistema de permisos del sistema operativo en archivos de ```/var/postgresl``` (típicamente manejado bien por PostgreSQL y el instalador de adJ) y en ```/var/www/resbase``` (que es manejado por aplicaciones web y usuario).  
# Se recomienda que los archivos de ```/var/www/resbase``` tenga un dueño diferente a www:www en lo posible y que no tengan permiso de lectura para otros (sólo para dueño y si acaso grupo).
# Los respaldos se han ubicado en ```/var/www``` para que las aplicaciones web recluidas en jaula ```/var/www``` puedan escribir (por ejemplo anexos como hace SIVeL). Pero esto requiere ser extra cuidadoso con dueños, grupos y permisos en ```/var/www/resbase```, pues un atacante que logre explotar alguna falla de una aplicación web recluida en la jaula ```/var/www``` típicamente ingresará como usuario www con grupo www.
# Las aplicaciones web que no queden recluidas a la jaula /var/www por ejemplo escritas en ruby o node.js se recomienda que operen como el usuario www con grupo www. 

