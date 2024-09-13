---
title: Más descriptores de archivos
date: '2023-05-22'
tags: 

---

Desde la versión 7.1, adJ soporta abrir simultáneamente más de 500.000 archivos.
Por su parte FreeBSD y OpenBSD sólo permiten abrir 32.000 archivos 
simultáneamente.

El aumento en número de descriptores de archivos puede afectar algunos
binarios (los que en sus fuentes empleen `fileno`) por lo que es indispensable 
recompilar algunos programas del sistema base (`doas`, `lex`, `make`, `perl`), 
así como algunos paquetes incluyendo:
`git`, `unzip`, `bison`, `m4`, `python`, `ruby`, `gettext-tools`,  `gmake`, 
muchos descriptores de archivos.

La limitación comienza por el campo `short _file` de la estructura `FILE` 
declarada en `/usr/include/stdio.h`.


