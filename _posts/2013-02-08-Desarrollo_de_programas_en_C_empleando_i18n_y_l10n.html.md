---
title: Desarrollo_de_programas_en_C_empleando_i18n_y_l10n
date: 2013-02-08
tags:
---
ATENCION: DOCUMENTO EN CONSTRUCCIÓN

Tanto C99 (ver {2}) como POSIX 2008 (ver {3}) tienen en cuenta el locale y ofrecen un API aumentado y rediseñado.

! Codificación de las fuentes

Para diferenciar hablaremos del conjunto de caracteres de las fuentes y del conjunto de caracteres de ejecución.  Ambos incluyen el conjunto de caracteres básicos, el conjunto de caracteres extendido depende de la codificación que emplee al guardar las fuentes o bien en el locale cuando lo ejecute (ver [i18n]).

Es importante que utilice un editor de texto que le permita guardar archivos con diferentes codificaciones, al menos LATIN1 (o ISO-8859-1) y UTF-8, ```vim``` es buena opción (ver [i18n]), pues con ```:set fenc=latin1``` guardará en LATIN1 y con  ```:set fenc=utf8``` guardará en UTF-8 (el programa ```recode``` también ayuda a convertir archivos de una codificación a otra por ejemplo ```recode latin1..utf8 fuente.c``` convierte ```fuente.c``` de LATIN1 a UTF-8).   

Recomendamos que prefiera codificación UTF-8 para sus fuentes y ambientes de ejecución, pero con cuidado, pues el siguiente programa resalta el efecto de la codificación de las fuentes:
<pre>
#include <stdio.h>
char n='ñ';
int main() {
    printf("ñ: %i, %i\n",(int)n, (int)(unsigned char)n);
    return 0;
}
</pre>


| Cod. Fuente| Cod. Ejecutable | Ejecución con es_CO.ISO8859-1  | Ejecución con es_CO.UTF-8 |
| LATIN1 | LATIN1 | ñ: -15, 241 |  -15, 241 |
| UTF-8 | UTF-8 | ñ: -79, 177 | ñ: -79, 177 |

* La ejecución en ambiente es_CO.ISO88591-1 la logra iniciando una terminal con ```LANG=es_CO.ISO8859-1 xterm -en ISO8859-1```
* La ejecución en ambiente es_CO.UTF-8 la logra iniciando una terminal con ```LANG=es_CO.UTF-8 xterm -en UTF-8```

En secciones posteriores se aclarará la diferencia por lo pronto note los resultados y tenga en cuenta que en codificación LATIN1 la ñ se representa con el byte 241, mientras que en UTF-8 (que es codificación multibyte) se representa con los dos bytes 195,177 (que equivalen al entero 50097). 


! ```unsigned char``` y ```char```

Permiten representar por lo menos el conjunto de caracteres básicos, que incluye los digitos, las letras del alfabeto inglés en mayúsculas y minúsculas y los signos de puntuación y control requeridos para programar en C. En el caso de adJ se representan en un byte de 8 bits codificados en ASCII (los 7 primeros bits) y ```char``` es equivalente a ```signed char``` (por eso el programa de la primera sección presenta -15 o -79).


! Caracteres multibyte

Un caracter multibyte es una secuencia de uno o más bytes que representa un miembro del conjunto extendido de carácteres bien en el ambiente de ejecución o bien el ambiente de fuentes (ver {2}).

En UTF-8 el juego de caracteres ASCII (7 bits) se mantiene intacto por lo que puede representarse en un byte, pero otros caracteres requieren más de un byte, como se dijo en la primera sección la ñ requiere los bytes 195 y 177, de forma análoga otros caracteres requiren 3, 4 y hasta 6 bytes (ver {4}). En el caso de adJ se soporta UTF-8 tanto en fuentes como en ejecutables.  Así que si se codifica en UTF-8 un programa con:
<pre>
printf("%i", sizeof("ñ"));
</pre>
imprimirá un valor de 3 (2 bytes para la ñ y uno para el \0.)

En la librería de C encontrará estas funciones para operar con caracteres multibyte:
* ```mblen(mb, n)``` da la cantidad de bytes con los que se representa el primer caracter multibyte de ```mb``` examinando máximo los primeros n bytes y opera en el locale de ejecución.
* ```mbtowc(wc, mb, 3)``` que convierte el primer caracter multi-byte de la cadena mb (examinando máximo 3) a un caracter amplio y lo almacena en ```wc```.  
* ```wctomb(mb, wc, 3)``` que convierte el caracter amplio ```wc``` en cadena multibyte de máximo 3 bytes que guarda en ```mb```.  

Tenga en cuenta que así emplee cadenas de más de un caracter, las funciones ```mblen```, ```mbrlen```, ```mbtowc``` y ```mbrtowc``` sólo tendrán en cuenta el primer caracter multibyte.  Por esto ```mblen("ñandu")``` sera el mismo ```mblen("ñ")``` es decir 2.

Estas funciones acuden al locale para determinar la codificación que emplean. Además emplean un estado interno, que depende del locale, por lo que al cambiar el locale no garantizan su ejecución.  Hay versiones análogas pero que recibien el estado: ```mbrlen```, ```mbrtowc```, ```wcrtomb```

El estándar C99 {2} limita las fuentes de un programa a estar en alguna codificación multibye, la implementación del preprocesador ```cpp``` de adJ limita las fuentes de un programa en C a estar codificadas en ASCII o en UTF-8 (ver
http://linux.web.cern.ch/linux/scientific4/docs/rhel-cpp-en-4/implementation-details.html),   de acuerdo al estándar C99 los caracteres en tiempo de ejecución de los programas compilados, se convierte del conjunto de caracteres de la fuente al conjunto de caracteres de ejecución durante la compilación, en el caso de adJ la librería de C (libc) sólo soporta codificaciones de caracteres en un sólo byte (como ISO-8859-1) o UTF-8.  


! ```wchar``` - Caracteres amplios

Un caracter amplio puede representar cualquier de los caracteres del locale que use, cabe en un objeto de tipo ```wchar_t``` (ver {2}).   Cada caracter de este tipo tiene un número asociado de tipo ```wint_t```. En adJ ambos se representan con enteros de 32 bits.

En C pueden emplearse caracteres y cadenas literales representadas en caracteres amplios agregando el prefijo L.  Por ejemplo:
* L'a' 
* L"esta es una cadena de wchar_t"

En las fuentes de un programa, los literales de caracteres amplios con el gcc de adJ se esperan en codificación UTF-8. Si se intentan codificar por ejemplo en LATIN1 obtendrá un error como:
<pre>
error: converting to execution character set: Invalid argument
</pre>

Internamente el compilador y la librería de C de adJ convierten y almacenan en un wchar_t el valor Unicode del caracter, por esto la letra ñ aunque en UTF-8 se representa con 2 bytes, al asignarse a un wchar_t se convertirá a 241 que corresponde al valor en Unicode (o 0xF1 en hexadecimal).  La letra griega &#946; corresponderá en un wchar_t a su valor Unicode 0x3B2 (o 946 en decimal).   

El compilador convierte tanto los literales de caracter como los literales de caracter amplio a enteros de 32 bits.  Esto se revela cuando ejecuta:
<pre>
printf("Tamaño de 'a'=%i, y de L'a'=%i\n", sizeof('a'), sizeof(L'a'));
</pre>
Que le dará 4 y 4.  Como los primeros 255 caracteres en Unicode corresponden a los caracteres ISO-8859-1 al almacenarse en caracteres amplios, los caracteres de Español y otros caracteres ISO-8859-1 requerirán sólo un byte.

Recuerde que en todo caso el tipo ```char``` continúa siendo de un byte, por lo que tendrá problemas si codifica las fuentes en UTF-8 y realiza una asignación como la siguiente, que sólo guarda los 8 bits finales:
<pre>
char n = 'ñ'
</pre>
(por eso en el ejemplo de la primera sección el caso de fuentes en UTF-8 da 177 que corresponde al segundo byte de la codificación multibyte de la ñ).

Pero si la asignación que hace es
<pre>
char n = L'ñ'
</pre>
La literal de caracter amplio se convertirá a Unicode representado en 4 bytes, pero por estar la ñ en ISO-8859-1, requiere un sólo byte, el byte más bajo de los 4 del caracter amplio y al convertir a un sólo byte, será justamente 241.  En todo caso recuerde que se perderá información en una asignación equivocada, como esta cuando asigne un carácter que no esté en el estándar ISO-8859-1 (como las letras griegas).


Hay varias funciones de entrada/salida que operan con caracteres amplios:
 
* ```putwchar()``` envia un caracter amplio a salida estándar, supone codificación de acuerdo a locale.
* ```getwchar()``` espera de entrada estándar un carácter, supone codificación de acuerdo al locale.
* ```getwc(f)``` y ```fgetwc(f)``` leen un caracter del archivo ```f```
* ```putwc(f,c)``` y ```fputwc(f, c)``` escriben un caracter al archivo ```f```
* ```ungetwc(f)``` Retorna un caracter al buffer del archivo ```f``` para que sea leido la próxima vez por ```getwc```
* fwide(f, m) establece modo m de un archivo orientado a byte (m ``` 0) o orientado a caracteres amplios (m>0).  Sin embargo con gcc en adJ no hay diferencia.

Se definen especificadores de formato para ```printf``` como "%ls" y "%lc" para imprimir cadenas y caracterew wchar_t.  Tenga cuidado con estos, si los aplica a caracteres de otros tipos su programa puede terminar abruptamente sin mayor explicación.



!Cadenas multibyte y cadenas de caracteres amplios

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


!Funciones de cotejación

Las funciones ```strcmp``` y ```wcscmp``` comparan cadenas o cadenas de caracteres amplios respectivamente dando 0 si son iguales, un negativo si el primero operando es menor que el segundo o positivo si el primero es mayor que el segundo.  Sin embargo no tienen en cuenta el locale, sólo el valor numérico de sus operandos (lo cual da resultados errados para caracteres del español como ñ y vocales tildadas).

Por su parte las funciones ```strcoll``` y ```wcscoll``` si lo tienen en cuenta el locale y efectivamente permiten ordenar en el idioma especificado en el locale.

Las funciones ```strxfrm``` y ```wcsxfrm``` transforman la cadena que reciben a cadenas comparables con ```strcmp``` y ```wcscmp``` para que den el mismo resultado de ```strcoll``` y ```wcscoll``` respectivamente.

! xlocale - Funciones que operan directamente con locale 

uselocale
querylocale
newlocale
duplocale

##BIBLIOGRAFÍA

* {0} Biblia
* {2} ISO/IEC 9899:TC2 www.open-std.org/jtc1/sc22/wg14/www/docs/n1124.pdf
* {3} POSIX 2008. http://pubs.opengroup.org/onlinepubs/9699919799/
* {4} Explicación de UTF-8. http://en.wikipedia.org/wiki/UTF-8
