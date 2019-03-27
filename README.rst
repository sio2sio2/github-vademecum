Brevísimo recetario para manejarme en github
********************************************

Configuración del git local
===========================
Tras instalar es conveniente antes de empezar:

.. code-block:: console

   $ git config --global user.name "Perico de los Palotes"
   $ git config --glonal user.email "perico@example.com"
   $ git config --global credential-helper "cache --timeout=3600"

Obtención del repositorio
=========================

* En caso de que no lo tengamos aún copiado en el disco local:

  .. code-block:: console

     $ mkdir ~/Proyectos
     $ git clone https://github.com/sio2sio2/pruebas2.git

* En caso de que ya se tuviera el proyecto en local y se desee actualizar con los últimos cambios habidos en el servidor:

  .. code-block:: console

     $ cd ~/Proyectos/prueba2
     $ git pull
