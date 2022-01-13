---
categories:
- documentación
tags: []
title: Semaforos
excerpt_separator: ''

---
Un semáforo previene el uso simultaneo de un recurso compartido por varios hilos de ejecución.

# Un ejemplo de su utilidad

Por ejemplo imagine que debe coordinar el saldo de una cuenta bancaria de la cual podría retirarse desde varios cajeros automáticos (cada uno en un hilo de ejecución).   Es necesario un mecanismo (como un semáforo) que asegure que se hagan de manera atómica las siguientes 3 operaciones: 1) examinar el saldo, 2) si hay suficiente permitir el retiro, 3) descontar lo retirado del saldo.

Sin un mecanismo de este estilo dos atacantes podrían coordinarse para intentar sacar más de lo que hay en una cuenta.  Por ejemplo si el saldo en una cuenta es 100 y dos atacantes se coordinan para sacar 80 exactamente en el mismo momento, podría ocurrir que para ambos diera que hay suficiente saldo y que permitiera los dos retiros.

 Simulemos la situación sin semaforo y con semáforo:

# Caso PostgreSQL

En el caso de PostgreSQL tiene un semaforo para cada conexión.

con una base de datos las escrituras por parte de dos usuarios 

Por su parte PostgreSQL implementa PGSemaphore de diversas maneras en cada plataforma, en OpenBSD/adJ con el API de POSIX. La implementación de PGSemaphore localiza un máximo de semaforos usables.

# Implementación de semáforos en OpenBSD/adJ

Hay 2 grandes APIs para usar semáforos en OpenBSD

1. System V: `semctl`, `semget`, `semop`
2. POSIX: `sem_init`, `sem_wait`, `sem_post` y además `sem_timedwait`, `sem_trywait`. Otras para semáforos con nombre: `sem_open`, `sem_close`, `sem_destroy` y `sem_unlink`,

Según sys/sem.h

* kern.seminfo.semmni es número de identificadores de semáforos
* kern.seminfo.semmns es número de semáforos en el sistema
* kern.seminfo.semmnu es número de estructuras para deshacer en el sistema
* kern.seminfo.semmsl es máximo número de semáforos por id
* kern.seminfo.semopm es máximo de operaciones por llamada a semop
* kern.seminfo.semum es máximo número de entradas para deshacer por proceso
* kern.seminfo.semusz es tamaño en bytes de la estructura para deshacer
* kern.seminfo.semvmx es máximo valor del semáforo
* kern.seminfo.semaem es máximo valor para ajustar a la salida
* kern.seminfo.maxid es el número válido de sysctls de semáforos

Repasando kern/sysv_sem.c