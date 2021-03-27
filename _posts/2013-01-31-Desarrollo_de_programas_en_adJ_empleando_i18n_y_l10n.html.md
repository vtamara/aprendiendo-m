---
title: Desarrollo_de_programas_en_adJ_empleando_i18n_y_l10n
date: 2013-01-31
tags:
---
Tanto C99 (ver {2}) como POSIX 2008 (ver {3}) tienen en cuenta el locale y ofrecen un API aumentado y rediseñado.

! ```wchar``` - Caracteres amplios

Si quiere ver la dificultad de definir lo que es un caracter: http://techpubs.sgi.com/library/tpl/cgi-bin/getdoc.cgi?coll=fw&db=info&fname=/usr/freeware/info_tpl/gcc/gcc_3.html

Pensado para almacenar caracteres que se representan típicamente en más de un byte.  El tipo básico es wchar_t. Cada caracter de este tipo tiene un número asociado de tipo wint_t. En adJ tanto wchar_t como wint_t son enteros de 32 bits.

Para diferenciar hablaremos del conjunto de caracteres de las fuentes y del conjunto de caracteres de ejecución.  wchar permite representar internamente en cualquier codificación.

Afortunadamente la implementación del preprocesador cpp limitó las fuentes de un programa en C a estar codificadas en ASCII o UTF-8 (ver
http://linux.web.cern.ch/linux/scientific4/docs/rhel-cpp-en-4/implementation-details.html),   los caracteres en tiempo de ejecución de los programas compilados no están limitados de esta manera.
(ver historia de caracteres amplios en http://en.wikipedia.org/wiki/Wide_character).  


En C pueden emplearse caracteres y cadenas literales representadas en caracteres amplios agregando el prefijo L.  Por ejemplo_
* L'a' 
* L"esta es una cadena de wchar_t"

Los caracteres literales con el gcc de adJ se traducen en tiempo de ejecución a enteros de 32 bits (como wchar_t).  Esto se revela cuando ejecuta:
* printf("Tamaño de 'a': %i\n", sizeof('a'));
Que le dará 4

Sin embargo el tipo char continúa siendo de un byte, como puede ver si ejecuta
* printf("Tamaño del tipo char: %i\n", sizeof(char));

Por este motivo tendrá problemas si las fuentes emplean codificación UTF-8 y realiza una asignación como:
* char n='ñ'
que almacenarán solo 8 de los 32 bits del literal.   


Hay varias funciones de entrada/salida que operan con caracteres amplios para realizar entrada y salda:
 
* putwchar() envia un caracter amplio a salida estándar, supone codificación de acuerdo a locale.
* getwchar() espera de entrada estándar un carácter, supone codificación de acuerdo al locale.
* getwc(f) y fgetwc(f) leen un caracter del archivo f
* putwc(f,c) y fputwc(f, c) escriben un caracter al archivo f
* ungetwc(f) Retorna un caracter al buffer del archivo f para que sea leido la próxima vez por getwc
* fwide(f, m) establece modo m de un archivo orientado a byte (m ``` 0) o orientado a caracteres amplios (m>0).  Sin embargo con gcc en adJ no hay diferencia.



! Caracteres multibyte

Sin acudir a wchar_t, puede intentar:
* char *n = "ñ"
Que de estar codificado en UTF-8 mostrara un sizeof de 3 (2 bytes para la ñ y uno para el \0.)

Este ejemplo corresponde a un caracter multibyte.   Y hay toda una serie de funciones para operar con estos :
Por ejemplo 
mblen() da la cantidad de bytes con los que se representa un caracter multi-byte en el locale de ejecución.
mbtowc(aw, mb, 3) que convierte el primer caracter multi-byte de la cadena mb (examinando máximo 3) a un caracter amplio y lo almacena en aw.
wctomb(mb, wc)
s = wctomb(mb, w) que convierte el carater amplio w a caracter multibyte que almacena en el arreglo mb y retorna el número de bytes que escribió (por lo que puede valer la pena seguirlo de mb[s] = '\0', especialmente para imprimir con %s con printf en una consola que soporte el mismo locale de la conversión).

Tenga en cuenta que así asigna más caracteres, estas funciones sólo tendrán en cuenta el primer caracter multibyte. 
Por esto mblen("ñoño") sera el mismo mblen("ñ")

Estas funciones acuden al locale para determinar la codificación.

Estas funciones emplean un estado interno, que depende del locale, por lo que al cambiar el locale no garantizan su ejecución.  Hay versiones análogas pero que recibien el estado: mbrlen, mbrtowc, wcrtomb


Se definen especificadores de formato para printf como "%ls" y "%lc" para imprimir cadenas y caracterew wchar_t.  Tenga cuidado con estos, si los aplica a caracteres de otros tipos su programa puede terminar abruptamente sin mayor explicación.


!Cadenas multibyte

Se tratan de cadenas terminadas con nulo \0 pero con caracteres multibyte. Por ejemplo
char *cmb = "ñoño";

Hay funciones análogas al caso con caracteres multibyte:

*  size_t
     mbstowcs(wchar_t * restrict pwcs, const char * restrict s, size_t n);
Convierte la cadena multibyte s a cadena de caracteres amplios pwcs escribiendo a los sumo n bytes en pwcs. Siempre inicializa estado y no afecta el estado de mbtowc.
*  size_t
     wcstombs(char * restrict s, const wchar_t * restrict pwcs, size_t n);

Estas funciones tiene versiones análogas que permiten pasar el estado como parámetro: wcsrtombs y mbsrtowcs.  

Hay además análogas para detener procesamiento tras un máximo de bytes en la cadena de entrada: wcsnrtombs y mbsnrtowcs



!Tipo de caracteres

Un locale además de definir la codificación referida (UTF-8, ISO8859-1), define varias categorias, entre las que están LC_TYPE que determina el tipo de los caracteres.  El tipo puede ser: alnum alpha blank cntrl digit graph
lower print punct space upper xdigit

Puede establecerse el local





En cambio la siguiente si funcionará:
* wchar_t n='ñ'

La codificación interna de n debido a las restricciones de cpp, será en UTF-8. Podría convertir a otra codificación multi-bye con 

La anterior asignación más bien hag


En adJ un caracter literal (e.g 'a') por defecto se representa en 32 bits, mientras que las cadenas (e.g "a") se representan con caracteres de 8 bits.  El siguiente programa lo revela:

<pre>
#include <stdio.h>
#include <wchar.h>

int
main() {
        printf("Tamaño de wchar: %i\n", sizeof(wchar_t));
        printf("Tamaño de char: %i\n", sizeof(char));
        printf("Tamaño de L'a': %i\n", sizeof(L'a'));
        printf("Tamaño de 'a': %i\n", sizeof('a'));
        printf("Tamaño de (char)'a': %i\n", sizeof((char)'a'));
        printf("Tamaño de \"a\": %i\n", sizeof("a"));
        printf("Tamaño de L\"a\": %i\n", sizeof(L"a"));

        return 0;
}
</pre>
Que da como salida:
<pre>
Tamaño de wchar: 4
Tamaño de char: 1
Tamaño de L'a': 4
Tamaño de 'a': 4
Tamaño de (char)'a': 1
Tamaño de "a": 2
Tamaño de L"a": 8
</pre>

##BIBLIOGRAFÍA

* {0} Biblia
* {2} ISO/IEC 9899:TC2 www.open-std.org/jtc1/sc22/wg14/www/docs/n1124.pdf
* {3} POSIX 2008. http://pubs.opengroup.org/onlinepubs/9699919799/
