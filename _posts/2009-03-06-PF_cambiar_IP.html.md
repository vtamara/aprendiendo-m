---
title: PF_cambiar_IP
date: 2009-03-06
tags:
---
PREGUNTA FRECUENTE: ¿Cómo cambio la configuración de una tarjeta de red?

# Identifique la interfaz de red que corresponde a la tarjeta (con ```ifconfig```)
# Configure IP y mascara de red de la interfaz (archivos ```/etc/hostname.interfaz```).
# Configure compuerta de enlace (archivo ```/etc/mygate```)
# Reinicie servicio de red (con ```sudo sh /etc/netstart```)

##Explicación breve

http://structio.sourceforge.net/guias/servidor_OpenBSD/lan-ethernet.html


##Explicación extra detallada


!1. IDENTIFICAR INTERFAZ DE RED

Vea las interfaces reconocidas con
<pre>
ifconfig
</pre>
en el siguiente ejemplo se lista el resultado en un computador con 4 interfaces de red (tiene 2 tarjetas ethernet):
<pre>
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 33160
        priority: 0
        groups: lo
        inet 127.0.0.1 netmask 0xff000000
        inet6 ::1 prefixlen 128
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x4
fxp0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:08:c7:b2:b9:fd
        priority: 0
        groups: egress
        media: Ethernet autoselect (100baseTX full-duplex)
        status: active
        inet 192.168.177.30 netmask 0xffffff00 broadcast 192.168.177.255
        inet6 fe80::208:c7ff:feb2:b9fd%fxp0 prefixlen 64 scopeid 0x1
nfe0: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:1d:92:0b:9e:8b
        priority: 0
        media: Ethernet autoselect (none)
        status: no carrier
enc0: flags=0<> mtu 1536
        priority: 0
</pre>

* Note que a la izquierda están las identificaciones de las 4 interfaces de red: ```lo0=, ```fxp0```, ```nfe0``` y ```enc0```.  A continuación de cada identificación está la configuración de cada
una, por ejemplo la configuración de ```fxp0``` es
<pre>
fxp0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:08:c7:b2:b9:fd
        priority: 0
        groups: egress
        media: Ethernet autoselect (100baseTX full-duplex)
        status: active
        inet 192.168.177.30 netmask 0xffffff00 broadcast 192.168.177.255
        inet6 fe80::208:c7ff:feb2:b9fd%fxp0 prefixlen 64 scopeid 0x1
</pre>
* En una configuración se ven opciones (flags), dirección MAC (la que inicia
con lladdr), medio físico (inicia con media), estado (status) que puede
ser activo (active) o sin señal (no carrier), configuración IP versión 4 (comienza con inet y consta de IP, máscara de red y dirección de envio amplio) y configuración de IP versión 6 (comienza con inet6).

* Las interfaz ```lo0``` siempre está presente y representa el mismo computador (loopback), su dirección IPv4 siempre es 127.0.0.1.
* La interfaz ```enc0``` se emplea sólo para tráfico encriptado IPsec.

Por las indicaciones anteriores se concluye  en el ejemplo que las interfaces
correspondientes a tarjetas ethernet son ```fxp0``` y ```nfe0```, además
```nfe0``` es una tarjeta desconectada (status: no carrier), mientras que ```fxp0``` está conectada y tiene configurada la IP 192.168.177.30 con puerta de enlace en hexadecimal 0xffffff00 (en notación IP 255.255.255.0) y dirección de envio amplio 192.168.177.255.


!2. CONFIGURAR IP, MÁSCARA DE RED Y COMPUERTA DE ENLACE


La IP y la mascara de enlace se configuran en un archivo que depende de la identificación de la interfaz de red, está ubicado en el directorio ```/etc/``` y se llama ```hostname```.interfaz, siendo interfaz la identifcación de la interfaz de red (como se explico en sección anterior).

Retomando el ejemplo de la sección anterior si quisieramos cambiar la configuración de la interfaz ```fxp0``` editariamos el archivo 
