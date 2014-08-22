# Índice

1.  [Introducción](#introduccion)
2.  [Qué es Subversion](#queessubversion)
3.  [Creación y configuración de un repositorio](#creacionrepositorio)
4.  [Seguridad básica en un repositorio de Subversion](#seguridadbasica)
5.  [Estructura recomendada de un repositorio](#estructura)
6.  [Creación y manejo de proyectos](#creacionproyectos)
7.  [Comandos básicos](#comandosbasicos)
8.  [Creación y manejo de *branches* (ramas)](#creacionbranches)
9.  [Resumen de comandos](#resumencomandos)
10. [Anexo A: Script de inicio del servidor de Subversion — svnserve](#anexo)

# Introducción

Este documento describe el funcionamiento básico de un servidor y los clientes de la aplicación de control de versiones [Subversion](http://subversion.tigris.org/) bajo la plataforma Linux.

Con este documento será posible configurar el servidor, crear nuevos repositorios y manejar proyectos.

# Qué es Subversion

Subversion es un [sistema de control de versiones](http://es.wikipedia.org/wiki/Sistema_de_control_de_versi%C3%B3n), generalmente de código fuente de proyectos aunque se puede utilizar para cualquier contenido susceptible de ser versionado, desarrollado por la empresa Tigris y publicado bajo licencia libre Apache/BSD.

Este software está destinado a ser el sustituto natural del obsoleto [CVS](http://es.wikipedia.org/wiki/CVS).

La filosofía de Subversion es tener un repositorio central, donde se guarda el proyecto, y todos los clientes acceden a dicho repositorio, tanto para descargarse el proyecto como para subir los cambios realizados por los propios clientes.

Este enfoque es contrario a otros sistema de control de versiones para proyectos grandes como [Git](http://es.wikipedia.org/wiki/Git), donde cada cliente se trata como un cliente P2P y no hay un repositorio central, sino que todos los clientes son parte del repositorio.

# Creación y configuración de un repositorio

El repositorio básico de Subversion se crea en una máquina, generalmente dedicada, que va a actuar como servidor principal de desarrollo y es a esta máquina a la que tienen que acceder todos los clientes tanto para actualizar sus proyectos como para subir los cambios nuevos.

Para crear un repositorio nuevo donde albergar los proyectos, se usa el siguiente comando:

```bash
root@server:\~:\# svnadmin create /path/to/repository
```

Con este comando, en el directorio dado, creamos todos los archivos necesarios para que dicho directorio se comporte como un repositorio de Subversion. Debajo de este directorio se guardarán todas las versiones de nuestros proyectos así como la configuración de dicho repositorio y algunos elementos para la seguridad de acceso.

Además de tener el repositorio en la máquina servidor, es necesario tener una aplicación corriendo, `svnserve`, para que se pueda acceder de forma remota al repositorio. Este servidor viene cuando se instala Subversion y para arrancarlo basta con utilizar el comando siguiente:

```bash
root@server:\~:\# svnserve -d -T -r /path/to/repository
```

Con la opción -d hacemos que svnserve se comporte como un daemon, con -T hacemos que use threads en lugar de procesos y con -r indicamos donde está el repositorio.

En caso de que queramos arrancarlo cada vez que se inicie el sistema, será necesario configurar un script de inicio que hay que ubicar en el directorio /etc/init.d/ (en máquinas Debian y Ubuntu). Un ejemplo de este script se puede ver en el Anexo A de este documento.

Una vez funcionado el repositorio, para acceder al mismo mediante el comando svn, es necesario poner la ruta de la misma forma que una URL. Un ejemplo podría ser el siguiente (lista los contenidos del repositorio):

```bash
diego@server:\~:\$ svn list svn://servidor/repositorio
```

# Seguridad básica en un repositorio de Subversion

La configuración de la seguridad en un repositorio de Subversion se encuentra en el directorio `/ruta/al/repositorio/conf`. En en este directorio se encuentran tres archivos de configuración para el servidor `svnserve` junto con la seguridad de acceso al mismo:

-   `authz` : En este archivo se definen los grupos de usuarios del repositorio y sus correspondientes permisos. Con esto se consigue que sólo determinados usuarios tengan acceso de lectura y escritura y otros sólo de lectura.
-   `passwd` : En este archivo de definen los usuarios y contraseñas de todos los usuarios que pueden acceder al repositorio. Hay que tener cuidado con este archivo ya que las contraseñas no están encriptadas (uno de los grandes fallos del svnserve). Si se requiere que las contraseñas estén encriptadas, se debe usar mod\_dav mediante Apache (http) para el acceso al repositorio.
-   `svnserve.conf` : En este archivo está la configuración del servidor svnserve para el acceso remoto al repositorio con los siguientes campos:
    -   `anon-access = [none | read | write]` : Con none, nadie puede acceder al repositorio para lectura si no tiene usuario y contraseña. Con read pueden acceder todos sólo para lectura. Y con write pueden acceder todos sin usuario ni contraseña para lectura y escritura (esto no está recomendado ya que no se controlan los usuarios que hacen los cambios).
    -   `auth-access = [none | read | write]` : Aquí basta con poner write para que todos los usuarios necesiten usuario y contraseña para acceder al repositorio como lectura y escritura.
    -   `password-db` : Indica la ubicación de un archivo de contraseñas.
    -   `auth-db` : Indica la ubicación del archivo de reglas de acceso a usuarios y grupos.
    -   `realm` : Indica el nombre del repositorio.

Además de esta configuración básica, se puede usar Subversion mediante un túnel por SSH poniendo las URL de la forma `svn+ssh://servidor/repositorio/`. Para ello es necesario configurar el servidor `svnserve` añadiendo la clausula `[tunnels]` en `svnserve.conf` e indicando cual va a ser el programa de *tunneling* (rsh = ssh, por ejemplo).

# Estructura recomendada de un repositorio

Los repositorio de Subversion pueden tener cualquier estructura ya que se comportan igual que un directorio local al que pueden acceder varios usuarios, pero el equipo de desarrollo de Subversion recomienda una estructura de directorios para cada proyecto de la siguiente forma:

-   El proyecto debe estar en un directorio con un nombre los suficientemente descriptivo, por ejemplo `svn://servidor/repositorio/miproyecto/`.
-   Dentro del directorio del proyecto, debe haber tres subdirectorios de la siguiente forma:
    -   `svn://servidor/repositorio/miproyecto/trunk` : En este directorio se encontrará la rama principal de desarrollo (tronco), es decir, el código fuente principal del proyecto sobre el que se realizan los cambios principales y se pone en producción.
    -   `svn://servidor/repositorio/miproyecto/tags` : En este directorio se encontrarán las diferentes versiones del proyecto que ya no se van a cambiar a lo largo del desarrollo del mismo. Es algo así como un versión (*release*) inmodificable.
    -   `svn://servidor/repositorio/miproyecto/branches` : En este directorio se encontrarán versiones del proyecto de desarrollo, es decir, si en un momento dado una versión de desarrollo va a modificar tantos elementos de la rama principal (*trunk*) que no funcione dicha rama, se crea un nuevo *branch* donde se hacen los cambios pertinentes en desarrollo. Una vez que está listo el desarrollo en el *branch*, se combinan el *branch* con el *trunk* para introducir los cambios de desarrollo en la rama principal. Una vez finalizado, el *branch* creado para el desarrollo puede borrarse.

-   Para desplegar el proyecto en producción se utiliza la rama principal de la siguiente forma: `svn checkout svn://servidor/repositorio/miproyecto/trunk`.
-   Para desarrollo es lo mismo excepto en caso de estar desarrollando un branch donde el checkout es de ese branch de la siguiente forma: `svn checkout svn://servidor/repositorio/miproyecto/braches/mi-branch-de-desarrollo`.
-   En caso de necesitar cambiar en desarrollo de la rama principal a una rama de tipo *branch*, se puede usar el modificador `switch` del comando `svn` de la siguiente forma (dentro del directorio local de la rama principal —trunk—): `svn switch svn://servidor/repositorio/miproyecto/branches/mi-branch-de-desarrollo`.
-   Para volver a la rama principal desde un *branch*, ubicados en el directorio local del *branch*, se ejecuta el siguiente comando: `svn switch svn://servidor/repositorio/miproyecto/trunk`.

# Creación y manejo de proyectos

La primera vez que se importa un proyecto al repositorio, suponiendo que ya esté dicho repositorio creado, se realiza mediante el siguiente comando, ubicado donde está nuestros archivos fuente del proyecto:

```bash
diego@server:/proyectos/miproyecto:\$ svn import -m "Primera importación." svn://servidor/repositorio
```

El parámetro `-m` indica el mensaje de la importación, aunque si no se especifica nada, se ejecuta el editor que exista en la variable de entorno `$SVN_EDITOR` o `$EDITOR`.

Una vez hecha la primera importación, ya no es necesario seguir disponiendo de la copia local del proyecto recién importado. De hecho, si se siguen haciendo cambios en el directorio importado, éstos no serán incluidos en el repositorio.

Para que los cambios sean seguidos por el repositorio, es necesario hacer una descarga inicial del proyecto (de cualquier proyecto). Para ello, ubicado en el directorio donde se desea descargar el proyecto, se ejecuta el siguiente comando:

```bash
diego@server:/proyectos:\$ svn checkout svn://servidor/repositorio/miproyecto
```

A partir de aquí, ya se pueden hacer cambios en el código fuente del proyecto y que dichos cambios sean seguidos por el repositorio. Hay que tener en cuenta que los cambios no son seguidos automáticamente sino que es necesario usar los comandos del siguiente punto para tener un control total de los proyectos.

Además, una vez hecho el *checkout*, ya no es necesario disponer de copia local del proyecto si no se está desarrollando. Se puede borrar sin problemas siempre que todos los cambios estén en el repositorio.

# Comandos básicos

Una vez descargado localmente el proyecto en el que se va a trabajar, existen una serie de comandos para mantener actualizado tanto el proyecto en el repositorio como la copia local. Estos comandos son de la forma `svn [comando]` y teniendo que ejecutarse en el directorio o algún subdirectorio de la copia local. Los subcomandos `[comando]` son alguno de los siguientes:

-   `update` : Actualizar la copia local del proyecto con la versión más reciente del repositorio.
-   `commit` : Subir al repositorio los cambios realizados en la copia local. Además, hay que poner un mensaje de lo que se ha hecho en dichos cambios (con -m o mediante el editor que esté en la variable de entorno `$SVN_EDITOR`).
-   `add` : Con este comando se añade un nuevo archivo al repositorio. Hay que tener en cuenta que sólo se marca como añadido y hasta que no se haga el `commit` no se añade realmente.
-   `checkout` : Para descargar por primera vez una copia remota del proyecto a la máquina local.
-   `revert` : Restituye el archivo de copia de trabajo con los cambios de la última versión de la que se ha actualizado la copia local. Este comando no contacta con el servidor.
-   `list` : Lista los contenidos de un directorio dentro del repositorio.
-   `status` : Comprueba el estado de los archivos de la copia local con respecto a la última actualización de dicha copia. No realiza ninguna conexión con el repositorio.
-   `info` : Muestra información de algún directorio de algún proyecto del repositorio.
-   `lock` : Bloquea una ruta en el repositorio para que ningún usuario pueda hacer `commit` sobre dicha ruta.
-   `unlock` : Desbloquea rutas bloqueadas con `lock`.
-   `blame` : Imprime los cambios de un archivo respecto de versiones y quién los ha hecho.
-   `merge` : Une las diferencias entre dos copias de los archivos de trabajo que generalmente están en conflicto.
-   `resolved` : Indica que un conflicto ha sido resuelto.
-   `log` : Muestra los mensajes de log de la copia de trabajo. Es necesario hacer un `update` para que estén todos los logs del proyecto.
-   `switch` : Actualiza la copia de trabajo a un directorio distinto. Esto es útil para cambiar entre diferente branches que se verán en el siguiente punto.

# Creación y manejo de *branches* (ramas)

Un *branch* es una copia completa, generalmente de la rama principal de desarrollo, que se usa para desarrollar una funcionalidad que afecta al funcionamiento correcto del proyecto por lo que no se puede desarrollar directamente sobre el *trunk* del proyecto.

Mediante un *branch* no se interfiere en la rama principal del proyecto pudiendo combinar los cambios del *branch* al *trunk* en el momento de su finalización. Además, se pueden hacer *branches* de *branches* y combinarlos finalmente en la rama principal.

Para crear un *branch* a partir de la rama principal del proyecto, se puede ejecutar el siguiente comando:

```bash
diego@server:/proyectos/miproyecto:\$ svn copy svn://sevidor/repositorio/miproyecto/trunk svn://servidor/repositorio/miproyecto/branches/mi-branch-de-desarrollo -m "Creando un branch para pruebas."
```

Una vez creado la *branch*, para trabajar con ella es de la misma forma que para trabajar con el *trunk*: se hace un `checkout` del proyecto inicialmente, para luego seguir haciendo `commits` y `updates`.

Hay que tener en cuenta que tu *branch* debería estar lo más actualizado posible con respecto a la copia de la rama principal, independientemente del desarrollo que se esté haciendo sobre la misma. Para realizar una actualización una *branch*, lo que hay que hacer es una combinación de la *branch* de desarrollo con la rama principal de la siguiente forma:

```bash
diego@server:/proyectos/miproyecto-branch1:\$ svn merge svn://servidor/repositorio/miproyecto/trunk
```

Posteriormente habría que examinar los cambios con `svn diff` y, finalmente, si todos los cambios son correctos, se deben subir los cambios al repositorio mediante `svn commit`.

Para mezclar los cambios de un *branch* con la rama principal una vez terminados los cambios, se utiliza el siguiente comando:

```bash
diego@server:/proyectos/miproyecto:\$ svn merge -reintegrate svn://servidor/repositorio/miproyecto/branches/mi-branch-de-desarrollo\
diego@server:/proyectos/miproyecto:\$ svn commit -m "Combinado mi branch con el trunk."
```

# Resumen de comandos

Acción/Comando

Crear una *branch* o una *tag*

`svn copy URL1 URL2`

Intercambiar la copia de trabajo con una *branch* o una *tag*

`svn switch URL`

Sincronizar una *branch* con el *trunk*

`svn merge TRUNK_URL; svn commit`

Combinar una *branch* con el *trunk*

`svn merge ––reintegrate BRANCH_URL; svn commit`

Unir cambios

`svn merge -c REV URL; svn commit`

Prever una combinación (no hace nada)

`svn merge ––dry-run`

Abandonar los cambios

`svn revert -R .`

Copiar algo del historial

`svn copy URL@REV local-path`

Deshacer los cambios enviados

`svn merge -c REV URL; svn commit`

Crear una *tag* desde la copia de trabajo

`svn copy . TAG_URL`

Mover una *branch* o una *tag*

`svn mv URL1 URL2`

Eliminar una *branch* o una *tag*

`svn rm URL`

# Anexo A: Script de inicio del servidor de Subversion – svnserve

```bash
\#! /bin/sh -e\
\#\#\#\# BEGIN INIT INFO\
\# Provides: synserve\
\# Required-Start: \$syslog \$time \$local\_fs \$remote\_fs\
\# Required-Stop: \$syslog \$time \$local\_fs \$remote\_fs\
\# Default-Start: 2 3 4 5\
\# Default-Stop: S 0 1 6\
\# Short-Description: Subversion repository server\
\# Description: Debian init script for the Subversion repository server\
\#\#\# END INIT INFO\
\#\
\# Author: Rick Beton\
\#\
set -e

PATH=/bin:/usr/bin:/sbin:/usr/sbin\
 REPOS=/var/svnroot\
 DAEMON=/usr/bin/svnserve

test -x \$DAEMON || exit 0

. /lib/lsb/init-functions

case “\$1″ in\
 start)\
 log\_daemon\_msg “Starting Subversion server” “svnserve”\
 sudo -u svn \$DAEMON -d -T -r \$REPOS\
 \# start-stop-daemon ––start ––user svn ––oknodo ––exec \$DAEMON –– -d -T -r \$REPOS\
 log\_end\_msg \$?\
 ;;\
 stop)\
 log\_daemon\_msg “Stopping Subversion server” “svnserve”\
 killall \$DAEMON\
 \# start-stop-daemon ––stop ––oknodo ––name svnserve\
 log\_end\_msg \$?\
 ;;\
 force-reload|restart)\
 \$0 stop\
 \$0 start\
 ;;\
 \*)\
 echo “Usage: \$0 {start|stop|restart|force-reload}”\
 exit 1\
 ;;\
 esac

exit 0
```
