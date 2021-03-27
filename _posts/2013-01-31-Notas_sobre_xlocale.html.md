---
title: Notas_sobre_xlocale
date: 2013-01-31
tags:
---
# Detalle de la implementación

## Estrategia

Una serie de parches ordenados (en un orden lógico-histórico) para facilitar rehacerlos cuando cambio OpenBSD en las partes afectadas cambia mucho de una versión a otra:

# Corrección de errores (en nuestra opinión)
# Mejoras sencillas
# Un locale global en orden: ctype, tiempo, cotejación, numérico, monetario
# xlocale


## ctype
! Historia y referencias 

! 6.2

- OpenBSD implementa completo xlocale, pero vacío, es decir existen todas las funciones pero no operan con los argumentos como se espera. En realidad sólo hay soporte para LC_CTYPE con codificaciones ASCII y UTF-8.
- locale_t es void * (definido en locale.h), sólo soporta 3 locales definidos en locale/rune.h: NONE (apuntador a 0), C (1) y UTF-8 (2).
- cómo es tan simple lo soportado (ctype con ascii y utf-8), en realidad no existe un locale global sino en tiempo de ejecución cuando se requiere algo de ctype se examina __mb_cur_max si es uno es C o NONE, i.e ASCII en otro caso es UTF-8.  Se nota este comportamiento en mbsinit, mbrtowc, mbsrtowcs, mbsnrtowcs, wcrtomb, wcstombs, wcsnrtombs definidos en locale/multibyte_citrus.c  setlocale mantiene la variable static global_locname y global_locale que cambia para retornar el valor que se establezca.  global_locale es un vector con 6 cadenas, cada una con el locale de la respectiva categoria. global_locname es o bien un cadena con el nombre común de todas las categorias o bien los 6 nombres de las categorias concatenados en orden separados con '/'.
- En 6.1 y 6.2 _CurrentRuneLocales es bien _Utf8RuneLocale o _DefaultRuneLocale, como se ve en locale/__mb_cur_max.c y locale/setlocale.c


! 6.0 
* OpenBSD elimina todas las codificaciones diferentes a UTF-8 y US-ASCII


! Antes

En {2} se describe la codificación interna como la empleada por datos de tipo wchar_t, y la codificación externa la empleada por ejemplo en archivos o en sálida estándar. Allí mismo se describe que Unicode no es apropiado para lenguajes asiáticos, pues para incluir la escritura de varios paises incurrieron en una unificación (llamada unificación han), entre varias escrituras asiáticas pero que hace perder fidelidad (Koreano, Chino, Taiwanes y Japones).  En el momento de este escrito OpenBSD puede emplear como codificaciones internas LATIN-1 o UTF-8.


En adJ rune_t, wchar_t y wint_t son enteros de 32 bits que representan lo mismo.

Un ```locale_t loc``` tiene un ```_RuneLocale *``` que se obtiene asi:
<pre>
FIX_LOCALE(loc)
_RuneLocale *runes = XLOCALE_CTYPE(loc)->runes;
</pre>

Si además se quiere obtener el valor del campo ```__mb_sb_limit``` de ```loc``` en ```limit```:
<pre>
int limit;
_RuneLocale *runes = __runes_for_locale(loc, &limit);
</pre>

Entre la implementación de FreeBSD 9 y OpenBSD para ```_RuneLocale```:

* En FreeBSD se han renombrado estructuras y funciones que han tenido que dejarse con prototipo en /usr/include, para evitar polución.  
* En FreeBSD ```wctype_t``` es _RuneType que es ```unsigned long``` que es una mascara que se usa directamente con las constantes de tipo _CTYPE_A, _CTYPE_B, etc. mientras que en OpenBSD es un apuntador void *, tipicamente _WCTypeEntry *, y la mascara de ```wctype_t c``` se obtiene con ```((_WCTypeEntry *)c)->te_mask```
* En la estructura _RuneLocale de ambos hay  un arreglo ```unsigned long __runetype[_CACHED_RUNES];``` pero en la de OpenBSD además un arreglo 
```_WCTypeEntry  rl_wctype[_WCTYPE_NINDEXES];``` que es el usado por las funciones que retornan tipo de los caracteres.
* En OpenBSD para 

_RuneEntry:
| __OpenBSD__ | __FreBSD__ | Contenido |
| rune_t re_min; | !__rune_t !__min | Primer rune del rango |
| rune_t re_max; | !__rune_t !__max | Último rune (inclusive) del rango |
| rune_t re_map; | !__rune_t !__map | Al que primero mapea en mapeos |
| _RuneType *re_rune_types; | unsigned long   *__types; | Arreglo de tipos en el rango |

_RuneLocale definido en locale/runetype.h:

| __OpenBSD__ | __FreBSD__ | Contenido |
| rl_magic[8] |  | Identificación de la versión |
| char rl_encoding[32] | | Nombre en ASCII de esta codificación |
| rune_t rl_invalid_rune | | |
| _RuneType  rl_runetype[_CACHED_RUNES] | | |
| rune_t  rl_maplower[_CACHED_RUNES] | | |
| rune_t  rl_mapupper[_CACHED_RUNES] | | |
| _RuneRange rl_runetype_ext | | |
| _RuneRange rl_maplower_ext | | |
| _RuneRange rl_mapupper_ext | | |
| void *rl_variable | | |
| size_t rl_variable_len | | |
| char  *rl_codeset | | |
| struct _citrus_ctype_rec *rl_citrus_ctype | | |
| _WCTransEntry rl_wctrans[_WCTRANS_NINDEXES] | | |
| _WCTypeEntry rl_wctype[_WCTYPE_NINDEXES] | | |
| struct old_tabs *rl_tabs | | |

* Hasta OpenBSD 6.1 el locale sólo constaba de ```LC_CTYPE``` que se representa ba con un ```RuneLocale```, hay uno por defecto (```_DefaultLocale```) y uno actual (```_CurrentLocale```), los runes que se cargan quedan a manera de cache en una lista ```localetable_head``` del archivo ```setrunelocale.c```.  En POSIX 2008 con ```xlocale``` se pueden definir todos los locales que se deseen, hay un locale por defecto (```C```) y uno global.  xlocale incluye funciones para crear y borrar locales. Cada locale consta de componentes para cada tipo ```LC_COLLATE```, ```LC_CTYPE``` y así sucesivamente.  En la implementación de FreeBSD El componente de ```LC_CTYPE``` incluye un ```RuneLocale```.
* La implementación de cotejación de FreeBSD emplea cadenas definidas como u_char * y tablas para UCHAR_MAX caracteres, es decir supone que los caracteres están en un solo byte. Tal esquema no soporta codificaciones multi-byte como UTF-8 --que es soportado por OpenBSD.  Por esto se adaptaron las funciones para emplear caracteres amplios --aún cuando por el momento se mantuvieron las tablas de cotejación de un sólo byte, para próximas mejoras vale la pena revisar http://www.unicode.org/reports/tr10/.
* A partir de OpenBSD 6.1 hay una implementación vacía de xlocale (es decir define todas las funciones, pero no hacen nada).  Un locale_t (definido en locale.h) es un apuntador a void. Los locales estándar son las direcciones 0 (_LOCALE_NONE), 1 (_LOCALE_C) y 2 (_LOCALE_UTF8), y no se soportan otros locales.

NetBSD y FreeBSD han avanzado en implementación, la de FreeBSD está más adelante, las dos han empelado vías diferentes, una discusión de la parte de la implementación en NetBSD está en:
http://muc.lists.netbsd.tech.userlevel.narkive.com/g2IPspZ8/ctype-chrtbl-rune-tables-and-posix2008

Se basó especialmente en la implementación de FreeBSD, el soporte incluido hasta OpenBSD 5.2 de locale se basó en el de NetBSD.  Tanto el de NetBSD como el de FreeBSD parecen estar basados en una implementación del proyecto CITRUS.

En el caso de FreeBSD renombraron constantes y estructuras de runetype.h y las pusieron en _ctype.h, que es incluido por wctype.h 

#define _RUNETYPE_SWS   30              /* Bits to shift to get width */
#define _CTYPE_SWS      30              /* Bits to shift to get width */

runetype en OpenBSD 5.2 solo está en fuentes, en NetBSD está en /usr/include así como en FreeBSD.  En http://svnweb.freebsd.org/base/release/5.0.0/include/?view=log se menciona  el problema de polución de <runetype.h> asi "Solve the <runetype.h> pollution problem by disabling inline optimizations when a standard has been requested, except when the inline optimizations are also specifically requested."  



Lo que se hizo hasta 6.1 fue: 
* Incluir las extensiones de FreeBSD al xlocale estándar (definidio en POSIX 2008)   
* Se incluyeron y adaptaron las páginas del manual de FreeBSD para las funciones de xlocale y se agregó documentación faltante en FreeBSD: mblen_l (3), mbstowcs_l(3), mbtowc_l(3), wcstombs_l(3), wctomb_l(3), strcoll_l(3), strxfrm_l(3), strcasecmp_l(3), strcasestr_l(3), strncasecmp_l(3).
* FreeBSD incluye en sus fuentes mapeos entre diversas codificaciones y soporte para una amplia gama de locales.  OpenBSD sólo incluye mapeos entre ASCII (e ISO-8859-1) y UTF-8.
* Por mejorar: 
** posible mezcla entre locale/none.c y citrus/citrus_none.c
** Remplazar CurrentLocale y DefaultLocale con los campos apropiados de xlocale_global_locale y xlocale_default_locale

!
En nuestro caso movimos runetype.h a /usr/include

Comparando OpenBSD con FreeBSD vemos 
En OpebBSD runetype.h
typedef uint32_t      rune_t;
typedef uint32_t _RuneType;

En FreeBSD sys/_ctypes.h

_ct_rune_t
__rune_t

http://osdir.com/ml/os.freebsd.devel.standards/2002-09/msg00000.html


##Implementación de ```LC_NUMERIC```

En OpenBSD 5.4 está iniciado el soporte se declara en:
```src/sys/sys/localedef.h``` como:

<pre>
typedef struct
{
        const char *decimal_point;
        const char *thousands_sep;
        const char *grouping;
} _NumericLocale;

extern const _NumericLocale *_CurrentNumericLocale;
extern const _NumericLocale  _DefaultNumericLocale;
</pre>
y en ```lib/libc/locale/_def_numeric.c```
con 
<pre>
const _NumericLocale _DefaultNumericLocale =
{
        ".",
        "",
        ""
};

const _NumericLocale *_CurrentNumericLocale = &_DefaultNumericLocale;
</pre>

Se usa en ```localeconv.c```:
<pre>
    if (__nlocale_changed) {
        /* LC_NUMERIC */
        ret.decimal_point       = (char *) _CurrentNumericLocale->decimal_point;
        ret.thousands_sep       = (char *) _CurrentNumericLocale->thousands_sep;
        ret.grouping            = (char *) _CurrentNumericLocale->grouping;
        __nlocale_changed = 0;
    }
</pre>
y en ```nl_langinfo```
<pre>
        case RADIXCHAR:
                s = _CurrentNumericLocale->decimal_point;
                break;
        case THOUSEP:
                s = _CurrentNumericLocale->thousands_sep;
                break;
</pre>


Exactamente la misma estructura en FreeBSD 9.x se declara en ```lib/libc/locale/lnumeric.h``` como:
<pre>
struct lc_numeric_T {
        const char      *decimal_point;
        const char      *thousands_sep;
        const char      *grouping;
};
</pre>

Se usa en ```lnumeric.c``` y en ```localeconv.c``` como:
<pre>
    if (loc->numeric_locale_changed) {
        /* LC_NUMERIC part */
        struct lc_numeric_T * nptr;

#define N_ASSIGN_STR(NAME) (ret->NAME = (char*)nptr->NAME)

        nptr = __get_current_numeric_locale(loc);
        N_ASSIGN_STR(decimal_point);
        N_ASSIGN_STR(thousands_sep);
        N_ASSIGN_STR(grouping);
        loc->numeric_locale_changed = 0;
    }
</pre>

Por su parte en ```nl_langinfo.c``` dice:
<pre>
        case RADIXCHAR:
                ret = (char*) __get_current_numeric_locale(loc)->decimal_point;
                break;
        case THOUSEP:
                ret = (char*) __get_current_numeric_locale(loc)->thousands_sep;
                break;
</pre>

En lnumeric.c esta estructura se carga directamente de los archivos /usr/share/locale/*/LC_NUMERIC con la función ```__part_load_locale``` definida
en ```ldpart.c```   La función ldpart.c carga líneas de un archivo y las pones en un ```char **``` --o como en el caso de lnumerico los ```char *``` de la estructura ```lc_numeric_T```.

Como son identicas ```lc_numeric_T``` y ```_NumericLocale``` remplazamos usos de la primera por la segunda.

Además debe mantenerse el invariante

<pre>
&(__xlocale_global_numeric.locale) ``` _CurrentNumericLocale
</pre>

Cabe anotar que en la estructura _NumericLocale,  ```const char *grouping;``` es extensión a POSIX donde bastaría un char para especificar cada cuanto se agrupan los miles.  En FreeBSD en las fuentes de LC_NUMERIC se usa por ejemplo 3;3 para indicar que el primer grupo es de 3, y los subsiguientes son de 3.   Por ejemplo ```/usr/src/share/locale/numericdef/hi_IN.ISCII-DEV.src``` dice ```2;3```, mientras que ```/usr/src/share/locale/numericdef/it_IT.ISO8859-1.src``` indica ```0;0```.  En muchos archivos está en -1 (que se transforma a 127), y se ha probado lo estándar ```3``` con éxito.


##Implementación de ```LC_MONETARY```

La situación es similar al caso de ```LC_NUMERIC```, excepto que las estructructuras
```_MonetaryLocale``` declarada en ```sys/sys/localedef.h``` al compararla con la
estructura ```lc_monetary_T``` declarada en ```lmonetary.h``` resulta tener menos campos y los campos finales son de tipo char en la primera y de tipo char* en la segunda.

Consideramos que en FreeBSD lo dejaron así para facilitar la carga por parte de la misma funcion ```__part_load_locale```, pero nos parece mejor dejar ```_MonetaryLocale``` y sólo usar ```lc_monetary_T``` para cargar.



<pre>
typedef struct
{
        char *int_curr_symbol;
        char *currency_symbol;
        char *mon_decimal_point;
        char *mon_thousands_sep;
        char *mon_grouping;
        char *positive_sign;
        char *negative_sign;
        char int_frac_digits;
        char frac_digits;
        char p_cs_precedes;
        char p_sep_by_space;
        char n_cs_precedes;
        char n_sep_by_space;
        char p_sign_posn;
        char n_sign_posn;
        char int_p_cs_precedes;
        char int_n_cs_precedes;
        char int_p_sep_by_space;
        char int_n_sep_by_space;
        char int_p_sign_posn;
        char int_n_sign_posn;
} _MonetaryLocale;

extern const _MonetaryLocale *_CurrentMonetaryLocale;
extern const _MonetaryLocale  _DefaultMonetaryLocale;
</pre>


##Implementación de ```LC_TIME```

Cambio orden de estructura de OpenBSD 5.5 a 5.6 de
```src/sys/sys/localedef.h```, en 5.6 es: 

<pre>
typedef struct {
        const char *abday![7];
        const char *day![7];
        const char *abmon![12];
        const char *mon![12];
        const char *am_pm![2];
        const char *d_t_fmt;
        const char *d_fmt;
        const char *t_fmt;
        const char *t_fmt_ampm;
} _TimeLocale;
</pre>

Por otra parte struct ```lc_time_T``` de ```strftime.c``` y ```wcsftime.c``` es:

<pre>
struct lc_time_T {
       const char *    mon![MONSPERYEAR];
       const char *    month![MONSPERYEAR];
       const char *    wday![DAYSPERWEEK];
       const char *    weekday![DAYSPERWEEK];
       const char *    X_fmt;
       const char *    x_fmt;
       const char *    c_fmt;
       const char *    am;
       const char *    pm;
       const char *    date_fmt;
};
</pre>

Examinando valores con los que se inicializan ambas estructuras encontramos la
siguiente equivalencia

| __ _LocaleTime__ | __lc_time_T__ | __Bandera de  ```strftime```__ |
| mon |       month   | |
| abday |    wday    | |
| day |   weekday  | |
| t_fmt |  X_fmt   | %X |
| d_fmt |   x_fmt | %x  |
| d_t_fmt |    c_fmt | %c |
| am_pm![2] |     am, pm |
|   |     date_fmt | %+ |
| t_fmt_ampm |   ampm_fmt |  %r |

Entonces podría remplazarse ```lc_time_T``` por ```_LocaleTime```, excepto por ```date_fmt``` por lo que propusimo añadir ```date_fmt``` a ```_LocaleTime```.

En ```strftime``` de 5.6 se encuentra la función  ```static struct lc_time_T * _loc(void)``` que 
es la base para la carga de todos los componentes en FreeBSD tras renombrarla a ```__part_load_locale``` y dejarla en ```ldpart.c```.

Haremos lo análogo posteriormente pero renombrando de otra forma.


## Implementación de xlocale

| Componente | LC_componente | LC_componente_MASK | XLC_componente |
| COLLATE | LC_COLLATE = 1 | !LC_COLLATE_MASK = 2 | XLC_COLLATE = 0 |
| CTYPE | 2 | 4 | 1 |
| MONETARY | 3 | 8 | 2 |
| NUMERIC | 4 | 16 | 3 |
| TIME | 5 | 32 | 4 |
| MESSAGES | 6 | 64 | 5 |
| ALL | 7 | 126 | XLC_LAST = 6 |


##BIBLIOGRAFÍA

* {2} Citrus project true multilingual support for BSD operating systems. Jun-Ichiro Itojun Hagino. FREENIX. 2001. http://static.usenix.org/event/usenix01/freenix01/hagino.html
* {3} Paul Borman. runes - an implementation of setlocale(3) and all its friends. https://groups.google.com/forum/?fromgroups=#!msg/comp.sources.unix/uCc4PLo-l9U/lX9xt5lOpygJ
