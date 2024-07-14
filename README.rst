Vademécum
*********
Configuración del *git* local
=============================
Tras instalar es conveniente antes de empezar  indicar cuál es el usuario que
realizará los cambios:

.. code-block:: console

   $ git config --global user.name "Perico de los Palotes"
   $ git config --glonal user.email "perico@example.com"

Y agilizar las autenticaciones:

.. code-block:: console

   $ git config --global credential-helper "cache --timeout=3600"

Esta última línea conserva en memoria la contraseña durante 1 hora. El problema
de esta solución es que Github últimamente no permite el acceso mediante usuario
y contraseña, con lo que tenemos dos alternativas prácticas:

+ Utilizar un **token personal de acceso**, que sustituirá a la contraseña en
  las autenticaciones y se genera en la propia página de Github a través de
  ``Settings>Developer Settings>Access Personal Token``. Estos *tokens* son muy
  largos y complicados de memorizar y digitalizar por lo que se hace
  indispensable usar un gestor de contraseñas. En *Linux* puede usarse el
  derivado de `Gnome Keyring <https://wiki.gnome.org/Projects/GnomeKeyring>`_:

  .. code-block:: console

     $ git config --global credential-helper "/ruta/donde/este/git-credential-libsecret"
     $ git config --global credential.credentialStore secretservice

  Es probable que la distribución no disponga del complemento compilado y
  haya que hacerlo a mano como se explica en `este tutorial
  <https://itectec.com/ubuntu/ubuntu-the-correct-way-to-use-git-with-gnome-keyring-and-https-repos/>`_.

  Hecho esto, la primera vez habrá que autenticarse usando el usuario y este
  largo token, pero las restantes veces no será necesario, porque las
  credenciales quedarán almacenadas en el gestor de contraseñas.

+ Utilizar la autenticación OAuth para lo cual existe en las modernas
  distribuciones  :command:`git-credential-oauth`. Instalada en el sistema,
  puede usarse esta configuración:

  .. code-block:: console

     $ git config --global credential-helper "cache --timeout=3600"
     $ git config --global credential-helper oauth

  De esta forma, al autenticarnos, el proceso de validación nos derivará el
  navegador para nos validemos y durante una hora las credenciales serán
  recordadas. Es importante que este sea el orden (primero la declaración de
  usar la caché), porque de lo contrario la caché no funcionará.

Creación de un repositorio
==========================
Al crear un repositorio tenemos dos alternativas:

- Clonar un repositorio ya creado.
- Crear uno *ex novo*.

Clonación
---------

.. code-block:: console

  $ git clone https://github.com/sio2sio2/proyecto.git

Creación
--------

.. code-block:: console

   $ mkdir proyecto
   $ cd proyecto
   $ git init
   $ cat > .gitignore
   **.bak
   **.swp
   $ echo "Documentacion..." > README.rst

.. note:: ``,gitignore`` excluye ficheros que no queramos que formen parte
   del repositorio. En este caso, hemos incluido copias de seguridad y archivos
   de intercambio de **vim**. La notación :code:`**.ext`` significa
   todo fichero con la extensión indicada esté en el subdirectorio que esté.

Si se desea crear un nuevo repositorio en Github_ a partir de este nuevo, hay
dos posibilidades:

- Crearlo a través de la web sin inicializarlo (o sea sin crearle un
  ``README.md``),

- Crearlo a través de la `API REST de Github
  <https://developer.github.com/v3/repos/>`_, después de haber `definido un
  token apropiado <https://github.com/settings/tokens>`_, esto es, un *token*
  que tenga habilitado al menos el `alcance public_repo
  <https://developer.github.com/apps/building-oauth-apps/understanding-scopes-for-oauth-apps/#available-scopes>`_:

  .. code-block:: console

     $ wget -qO - -S --header "Authorization: token XXXXXXX" \
         --post-data '{"name": "proyecto", "description": "Una descripción del proyecto"}' \
         https://api.github.com/user/repos

para después realizar una `Actualización`_ y finalmente:

.. code-block:: console

   $ git remote add origin https://github.com/sio2sio2/proyecto.git
   $ git push -u origin master

Actualización
=============
Desde local
-----------
Si se han modificado ficheros en el repositorio local, pueden comprobarse los
cambios del siguiente modo:

.. code-block:: console

   $ cd proyecto
   $ git status  # Conocemos la rama en la que estamos y cuáles son los ficheros.
   $ git diff    # Si queremos ver las diferencias entre los ficheros.
   $ git diff -- fichero  # Para ver los cambios en el fichero referido.

Para llevar a cabo la actualización:

.. code-block:: console

   $ git add --all .
   $ git commit -m "Comentario que describa la actualización"

Si la actualización requiere un comentario más exaustivo. se puede utilizar un
fichero con sintaxis Markdown_::

   $ git commit -F comentario.md

Por último, si queremos sincronizar con el directorio remoto:

.. code-block:: console

   $ git push

Desde remoto
------------
Si ya se disponía de una copia local del repositorio, pero la versión remota de
éste cambió (p.e. porque otro desarrollador realizó cambios), pueden obtenerse
las últimas modificaciones así:

.. code-block:: console

   $ cd proyecto
   $ git pull

.. warning:: Tenga en cuenta que es común que un proyecto disponga de
   distintas `ramas`_.

Ramas
=====
Las diversas ramas de un mismo repositorio permiten tener simultáneamente
distintas variantes del desarrollo. Por ejemplo, un desarrollador puede abrir
una rama nueva para implementar una nueva funcionalidad y, cuando la tenga lista
y se apruebe su inclusión, fusionarla con la rama principal.

La rama principal (la que se crea al crear el repositorio) se llama *master*. Es
común también crear otra rama llamada *development* donde van convergiendo las
distintas ramas que aparecen y desaparecen según las necesidades.

Creación
--------
.. code-block:: console

   $ git checkout -b development

Esto clona la rama en la que se esté actualmente (supongamos que *master*) en
otra llama *development* y nos camb ia a ella. Ahora si se hace:

.. code-block:: console

   $ git status

Comprobaremos que nos encontramos en la rama *development*. Ahora podemos
realizar cambios sobre esta rama y actualizarma como ya se ha visto. Si queremos
subir la rama al servidor de Github_:

.. code-block:: console

   $ git push -u origin development

pero sólo esta primera vez para sincronizar la rama con una rama aún inexistente
en el servidor también llamada *development*. A partir de este momento, las
siguientes sincronizaciones sí podremos hacerlas como ya se indicó:

.. code-block:: console

   $ git push

Cambio
------
Para cambiar entre ramas:

.. code-block:: console

   $ git checkout master

donde *master* es el nombre de la rama a la que queremos cambiar.

Fusión
------
Para fusionar la rama *development* con con la actual (*master*):

.. code-block:: console

   $ git merge development

Borrado
-------
Para borrar una rama local:

.. code-block:: console

   $ git branch -d development

Y si se quiere borrar del repositorio remoto:

.. code-block:: console

   $ git push origin :development

Versiones
=========
Para etiquetar un estado como versión:

.. code-block:: console

   $ git tag -a v1.0 -m "Versión 1.0"
   $ git push --tags

Para eliminar una etiqueta en local basta con:

.. code-block:: console

   $ git tag -d v1.0

y para eliminarla en el repositorio remoto, se hace de la misma forma que cuando
se eliminan ramas:

.. code-block:: console

   $ git push origin :v1.0

Regresión
=========
Commit antiguo
--------------
En alguna ocasión puede ser útil volver a un estado antiguo. Para ello podemos
crear una rama independiente:

.. code-block:: console

   $ git checkout -b test

y cambiar al commit que deseemos:

.. code-block::

   $ git log --oneline
   f446e5e (HEAD -> test) Comentario...
   8abe916 Comentario...
   2c595db Comentario...
   bfe76b5 Comentario...
   $ git reset 2c595db
   $ git restore .

Archivos
--------
Si queremos deshacer los cambios hechos en un archivo que aún no se han fijado
con un commit tenemos dos posibilidaes:

* Si ya se hizo un ``git add`` (el archivo aparece en verde al ahcer un *status*),
  podemos hacer:

  .. code-block:: console

     $ git restore --staged --worktree -- path/archivo

  Si se prescinde de ``--worktree`` el archivo  quedará en el estado anterior al
  ``git add`` (en rojo).  Si se especifica un directorio se restaurán todos los
  archivos modificados dentro de él.

* Si el archivo está modificado, pero sin haber hecho un ``git add`` (aparece en
  rojo):

  .. code-block:: console

     $ get restore -- path/archivo
     
  Esto eliminará todos los cambios en el archivo.

Varias cuentas
==============
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

   Esta técnica parece no funcionar. Al menos con :command:`git-credential-libsecret`

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

   .. code-block:: console

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

.. code-block::

   # Esto es ~/.gitconfig
   [user]
   name = Perico de los Palotes
   email = perico@example.com

   [includeIf "gitdir:~/Programacion/Trabajo/"]
   path = ~/Programacion/Trabajo/.gitconfig

Y en ese segundo archivo de configuración:

.. code-block::

   # Esto es ~/Programacion/Trabajo/.gitconfig
   [user]
   name = Pedro Palotes
   email = pedropalotes@corporacion.com


.. _Github: https://github.com
.. _Markdown:  https://daringfireball.net/projects/markdown/
