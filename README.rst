Vademécum
*********
Configuración del *git* local
=============================
Tras instalar es conveniente antes de empezar:

.. code-block:: console

   $ git config --global user.name "Perico de los Palotes"
   $ git config --glonal user.email "perico@example.com"
   $ git config --global credential-helper "cache --timeout=3600"

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

.. note:: :file:`,gitignore` excluye ficheros que no queramos que formen parte
   del repositorio. En este caso, hemos incluido copias de seguridad y archivos
   de intercambio de **vim**. La notación :code:`**.ext`` significa
   todo fichero con la extensión indicada esté en el subdirectorio que esté.

Si se desea crear un nuevo repositorio en Github_ a partir de este nuevo, hay
dos posibilidades:

- Crearlo a través de la web sin inicializarlo (o sea sin crearle un
  :file:`README.md`),

- Crearlo a través de la `API REST de Github
  <https://developer.github.com/v3/repos/>`_, después de haber `definido un
  token apropiado <https://github.com/settings/tokens>`_, esto es, un *token*
  que tenga habilitado al menos el `alcance public_repo
  <https://developer.github.com/apps/building-oauth-apps/understanding-scopes-for-oauth-apps/#available-scopes>`_:

  .. code-block:: console

     $ wget -qO - -S --header "Authorization: token XXXXXXX" \
         --post-data '{"name": "proyecto", "description": "Una descripción del proyecto"}' \
         https://api.github.com/user/repos

para después :ref:`actualizar el repositorio <update>`_ y finalmente:

.. code-block::

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

Y si queremos sincronizar con el directorio remoto:

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
   :ref:`distintas ramas <branch>`.

.. SEGUIR POR AQUÍ

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

Fusion
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

.. _Github: https://github.com
