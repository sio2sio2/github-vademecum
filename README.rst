Brevísimo recetario para manejarme en github
********************************************

Configuración del git local
===========================
Tras instalar es conveniente antes de empezar:

.. code-block:: console

   $ git config --global user.name "Perico de los Palotes"
   $ git config --glonal user.email "perico@example.com"
   $ git config --global credential-helper "cache --timeout=3600"

Obtención del repositorio remoto
================================

* En caso de que no lo tengamos aún copiado en el disco local:

  .. code-block:: console

     $ mkdir ~/Proyectos
     $ git clone https://github.com/sio2sio2/pruebas2.git

* En caso de que ya se tuviera el proyecto en local y se desee actualizar con los últimos cambios habidos en el servidor:

  .. code-block:: console

     $ cd ~/Proyectos/prueba2
     $ git pull

Actualización del repositorio remoto
====================================
Después de haber hecho cambios, es posible ver qué ficheros
han aparecido, desaparecido o cambiado con:

.. code-block:: console

   $ git status 

Y para ver más precisamente los cambios:

.. code-block:: console

   $ git diff

Para actualizar el repositorio remoto debe hacerse:

1. Aplicar los cambios:

   .. code-block:: console

      $ git add --all .

   .. note:: Pueden excluirse ficheros locales de la operación creando el fichero ``.gitignore`` dentro del cual se incluyen los nombres de los ficheros, uno por línea.  Es posible usar los comodices de la *shell*.

2. Confirmar los cambios

   .. code-block:: console

      $ git commit -m "Comentario al respecto"

3. Guardar los cambios en el repositorio remoto

   .. code-block:: console

      $ git push

Ramas
=====
Para crear una nueva rama de desarrollo y saltar a ella:

.. code-block:: console

   $ git checkout -b test

Versiones
=========
Para etiquetar un estado como versión:

.. code-block:: console

   $ git tag -a 1.0 -m "Versión 1.0"
   $ git push --tags
