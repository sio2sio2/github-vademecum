Vademécum
*********
Un *repositorio Git* es un conjunto estructurado de archivos (un proyecto, si se
prefiere) del que se conserva su historial de cambios (*commits*), de manera que
se puede restaurar uno o hacer comparaciones entre ellos. Este historial,
además, no es líneal, sino que puede bifurcarse en ramas que divergen o incluso
que llegado el caso acaban convergiendo después:

.. _figura 1:

.. image:: files/git.svg

.. _conceptos:

Conceptos
=========
Antes de empezar, es conveniente desglosar una serie de conceptos sobre `Git
<https://git-scm.com/>`_ que nos ayudarán a entender las recetas posteriores.

Repositorio
-----------
En la `figura 1`_ los boliches representan versiones del
proyecto que conserva el repositorio. Estas versiones se crean al realizar una
confirmación de cambios (*commit*) y se identifican por un *hash* (`af78aeb1`,
`e5c112ea`, etc).

.. _commit:

**Commit**
   Es una versión histórica de los archivos del proyecto y se identifica por un
   *hash* único. Un *commit* se origina al añadir modificaciones a un *commit*
   anterior. En la `figura 1`_ el commit `8a0d2ba8` es
   consecuencia de haber aplicado cambios (creación, borrado, modificación de
   archivos) al *commit* justamente anterior, el `da7c8245`.

**Tag** (etiqueta)
   Es una referencia estática a un *commit* determinado que permite identificarlo
   con un nombre alternativo fácilmente recordable. Muy comúnmente, si el
   repositorio es de código, identifica una versión de la aplicación y en
   consecuencia los nombres suelen ser *v1.2.0* o *2025.04*.

   Al definir una etiqueta pueden asociarse metadatos como un comentario, un
   autor, etc.

.. _HEAD:

**HEAD**
   Es el puntero que señala la versión (*commit*) activa del proyecto. Esto
   quiero decir que, si consultamos los archivos, nos encontraremos con los
   contenidos que tenían en esa versión. Por tanto, si cambiamos el puntero a
   otro *commit* distinto, el contenido del directorio del proyecto cambiará
   para acomodarse al estado que tenía cuando se produjo este *commit*:
   apareceran archivos, desaparecerán otros o habrá archivos que cambien su
   contenido.

.. _rama:

**Branch** (rama)
   Es un puntero (una referencia dinámica) a un *commit*. Es dinámico en la
   medida en que, si estamos en una rama (o lo que es lo mismo, el *HEAD* apunta
   al mismo *commit* que dicha rama), al hacer cambios y generar un nuevo
   *commit*, tanto el *HEAD* como la rama apuntarán a ese nuevo *commit*. En el
   ejemplo, la rama activa es **main**, ya que el *commit* activo es `674e3172`.
   Si hacemos algumos cambios y generamos un nuevo *commit* `f932eb8`, tanto
   *HEAD* como **main** apuntarán automáticamente a este nuevo *commit*:

   Aunque técnicamente una rama es esto, un puntero dinámico, dado que a partir
   de un commit_ se pueden calcular todos sus ancestros se suele identificar (o
   confundir) la rama con el *historial de la rama*, es decir, con todo los
   commits a los que en algún momento apuntó la *rama*.

   .. _figura 2:

   .. image:: files/git2.svg

Como un *commit* debe tener un padre a partir del cual se originó, Git_ permite
referir un ascendiente a partir de un descediente usando el operador virgulilla
(``~``). Volviendo a la `figura 1`_:

* ``HEAD~1`` es el padre de `HEAD`, o sea, `3b02bbgf`.
* ``experimental~2`` es el abuelo de la rama `experimental`, o sea, `558302c9`.
* ``8a0d2ba8~1`` es el padre de `8a0d2ba8`, o sea, `da7c8245`.

Ahora bien, hay un caso especial en que hay más de un padre: las bifurcaciones.
`v1.0` (o `3b02bb6f` como queramos referirlo) tiene dos padres: `e5c112ea` y
`deba176e`. En estos caso, se considera el primer padre, aquel que fue de la
misma rama; y el segundo padre, aquel que fue de otra rama. Por tanto,
`e5c112ea` es el primer padre y `deba176e` es el segundo. En estos casos:

* ``v1.0~1`` es el primer padre, esto es, `e5c112ea`; y ``v1.0~2`` el padre del
  primer padre, esto es, `af78aeb1`.
* ``v1.0^2`` es el segundo padre, esto es, `deba176e`; y ``v1.0^2~1`` el padre
  del segundo padre, esto es, `8a0d2ba8`.

Una característica a tener en cuenta de los *commits* es que necesitan estar
referenciados de algún modo para que Git_ no los acabe desechando. Los padres
siempre están referenciados por los hijos, como acabamos de ver, por lo que en
realidad el problema podría estar en los *commits* terminales si no les hemos
asociado ninguna rama. Por ejemplo, en el gráfico, `ddbaebaa` es un nodo
terminal y está referenciado por la rama `experimental`, por lo que no se
perderá. Sin embargo, si no estuviera referenciado por esta rama, no tendría
referencia ninguna y a la postre Git_ lo desecharía a él y a `7172f279` y
`558302c9`.

Por último, Git_ recuerda todos estos estados distintos del proyecto, gracias al
subdirectorio `.git` que crea dentro del directorio de proyecto.

En principio, un *repositorio Git* se circunscribe a un directorio local de
trabajo, que es aquel en el que realizamos cambios. Sin embargo, el trabajo
colaborativo exige que exista un repositorio remoto del que los distintos
colaboradores puedan obtener el código ajeno y aportar el suyo propio. Es aquí
donde entra `GitHub <https://github.com>`_, que es una plataforma que alberga
*repositorios Git* y añade otras funcionalidades. La existencia del repositorio
local y el repositorio remoto exige un mecanismo de sincronización entre uno y
otro, tarea de la que también se encarga Git_.

Flujo de trabajo
----------------
Para la gestión de las modificaciones del estado del proyecto Git define tres
áreas:

.. _wd:

**Directorio de trabajo**
   Es el directorio donde están almacenado los archivos de un proyecto. Git
   detecta los cambios (adiciones, borrados, modificaciones) que el usuario
   realiza y, cuando esto ocurre, los marca; de manera que es consciente de cuál
   es la diferencia entre este directorio y el *área de preparación* que veremos
   a continuación.

   En el propio directorio de proyecto el usuario puede crear un archivo llamado
   `.gitignore` que contiene la relación de directorios y archivos que queremos
   que Git_ deje fuera de esta vigilancia y, en consecuencia, del repositorio,
   aunque esté ubicados dentro del directorio de proyecto. Por ejemplo:

   .. code-block:: bash

      # Ejemplos de rutas en .gitignore
      docs/   # Se obviará la existencia de este directorio y todo su contenido.
      TODO    # Archivo que se ignora
      *.swp   # No se tiene en cuenta ningún archivo .swp, pero sólo en la raiz.
      **.bak  # No se tiene en cuenta ningún archivo .bak, en todo el proyecto.

   En cualquier caso, revise más adelante cómo se realizan exclusiones_, porque
   hay más localizaciones que sirven para este propósito.

.. _staging:

**Área de preparación** (*staging area*)
   Es el área que recoge los cambios que el usuario marca para su pase al
   próximo *commit*. Por tanto, cuando se crea un nuevo *commit* no todos los
   cambios efectuados pasan a constituir el *commit*, sino sólo aquellos que el
   usuario añadió a este área.

**Repositorio**
   El repositorio en sí del que hablamos en el apartado anterior.

.. _preliminares:

Preliminares
============
Antes de empezar a manejar cualquier repositorio es necesario hacer algunas
configuraciones previas básicas.

**Identificación**
   Git_ exige la identificación del usuario que realiza un *commit*, por lo
   que es indispensable definir una:

   .. code-block:: console

      $ git config --global user.name "Perico de los Palotes"
      $ git config --global user.email "perico@example.com"

   Las configuraciones de Git_, de menor a mayor prioridad, pueden ser:

   * Para todo el sistema (``--system``) que implica que afectan a todos los
     usuarios. En un *UNIX* se almacenan en el archivo ``/etc/gitconfig``.

   * Para todos los repositorios de un mismo usuario (``--global``). Se
     almacenan en el archivo ``~/.gitconfig``.

   * Para un repositorio particular (``--local``). Se almancenan en
     ``repo/.git/config``, siendo ``repo`` el directorio del proyecto.

   El archivo de configuración tiene formato `INI
   <https://es.wikipedia.org/wiki/INI_(extensi%C3%B3n_de_archivo)>`_ y puedo
   editarse a posteriori, bien directamente con el editor, bien con la orden:

   .. code-block:: console

      $ git config --global --edit

   .. caution:: Cuando se usan plataformas de desarrollo como GitHub_ la cuenta
      de correo electrónico sirve para identificar qué usuario de la plataforma
      es el que ha realizado el cambio. Por tanto, la cuenta debería estar
      relacionada con la de un usuario dado de alta en la plataforma.

**Autenticación**
   La otra parte básica de la configuración es la autenticación ante una
   plataforma de desarrollo (GitHub_ en nuestro caso) para poder sincronizar
   repositorios. Ya se ha definido el usuario (``user.email``) para esto no
   basta para unas credenciales. Tradicionalmente GitHub_ permitía la
   autenticación mediante usuario y contraseña, por lo que al intentar llevar a
   cabo la sincronización se requería interactivamente la contraseña y se podía
   añadir:

   .. code-block:: console

      $ git config --global credential-helper "cache --timeout=3600"

   para que se guardara en caché la próxima hora y no tener que teclearla
   constantemente en caso de hacer más sincronizaciones dentro de ese plazo.
   Pero GitHub_ eliminó la autenticación mediante este método de
   usuario/contraseña, por lo que tenemos que recurrir a alternativas:

   .. _pat:

   a. Token personal de acceso (**Personal access token**)

      Estos tokens pueden generarse a través de la propia página de GitHub_ en
      ``Settings>Developer Settings>Access Personal Token`` y sustituyen a la
      contraseña. Sin embargo, son muy largos y complicados de memorizar y
      digitalizar, por lo que es inviable usarlos como se usaría una contraseña.
      Sólo son útiles si se utiliza un gestor de contraseñas como `Gnome
      Keyring <https://wiki.gnome.org/Projects/GnomeKeyring>`_. Para ello
      puede echarse mano del complemento `git-credential-libsecret`:

      .. code-block:: console

         $ git config --global credential-helper "/ruta/donde/este/git-credential-libsecret"
         $ git config --global credential.credentialStore secretservice

      De este modo, la primera vez que el usuario se autentique necesitará
      digitalizar el largo token, pero éste se almacenará en el gestor de
      contraseñas con lo que no volverá a ser necesario más.

      .. note:: Si la distribución de *Linux* no dispone del complemento
         compilado será necesario `saber compilarlo
         <https://gist.github.com/JonasGroeger/4393203cc846da558fec1531cdd822db>`_.

      .. _oauth:

   b. Autenticación OAuth

      En este caso, el proceso de validación nos derivará al navegador para que
      introduzcamos nuestras credenciales. Requiere un complemento que suelen
      traer incorporado las distribuciones (`git-credential-oauth`) Como en el
      caso de la contraseña podemos definir un tiempo de vigencia para que no se
      nos vuelvan a pedir credenciales en un tiempo:

      .. code-block:: console

         $ git config --global credential-helper "cache --timeout=3600"
         $ git config --global credential-helper oauth

      .. caution:: Es importante mantener este orden (primero la declaración de
         la caché), porque de lo contrario la caché no funcionará.

   .. note:: Veremos más adelante cómo gestionar `varias cuentas`_.

**Ejecución de sentencias**
   La orden **git**, al igual que otras muchas como **docker** o **apt**, basa
   su sintaxis en añadir un subcomando que agrupa funcionalidades relacionadas.
   De este modo, ``git branch`` agrupa operaciones relacionadas con el manejo de
   ramas, mientras que ``git tag``, operacion es relacionadas con el manejo de
   etiquetas.

Inicialización
==============
Tras haber configurado *Git* nuestra primera tarea es crear un repositorio
local. Hay dos posibilidades:

Desde remoto
------------
Es el caso más simple y basta con hacer una `clonación`_.

.. _init:

*Ex novo*
---------
En este caso comenzamos un nuevo proyecto y, por tanto, no existe nada:

.. code-block:: console

   $ mkdir proyecto
   $ cd proyecto
   $ git init
   $ # Ahora podemos hacer algunos preparativos
   $ cat > .gitignore
   node_modules/

   $ echo '# Proyecto estupendo' > README.md

Esto crea el repositorio local, pero a diferencia de cuando se clona no habrá
relación con uno remoto. Debemos, pues, crear uno remoto por una de esta dos
vías:

#. Crearlo a través de la interfaz web de GitHub_ sin llegar a inicializarlo (o
   sea, sin crearle un archivo `README`.
#. Crearlo a través de la `API REST de GitHub
   <https://developer.github.com/v3/repos/>`_, después de haber definido `un
   token apropiado <https://github.com/settings/tokens>`_ que tenga habilitado
   al menos el `alcance public_repo
   <https://developer.github.com/apps/building-oauth-apps/understanding-scopes-for-oauth-apps/#available-scopes>`_:

   .. code-block:: console

      $ wget -qO - -S --header "Authorization: token XXXXXXX" \
         --post-data '{"name": "proyecto", "description": "Una descripción del proyecto"}' \
         https://api.github.com/user/repos

Por cualquiera de estas dos vías tendremos un nuevo repositorio, pero aún sin
relación con el que creamos localmente. Aún es necesario definir la
`sincronización`_.

Trabajo
=======
En el directorio de proyecto deberemos editar o añadir o borrar archivos. Esto
provocará unos cambios respecto al *commit* activo (*HEAD*). Para comprobar
estos cambios:

.. _git status:

.. code-block:: console

   $ git status                      # Muestra cambios en todo el repositorio.
   $ git status -- src/ .gitignore   # Muestra cambios sólo en las rutas proporcionadas.

.. note:: El comportamiento que presenta este subcomando ``git status`` es común
   a muchos otros que también operan sobre archivos del repositorio: si no se
   especifica ninguna ruta, se sobreentiende que quiere aplicarse a todo. Hay
   sin embargo, algunos como ``git add`` o ``git restore`` que obliga a indicar
   sobre qué se quieren aplicar.

Deberemos observar cómo el programa nos señala en rojo los archivos con
modificaciones respecto al *HEAD*. En este punto pueden, no obstante,
distinguirse tres tipos de archivos respecto a los cambios:

a. Archivos **rastreados** (*tracked*), que son los archivos que ya
   existían y se han modificado o borrado.
#. Archivos **sin rastreo** (*untracked*), que son los archivos que se han
   creado nuevos.
#. Archivos excluidos_, sobre los que Git_ jamás tomará ninguna decisión.

.. note:: Hay diferencias entre un archivo rastreado y no rastreado. Por
   ejemplo, si en este punto quisiéramos hacer una `comparación`_ entre la
   versión modificada de los archivos y la versión incluida en el *HEAD*, los
   archivos sin rastreo no entrarían dentro de la comparación. Podemos, sin
   embargo, hacer que Git_ siga los archivos sin llegar a pasarlos al
   área de intercambio con la orden:

   .. code-block:: console

      $ git add -N -- .

   Al hacer esto es como si los nuevos archivos hubieran pertenecido al *HEAD*,
   pero vacíos. Téngase presente esto, porque afecta al comportamiento posterior
   de ``restore``, que trataremos después.

.. _flujo:

Flujo habitual
--------------
Lo habitual cuando trabajamos es que estemos trabajando (cambiando cosas) sobre un commit_ asociado
a una rama_, o dicho de otro forma, que el puntero HEAD_ apunte al mismo commit_
que alguna de las rama_\ s. De esta forma, cada vez que generamos un commit_
nuevo, el puntero HEAD_ avanza y, solidariamente, también lo hace la rama_. Pero
antes de generar el nuevo *commit*, debemos decidir cuáles de los archivos
queremos añadir al `área de preparación`_, porque sólo los cambios situados en
este área pasarán finalmente a formar parte del siguiente *commit*.Para ello
debemos hacer:

.. code-block:: console

   $ git add -- source/ .gitignore

.. warning:: Esta orden no tiene en cuenta archivos borrados. Por ejemplo si
   dentro del directorio `source/` hemos borrado algún archivo, éste no se
   marcará como eliminable en el área de trabajo.

Otro opción es indicar que queremos añadir absolutamente todos los cambios (lo
cuál también implica marcar como eliminables los borrados) que se han producido
en el *directorio de trabajo*:

.. code-block:: console

   $ git add --all

.. note:: El subcomando **add** es uno de esos que exige que se le indiqué sobre
   qué rutas se debe aplicar. Sin embargo, si se le añade el argumento ``--all``
   para a no requerir ruta y se sobreentiende que es todo el repositorio.

Si ahora se volviese a consultar el *status*, los archivos pasados al área de
intercambio se verán en verde y los modificados en el área de trabajo seguirán
en rojo. El paso habitual ahora es crear un nuevo *commit* que incluya los
cambios registrados en el área de preparación de todo el repositorio:

.. code-block:: console

   $ git commit -m "Descripción del cambio"

Una alternativa es:

.. code-block:: console

   $ git commit -F archivo_con_comentarios

si es que queremos ser más descriptivos al explicar qué aporta el nuevo
*commit*. En el caso de incluir el comentario con un archivo, la convención es
la siguiente:


.. code-block:: none

   Título breve (máx 50 caracteres en una línea)

   Descripción más detallada en varias líneas (máx 72 caracteres/línea)
   - Item 1
   - Ítem 2
   - Enlace: https://example.net/ejemplo (GitHub lo hará clickable).

En resumidas cuentas, el flujo habitual de trabajo es en la mayor parte de los
casos:

.. code-block:: none

   $ git add --all
   $ git commit -m "Añade soporte para..."

.. note:: Hacer directamente:

   .. code-block:: console

      $ git -a commit -m "Añade soporte para..."

   no es equivalente a las dos líneas anteriores en su conjunto puesto que
   quedan fuera del commit los archivos nuevos (sin rastreo).

Arrepentimientos
----------------
Puede ocurrir que antes del *commit* queramos deshacer algo de lo hecho. Para
devolver al directorio de trabajo cambios ya añadidos al área de preparación:

.. code-block:: console

   $ git restore --staged -- .

De este modo los cambios salen del área de preparación, pero permanecen en el
directorio de trabajo, por lo que no se pierden. Además, los archivos nuevos
pasan a rezar otra vez como sin rastreo. Por su parte, si lo que queremos es
revertir los cambios del directorio de trabajo (en rojo) para volver a dejar la
versión de *HEAD*:

.. code-block:: console

   $ git restore -- .  # No afecta a archivos nuevos (sin rastreo)
   $ git clean -df     # Descarta archivos nuevos.

Por último, es posible que ya hayamos hecho el *commit*, pero sólo en el
repositorio local sin haber llegado a sincronizar con el remoto. En ese caso,
revertir cambios puede hacerse de tres modos distintos:

.. code-block:: console

   $ git reset --soft HEAD~1   # Cambios vuelven al área de preparación (verde)
   $ git reset --mixed HEAD~1  # Cambios vuelven en el directorio de trabajo (rojo)
   $ git reset --hard HEAD~1   # Volvemos al commit anterior, perdiendo cambios
   # git reset HEAD~1          # Como --mixed

.. note:: Aquí es importante tener presente los conceptos_ que se expusieron
   sobre Git_. En realidad, al hacer este reset estamos moviendo el puntero
   *HEAD* al commit anterior y solidariamente el puntero de rama también. El
   *commit* que abandonamos (a menos que lo etiquetáramos) queda sin referencia
   y, en consecuencia, Git_ lo acabará desechando. Pero hasta entonces es
   recuperable. Por supuesto, podemos retroceder varios *commits* y no solo uno.

También tienen sentido:

.. code-block:: console

   $ git reset HEAD
   $ git reset --hard HEAD

El primero devolvería al directorio de trabajo los archivos que se hubieran
añadido al área de preparación (acción que ya vimos que se podía hacer de otra
forma); y el segundo desecha todos los cambios estén añadidos o no al área de
preparación.

.. warning:: Sin embargo, ``--hard`` no elimina archivos sin rastreo, por lo que
   los archivos creado nuevos permanencen y habrá que hacer para limpiar:

   .. code-block:: console

      $ git clean -df

.. _exclusiones:

Exclusiones
-----------
En principio, todo el contenido del directorio de repositorio pasa a estar
gestionado con Git_, excepto ``repo/.git/`` que contiene los archivos que le sirven
para gestionar todo el control de versiones. Ahora bien, se nos ofrece la
posibilidad de que haya contenido que Git_ obvie por completo.

``repo/.gitignore``
   Define exclusiones que afectan al repositorio, pero se comparten. Son útiles
   cuando identifican archivos que ningún desarrollador querrá incorporar. Por
   ejemplo, en el caso de un proyecto con Maven_ el directorio ``target/`` que
   contiene los archivos compilados.

``repo/.git/info/excludes``
   Afecta también al repositorio, pero sólo al local, ya que no se sincronizan
   con el remoto. Por tanto, se usa para exclusiones que interesan a un
   desarrollador individual. Por ejemplo, excluir archivos ``*.swp`` sólo
   interesa a un desarrollador que use vim_.

``~/.gitignore_global``
   Afecta a todos los repositorios locales de un usuario, pero como en el caso
   anterior no al resto de desarrolladores. Si el desarrollador usase siempre
   vim_, la exclusión sugerida justamente antes podríamos incluirla aquí.

Otras manipulaciones
--------------------
Git_ provee de algunos subcomandos para borrar archivos:

``git clean``
   sirve para borrar archivos sin rastreo:

   .. code-block:: console

      $ git clean -f
      $ git clean -df
      $ git clean -df -x
      $ git clean -df -- source/

   ``-f`` es necesaria para forzar a Git_ a borrar, porque de lo contrario
   se negará. ``-d`` borra también directorios sin rastreo, directorios vacíos y
   archivos sin rastreo que estuvieran dentro de un directorio sin rastreo.
   ``-x`` borra archivos excluidos. Es posible añadir la opción ``-n`` para que
   se informe de qué archivos se borrarán, pero sin borrarlos efectivamente.

``git rm``
   sirve para borrar archivos rastreados. La diferencia de borrarlos
   directamente con **rm** es que en este segundo caso, habría que hacer un
   ``git add`` adicional para que se marcaran como eliminables en el área de
   preparación, mientras que con este subcomando se puede realizar directamente
   un *commit* y los archivos desaparecerán.

   .. code-block:: console

      $ git rm -- source/index.rst

   Esto borra el archivo (como la orden ``rm`` de la *shell*), pero además
   marca el archivo como eliminable (aparece en verde). Una variante es añadir
   la opción ``--cached`` que en vez de borrar el archivo, lo convierte en no
   rastreado. Por tanto, si en vez de lo anterior hiciéramos:

   .. code-block::

      $ git rm --cached -- source/index.rst
      $ git add -- source/index.rst   # Deshacemos lo anterior

   En un primer momento el archivo pasa a no tener rastreo (aparecerá como tal
   en rojo). Con la segunda orden, lo rastreamos y apuntamos al área de
   preparación, pero como no hay ninguna diferencia entre esta área y el
   *commit*, Git_ deja detectar ninguna diferencia en el estado de
   ``source/index.rst``.

Git_ también provee de una orden equivalente a ``ls``, pero que permite
consultar archivos atendiendo a su estado de vigilancia:

``git ls-files``
   En principio, lista el resultado de considerar para el *HEAD* los cambios que
   ya se encuentran añadidos al área de preparación. Por tanto, muestra cuáles
   serán los archivos que formarán parte del siguiente *commit*, si no se hacen
   más cambios y se ejecuta inmediatamente ``git commit``:

   .. code-block::

      $ git ls-files
      $ git ls-files -c  # Equivalente.

   .. note:: En realidad muestra todos los archivos rastreados a menos que en el
      área de preparación el archivo esté añadido como para eliminar. Por eso un
      archivo nuevo que hayamos marcado como rastreado en el directorio de
      trabajo (con ``git add -N -- archivo`` también aparecerá).

   Ahora bien, podemos añadir opciones para modificar qué se lista:

   Si nos interesa ver qué archivos rastreados se han modificado en el directorio de
   trabajo, pero no han pasado aún al área de preparación

   .. code-block:: console

      $ git ls-files -m

   Esto, eso sí, no muestra archivos nuevos puesto que no son rastreados. Con:

   .. code-block:: console

      $ git ls-files -d

   sólo veremos los archivos borrados que aún no han pasado como eliminables al
   índice. También podemos ver los archivos no rastreados aún y los excluidos_
   con:

   .. code-block:: console

      $ git ls-files -o
   
   Y, si no nos interesa ver estos últimos:

   .. code-block:: console

      $ git ls-files -o --exclude-standard

   También podemos ver el efecto que tiene añadir una nueva exclusión:

   .. code-block:: console

      $ git ls-files -c -i --exclude=**.bak

   .. note:: Podemos añadir -o si queremos ver también el efecto sobre los aún
      no rastreados.

Sincronización
==============
No hemos tratado aún la operación de mantener sincronizado el repositorio local
con el remoto, por lo que dedicaremos este apartado a ello. Antes, no obstante,
de cualquier sincronización es necesario relacionar el repositorio local con uno
remoto, que recibe el nombre de *origin*. Para ver cuál es:

.. code-block:: console

   $ git remote -v
   origin  https://github.com/perico/proyecto.git (fetch)
   origin  https://github.com/perico/proyecto.git (push)

El primero es el repositorio del que obtener los datos, mientras que el segundo
aquel al que se envían, aunque por lo general suele ser el mismo. Si la orden no
devolviese nada, deberíamos definir cuál es:

.. code-block:: console

   $ git remote add origin https://github.com/perico/proyecto.git

.. _push:

Envío
-----
Ya explicamos que en su momento que el `flujo habitual`_ es que trabajemos sobre
una rama_; y, en consecuencia, la sincronización suelen estar asociada con la
sincronización del historial de una rama. Por este motivo, es necesario
establecer relaciones entre las ramas del repositorio local y las ramas del
repositorio remoto. Habitualmente, simplemente, se llaman igual, pero no es
obligatorio.

Supongamos que tuviéramos un rama local llamada *experimental* que queremos
subir al repositorio remoto por primera vez. En ese caso:

.. code-block:: console

   $ git push -u origin experimental  # Sube experimenal como experimental

Y si *experimental* es nuestra rama activa, podemos prescindir de especificarlo:

.. code-block:: console

   $ git push -u origin               # Sube la rama activa con su propio nombre

En ambos casos, hemos mantenido el nombre de la rama para nombra a su
correspondiente en el repositorio remoto "*origin*". Si queremos cambiar su
nombre:

.. code-block:: console

   $ git push -u origin/quimicefa     # Sube la rama activa con nombre quimicefa

Estas órdenes tienen el efecto de crear la rama remota, asociarla a la local,
sincronizar y, además, establecer como permanente el vínculo
(gracias a `-u`). Gracias a ello a partir de ahora, para actualizar la remota
con los cambios locales de la rama local bastará con:

.. code-block:: console

   $ git push        # Sube actualizaciones de la rama activa´.

Nótese que al realizar esta operación y enviar los cambios locales de la rama al
remoto, estamos enviando los nuevos *commit* del historial de la rama
*experimental* que han aparecido en el repositorio local y aún no están en el
repositorio remoto. Para que esto sea posible, el historial de la rama
*experimental* en el remoto debe estar retrasada. Por ejemplo, observando la
`figura 2`_, la rama *experimental* de nuestro repositorio local podría estar en
el *commit* `f932eb8` y la misma rama del remoto estar en `674e3172` o
`3b02bb6f`. En cambio, si Git_ detecta una situación en la que los historial son
incompatibles, como por ejemplo que en el historial del remoto tras el *commit*
`674e3172` hay un *commit* `113af45d` inexistente en el repositorio local, se
produce un conflicto y la operación no podrá realizarse. Ya veremos cómo
resolver esto.

Otro aspecto que no debe pasarnos desapercibido es que la última orden
sincroniza la rama activa porque no se ha especificado ninguna, pero puede
añadirse un último argumento para que se sincronice otra distinta:

.. code-block:: console

   $ git push migracion    # Sube actualizaciones de la rama migracion

Si existen otras ramas nuevas o modificadas en el repositorio local, no se
sincronizarán, aunque tenemos la opción de hacer:

.. code-block:: console

   $ git push --all        # Sube actualizaciones de todas las ramas

para que sí se sincronicen todas las ramas.

.. note:: Si nuestra rama activa experimental va adelantada varios *commits*
   respecto a su correspondiente rama remota (que se llama igual), la orden:

   .. code-block:: console

      $ git push HEAD~1:experimental

   subiría a la rama remota todos los *commits* locales, excepto el último. Por
   tanto, la rama local seguiría adelantada un *commit*.

.. _sincro:

En conclusión, para sincronizar un nuevo repositorio local con un nuevo
repositorio remoto debemos hacer:

.. code-block:: console

   $ git remote add origin https://github.com/perico/proyecto.git
   $ git push -u origin/main

.. note:: Desde hace algún tiempo la rama predeterminada de GitHub_ se llama
   *main*, mientras que la de Git_ sigue siendo *master*.

Clonación
---------
Si el proyecto ya existe en un repositorio remoto y queremos crear un
repositorio local basta con:

.. code-block:: console

   $ git clone https://github.com/perico/proyecto.git
   $ cd proyecto/

La orden creará un directorio llamado ``proyecto`` dentro del cual se albergará
el repositorio local. Esta clonación relacionará automáticamente ambos
repositorios, copia todos los *commits* y las etiquetas, y sólo crea la rama
predeterminada, que en GitHub es *main*. Por tanto HEAD_ coincidirá con el
último commit_ de *main*. Las ramas restantes, sin embargo, no se crean y habrá
que `copiar la rama remota en local`_ tal como se estudian más adelante:

.. code-block:: console

   $ git checkout origin/experimental

.. note:: Ya advertimos de que en un repositorio local creado *ex novo* con Git_
   la rama predeterminada se denomina *master*; y que, por consiguiente, si
   queremos sincronizar con GitHub tendremos que relacionar esta rama local
   *master* con una rama remota llamada *main*. Pero en el caso de que clonemos
   un repositorio remoto, la rama predeterminada en nuestro repositorio local
   también se llamará *main*.

Existen otras variantes de esta orden interesantes. Para copiar todos los
*commits* como en el caso anterior, pero que HEAD_ se sitúe en otra rama:

.. code-block:: console

   $ git clone --branch experimental https://github.com/perico/proyecto.git

O para copiar exclusivamente los *commits* asociados a la rama principal:

.. code-block:: console

   $ git clone --single-branch https://github.com/perico/proyecto.git

O para copiar exclusivamente los *commits* asociados a una rama distinta:

.. code-block:: console

   $ git clone --single-branch --branch experimental https://github.com/perico/proyecto.git

Obtención
---------
Si el repositorio remoto está adelantado y queremos comprobar (que no obtener)
qué nuevos *commit* han aumentado el historial de la rama podemos hacer:

.. code-block:: console

   $ git fetch

Tiene un comportamiento semejante a ``git push`` por lo que en este caso sólo
consultaremos la rama remota relacionada con la local activa. Por supuesto
podemos especificar otra rama distinta o consultar todas a la vez:

.. code-block::

   $ git fetch migracion
   $ git fetch --all

En este punto, es probable que nos interese conocer qué es lo que exactamente ha
cambiado (los *commits* deberían tener comentarios asociados) antes de seguir
adelante, por lo que deberíamos saber hacer la `comparación`_. Como aún no hemos
visto tal cosa, aprendamos directamente cómo obtener los *commits* para
sincronizarnos con el remoto:

.. code-block:: console

   $ git pull              # Obtiene los nuevos *commits* de rama activa
   $ git pull otra_rama    # Ídem pero de otra_rama.
   $ git pull --all        # Ídem pero de todas las ramas.

.. note:: No hace falta haber hecho antes un ``git fetch``.


Esto, ahora bien, sólo sirve si la rama ya existe en el repositorio local
también. Si no existiera, podemos hacer:

.. code-block:: console

   $ git fetch --all      # Conocemos las ramas que hay todos los repositorios remotos
   $ git fetch origin     # Ídem, pero solo del remoto llamado 'origin'.
   $ git checkout origin/nueva_rama # Crea la rama a partir de la remota

.. note:: Como alternativa a la primera orden podemos hacer:

   .. code-block:: console

      $ git fetch origin     # Conocemos las ramas del remoto llamado 'origin'.
   
   que en la práctica es casi equivalente (en realidad puede haber varios
   repositorios remotos).

Como en el caso de ``git push``, pero justo al revés, el historial de la rama en
el repositorio local debe estar retrasado y esta operación simplemente supone
añadir los *commits* nuevos que hay en el repositorio remoto. Ante una
incompatibilidad se producirá un conflicto y no será posible la sincronización.

Gestión de ramas
================
Un repositorio suele componerse de distintas ramas de modo que nos conviene
saber cómo consultarlas y crear nuevas.

Listado
-------

.. code-block:: console

   $ git branch --show-current     # Muestra el nombre de la rama activa.
   $ git branch                    # Lista todas las ramas del repositorio local
   $ git fetch origin              # Obtenemos previamente información del remoto.
   $ git branch -r                 # Lista las ramas conocidas del repositorio remoto.
   $ git ls-remote --heads origin  # Ídem, pero consultado en el repositorio directamente

Creación
--------
Si queremos cambiar a otra rama local:

.. code-block:: console

   $ git checkout test           # Cambia a la rama test que ya existe.
   $ git checkout -b test        # Copia HEAD a test y cambia a ella.
   $ git checkout -b test master # Copia master a test y cambia a ella.
   $ git branch test             # Copia HEAD a test, pero no cambia.
   $ git branch test master      # Copia master a test, pero no cambia.

.. note:: El cambio a una rama existente (la primera orden) puede provocar
   problemas si la rama actual tiene cambios no confirmados en el directorio de
   trabajo o el área de preparación. En un principio, Git_ intenta preservar
   estos cambios, pero no será posible y se generará un conflicto cuando algunos
   de estos cambios no confirmados se haya hecho en un archivo que en la rama de
   destino también ha cambiado.

En estas ordenes hemos utilizado como origen de la copia ramas (o siendo más
precisos *commits* apuntados por ramas). Como en realidad, el origen de la copia
es un commit_, podemos usar también los *hash* de los commits o las etiquetas_:

.. code-block:: console

   $ git checkout -b test v1.0
   $ git branch test 3b02bb6f

La nueva rama local no estará relacionada con ninguna rama remota, de modo que
la primera vez que intentemos sincronizarla habrá que hacer:

.. code-block:: console

   $ git push -u origin test     # Sincroniza por primera vez con el remoto.
   $ git push -u origin          # Ídem, si la rama actual es test
   $ git push -u origin/rtest    # Si queremos cambiar el nombre en el remoto.

.. _branch-repo2local:

También podemos copiar un commit del repositorio remoto para generar una nueva
rama local:

.. code-block:: console

   $ git checkout -b test origin/test       # Usamos un nombre de rama.
   $ git checkout -b test origin/v1.0       # Usamos una etiqueta.
   $ git checkout -b test origin/3b02bb6f   # Usamos el hash identificador.

La diferencia entre la primera orden cualquiera de las dos restantes, es que la
primera establece una relación entre dos ramas, mientras que las otras dos no
establecen relación con ninguna rama remota y habrá que obrar en consecuencia la
primera vez que hagamos una sincronización.

Fusión
------
Hasta aquí hemos visto cómo crear nuevas ramas. Las ramas, sin embargo, suelen
crearse para añadir alguna funcionalidad que acaba incorporándose a la rama
principal:

.. _figura 3:

.. image:: files/git3.svg

En el gráfico, la rama principal en un punto determinado (*commit* `6f2294b3`)
se bifurca, La propia rama principal sufre algunos cambios (*commit* `e5c112ea`)
mientras la rama *migración* sufre también cambios (dos *commits*) hasta llegar
al *commit* `deba176e`. En este punto, se considera que se han acabado de
definir los cambios introducidos por esta rama y se decide incorporarlos a la
rama principal:

.. code-block:: console

   $ git branch --show-current
   main
   $ git merge migracion         # Incorpora los cambios de migración en main

.. caution:: Es muy recomendable que no tengamos cambios pendientes en la rama en
   el momento en que decidimos hacer una fusión.

Si no hay conflictos, se generará automáticamente el *commit* `3b02bb6f` de la
rama *main*, pero puede no darse el caso. Los conflictos surgen cuando existen
uno o más archivos en que ambas ramas han introducido cambios desde la
bifurcación: 

a. Ambas han modificado un mismo archivo.
#. Una ha borrado un archivo modificado por otra.
#. Una ha renombrado un archivo modificado por otra.

En este caso, no se genera un nuevo *commit* y los archivos que no han originado
conflicto se añaden al área de preparación; mientras que los archivos
conflictivos quedan en el directorio de trabajo a la espera de que los
resolvamos manualmente. Una vez resueltos deberemos:

.. code-block:: console

   $ git add --all        # Los conflictos resueltos pasan al área de preparación
   $ git commit -m "Fusión de miracion con main"

También podríamos decidir que no queremos realizar la fusión, en vez de de
corregir conflictos:

.. code-block:: console

   $ git merge --abort

que es equivalente, si no había cambios pendientes de commit antes de la fusión
a:

.. code-block:: console

   $ git reset --hard HEAD

Eliminación
-----------
Otra operación importante es la de eliminación de ramas que ya no son necesarias
(por ejemplo, porque se completó la fusión con ellas). Las ramas locales se
eliminan con:

.. code-block:: console

   $ git branch -d migracion     # Borra una rama ya fusionada.

Ahora bien, si la rama en cuestión no hizo `fusión`_ con otra que sigue
existiendo, los *commits* de su historial desde que se bifurcó quedarán
huérfanos. En ese caso, Git_ se quejará e impedirá la eliminación. Para forzarla
es necesario:

.. code-block:: console

   $ git branch -D experimental  # Fuerza el borrado de una rama, aun no fusionada

Por último, podemos borrar todas las ramas locales que ya no existen en el
repositorio remoto:

.. code-block:: console

   $ git fetch --prune      # Borra ramas locales que ya no existen en el remoto

En cambio, si se quieren borrar ramas remotas, debe usarse esta otra orden:

.. code-block:: console

   $ git push origin -d migracion  # Borra la rama remota (fusionada o no).
   $ git push origin :migracion    # Equivalente a lo anterior.

.. note:: La segunda notación sigue la filosofía que vimos antes de:

   .. code-block:: console

      $ git push origin HEAD~1:experimental

   por lo que significa apuntar a "nada" (no hay nada antes de los ":") la rama
   experimental, que se reinterpreta como borrar *experimental*. En este caso,
   sin embargo, no puede inferirse *origin* porque no hay ningún *commit* local
   (como ``HEAD~1``) del que inferirlo.

Etiquetas
=========
Una etiqueta es una referencia estática a un commit_ concreto que se define para
marcar una particularidad de ese *commit* como, por ejemplo, el hecho de que
constituye una nueva versión del *software*.

En el repositorio local (y sólo en el local) es fácil establecer una etiqueta:

.. code-block:: console

   $ git tag -a "v1.0" -m "Versión 1.0"           # Etiqueta el commit apuntado por HEAD
   $ git tag -a "v1.0" -m "Version 1.0" HEAD      # Ídem
   $ git tag -a "v1.0" -m "Version 1.0" main      # Etiqueta el commit apuntado por main
   $ git tag -a "v1.0" -m "Version 1.0" 3b02bb6f  # Etiqueta el commit 3b02bb6f

Hemos ilustrado cómo se definen etiquetas **anotadas**, que pueden incluir una nota
(``-m``) y anadir automáticamente otros metadatos como el autor. También existen
etiquetas **ligeras** que carecen de metadatos y son más aptas para marcar de
forma temporal un commit_ determinado:

.. code-block:: console

   $ git tag "bifurcacion-aqui"        # Etiqueta ligera en el actual HEAD.

Las etiquetas no se sincronizan a priori al hacer un ``push``, así que hay que
subirlas expresamente:

.. code-block:: console

   $ git push origin "v1.0"            # Sube al remoto una etiqueta particular
   $ git push --tags                   # Sube al remoto todas las etiquetas

.. note:: La segunda órden, a pesar de las apariencias, sólo sube etiquetas, no
   sincroniza ramas. Bien es cierto que si la etiquetas que se suben apuntan a
   *commits* que no existen en el remoto, estos *commits* subirán y todos los
   necesarios para completar su historial, pero el puntero de la rama remota no
   se actualizará en ningún caso.

Aunque sincronizar ramas y subir etiquetas son dos operaciones diferentes hay
una forma de actualizar la rama del remoto a la vez que se suben las etiquetas
asociadas a los *commits* que deben subirse:

.. code-block:: console

   $ git push --follow-tags  # Actualiza la rama remota y sube tags asociadas

En sentido inverso:

.. code-block:: console

   $ git ls-remote --tags origin  # Lista las etiquetas remotas
   $ git fetch origin tag "v1.0"  # Obtiene una etiqueta en particular
   $ git fetch --tags             # Obtiene todas las etiquetas del remoto

.. note:: En caso de que los *commits* asociados a la etiqueta que obtiene, no
   existan en el repositorio local, éstos se descargarán.

Queda aún por saber cómo borrar, en principio, etiquetas locales:

.. code-block:: console

   $ git tag -d "v1.0"           # Borra la etiqueta local "v1.0"
   $ git fetch --prune-tags      # Borra etiquetas locales que no existen en remoto

.. note:: Esta última acción no es incompatible con borrar ramas:

   .. code-block:: console

      $ git fetch --prune --prune-tags

Y, para rematar, etiquetas remotas:

.. code-block:: console

   $ git push --delete origin "v1.0"      # Borra la etiqueta remota "v1.0"
   $ git push origin :v1.0                # Mismo efecto

Regresión
=========
Ya se expuso cómo ejecutar arrepentimientos_ cuando queríamos deshacer cambios
que afectaban únicamente al repositorio local, porque tales cambios nunca se
habían sincronizado con el escritorio remoto:

.. code-block:: console

   $ git reset HEAD~3  # Retrocede 3 *commits* (cambios al directorio de trabajo)

Y sabemos que también podemos añadir ``--soft`` o ``--hard``. Una variante
de esta regresión local es cuando hemos hecho varios *commits* locales en una
rama y queremos regresar al commit_ de la rama remota que está atrasada puesto
que nunca recibió estos *commits*:

.. code-block:: console

   $ git fetch
   $ git reset --hard origin/main  # Deshacemos todos los cambios locales

Ahora bien, lo más peliagudo es cuando las regresiones afectan al repositorio
remoto, puesto que otros programadores pueden estar compartiendo el desarrollo.
Pero, en principio, supongamos que no es así, y hay otros programadores
involucrados. En ese caso nos basta, simplemete, con retroceder en el local y
forzar la sincronización:

.. code-block:: console

   $ git reset --hard HEAD~1       # Nos deshacemos del último commit.
   $ git push --force              # Forzamos para subir la acción al remoto.

Cuando la operación, en cambio, sí involucra a terceros no se puede alterar a la
ligera el historial de la rama haciendo desaparecer *commits*. En este caso, es
necesario:

``git revert``
   Genera un nuevo commit_ consistente en deshacer las modificaciones que
   introdujo un commit_ antiguo del historial de la rama:

   .. code-block:: console

      $ git revert HEAD      # Revierte los cambios del último commit

   En este ejemplo, generamos un nuevo *commit* cuyos cambios consisten en
   revertir los que introdujo el último commit. También es posible hacer:

   .. code-block:: console

      $ git revert --no-commit HEAD

   que hace los mismo, pero sin llegar a generar el nuevo commit_: los cambios
   quedan en el área de preparación a la espera de que confirmemos.

   Si, en cambio, intentáramos deshacer *commits* más antiguos:

   .. code-block:: console

      $ git revert HEAD~1      # Revierte los cambios del penúltimo commit

   obsérvese que esto no revierte los cambios de los dos últimos *commits*, sino
   exclusivamente del penúltimo. El problema es que los cambios del último
   commit_ afectaran a archivos que también había tocado el penúltimo: en ese
   caso, puede producirse conflictos que Git_ es incapaz de resolver
   automáticamente; y nos obligará, como en el caso de la `fusión`_, a realizar
   cambios manuales.

   Un caso particular de reversión es cuando se pretender revertir un commit_
   que se produjo como consecuencia de una `fusión`_. En la `figura 3`_ el
   etiquetado como `v1.0`. El problema que surge aquí es que este commit_ tiene
   dos padres y al revertir no es a priori posible saber si se quieren eliminar
   los cambios que produjo un padre (`e5c112ea`) o el otro (`deba176e`). Si
   imaginamos, para entenderlo mejor) que HEAD_ se encuentra en `v1.0`:

   .. code-block:: console

      $ git revert -m 1 v1.0   # Nos quedamos con la rama principal.
      $ git revert -m 2 v1.0   # Nos quedamos con la rama migración.

   Es necesario añadir la opción ``-m``: con **1** nos quedamos con la rama
   principal, es decir, el nuevo *commit* presentará el mismo estado que
   `ed5c112ea`; con **2** nos quedamos con la rama principal y, por tanto, el
   nuevo *commit* presentará el mismo estado que `deba176e`.

Consulta de cambios
===================
Un control de versiones eficiente exige saber qué cambios se han obrado. Hasta
ahora sólo hemos introducido una herramientsa muy simple, `git status`_, que
permite resumidamente comprobar qué archivos han cambiado respecto a HEAD_
(habitualmente, el último *commit* que hemos realizado).

Historial
---------
La consulta del historial de *commits* (ancestros) puede llevarse a cabo con:

.. code-block:: console

   $ git log               # Muestra el historial de ancestros de HEAD.
   $ git log --oneline     # Ídem, pero más sucinto.
   $ git log -n5 --oneline # Sólo muestra los últimos cinco.

Admite como argumento un commit_ concreto para que se muestre el historial de
ancestros de dicho *commit* y no el de HEAD_:

.. code-block:: console

   $ git log deba176e
   $ git log "v1.0"
   $ git log experimental

Incluso podemos ver el historial de algún commit_ remoto:

.. code-block:: console

   $ git fetch
   $ git log --oneline origin/main

Los historial, sin embargo, no tiene por qué ser lineales ya que en una
`fusión`_ hay dos padres, cuando esto ocurre la orden siempre sigue al primer
padre y si quisiéramos hacernos una idea mejor tendríamos que recurrir a:

.. code-block:: console

   $ git log --graph --decorate --oneline

Para ver información más completa sobre un *commit* completo, puede usarse:

.. code-block:: console

   $ git show "v1.0"                # Tnformación detallada sobre un commit (diff incluido)
   $ git show --name-only "v1.0"    # Ídem, pero sólo nombres de archivos modificados

Por último si hacemos:

.. code-block:: console

   $ git blame -- src/index.rst

se nos indicará que commit_ fue el último que modificó cada linea del archivo.

Comparación
-----------
La otra consulta fundamental es la de conocer cuáles han sido en concreto los
cambios para lo cual debe usarse el subcomando ``git diff``. En principio, si
hemos cambios después de realizar un *commit*:

.. code-block:: console

   $ git diff              # Diferencias entre HEAD y el directorio de trabajo
   $ git diff --name-only  # Como antes, pero sólo indica los nombres de archivo
   $ git diff --staged     # Diferencias entre HEAD y el área de preparación

La primera orden mostrará las diferencias entre la versión de los archivos
rastreados que se encuentra en HEAD_ y la que se encuentra en el directorio de
trabajo (que se muestran en rojo al usar `git status`_); la segunda orden, las
diferencias entre la que se encuentra en HEAD_ y la que se encuentra ya añadida
en el área de preparación (en verde al usar `git status`_). Si nos interesa sólo
comparar archivos o directorio particulares y no todo el repositorio:

.. code-block:: console

   $ git diff -- src/ .gitignore
   $ git diff --staged -- src/ .gitignore

Cuando se quieren ver ambas diferencias (tanto con el directorio de trabajo como
con el área de presentación) podemos hacer:

.. code-block:: console

   $ git diff HEAD      # Diferencias entre HEAD y el estado actual de los archivos
   $ git diff "v1.0"    # Ídem pero con el commit etiquetado "v1.0"

Incluso podemos comparar con un *commit* remoto:

.. code-block:: console

   $ git diff origin/main     # Diferencias entre el estado actual y el último commit subido

También podemos comprar cambios entre dos *commits* distintos:

.. code-block:: console

   $ git diff HEAD~1..HEAD          # Compara HEAD con el commit anterior
   $ git diff HEAD~1..HEAD -- src/  # Ídem, pero sólo dentro de src.

Y en el caso particular de que nos dispongamos a hacer una `fusión`_, puede
interesarnos la sintaxis con tres puntos:

.. code-block:: console

   $ git diff main...migracion      # Cambios para que migración se convierta en main

Si hubiéramos hecho esto antes de crear el commit_ etiquetado con `v1.0`, Git_
compararía ambas ramas respecto al punto en que divergieron.

Puede también interesarnos ejecutar estas órdenes añadiendo las opciones ``-w``
(no tiene en cuenta espaciados) o ``--color-words`` (sólo colorea las palabras
de diferencia, no la línea completa).

Otras operaciones
=================

Varias cuentas
--------------
Cuando se tienen varias cuentas en Github (p.e. una personal y otra de trabajo)
nos encontraremos con el problema que el gestor de contraseñas, en principio,
almacena estas credenciales atendiendo únicamente el nombre de máquina
(`github.com`), por lo que únicamente podremos usar unas únicas credenciales.
Tenemos al menos tres alternativas para solucionarlo:

#. Usar el nombre del usuario como parte del nombre de máquina, es decir, en
   vez de haber relacionado directorio local con repositorio remoto así:

   .. code-block:: console

      $ git remote add origin https://github.com/sio2sio2/proyecto.git

   deberíamos relacionarlo así:

   .. code-block:: console

      $ git remote add origin https://sio2sio2@github.com/sio2sio2/proyecto.git

   Y en caso de que está relación ya la hubiéramos hecho, aún podríamos acceder
   al archivo `.git/config` y editar la URL en la directiva correspondiente para
   añadir el usuario al nombre.

   La ventaja de este procedimiento es que no necesitaremos introducir
   nuevamente el token cada vez que creemos un repositorio relacionado con el
   usuario.

   Esta técnica parece no funcionar. Al menos con **git-credential-libsecret**.

#. Añadir a la configuración global:

   .. code-block:: console

      $ git config --global credential.useHttpPath "true"

   que provoca que al apuntar las credenciales en el gestor se use toda la URL y
   no solamente el nombre de máquina. La desventaja de esta solución es que cada
   vez que creemos un repositorio nuevo, tendremos que facilitar las
   credenciales.

#. Usar sendos métodos de validación (helper) diferentes para lo que podemos usar
   la configuración condicional que trataremos después o definir las
   credenciales dependiendo de cuál sea la ruta con la que sincronicemos:

   .. code-block:: ini

      [credential "https://github.com/sio2sio2/"]
      helper = /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
      credentialStore = secretservice
      [credential "https://github.com/otrousuario/"]
      helper = "cache --timeout=7200"
      helper = oauth

Cualquiera de las tres alternativas nos solucionaría la autenticación. Sin
embargo, también es probable que queramos cambiar quién será el que rece como
autor de los cambios. Para ello puede utilizarse la `configuración condicional
<https://github.blog/2017-05-10-git-2-13-has-been-released/#conditional-configuration>`_
introducida a partir de :program:`git` 2.13. De este modo, si tuviéramos la
prevención de que los desarrollos de uno de los usuarios siempre estuvieran
dentro de la misma ruta podríamos hacer:

.. code-block:: ini

   # Esto es ~/.gitconfig
   [user]
   name = Perico de los Palotes
   email = perico@example.com
   [credential]
   helper = /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
   credentialStore = secretservice

   [includeIf "gitdir:~/Programacion/Trabajo/"]
   path = ~/Programacion/Trabajo/.gitconfig

Y en ese segundo archivo de configuración:

.. code-block:: ini

   # Esto es ~/Programacion/Trabajo/.gitconfig
   [user]
   name = Pedro Palotes
   email = pedropalotes@corporacion.com
   # Podemos añadir también autenticación
   [credential]
   helper =
   helper = "cache --timeout=7200"
   helper = oauth

Limpieza
--------


.. _Personal Access Token: #pat
.. _Autenticación OAuth: #oauth
.. _varias cuentas: #multiple
.. _área de preparación: #staging
.. _sincronización: #sincro
.. _Maven: https://maven.apache.org/
.. _vim: https://www.vim.org/
.. _excluidos: #exclusiones
.. _copiar la rama remota en local: #branch-repo2local
