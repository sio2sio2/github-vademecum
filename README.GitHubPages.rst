Documentación con Sphinx
========================
Esta guía secundaria desarrolla cómo mantener un repositorio que contiene
exclusivamente documentación escrita en `Sphinx <https://www.sphinx-doc.org/>`_
que se publica gracias a las `GitHub Pages <https://pages.github.com/>`_.

La idea es que la documentación ocupe la rama principal (*main*) del
repositorio, mientras que el HTML generado ocupará una rama adicional
(*gh-pages*). Este HTML no forma parte del contenido que se sincroniza con el
comando ``git push``, sino que cada vez que se lleva a cabo esta sincronización,
se usan las capacidades de las `Github Actions
<https://github.com/features/actions>`_ para generarlas en remoto y que ocupan
la rama *gh-pages*.

Requisitos previos
------------------
#. Un *Personal Access Token* con permisos para *workflow*.
#. Un repositorio en que se permita que los trabajos definidos en un *workflow*
   tengan permisos de lectura y escritura (por defecto no tienen permisos de
   escritura). Para ello:

   .. code-block:: none

      Settings>Actions>General>Workflow permission>Read and write permissions

#. Una documentación escrita en Sphinx_ que almacene las fuentes en el
   subdirectorio ``sources`` y los documentos generados bajo ``docs``:

   .. code-block:: none

      repositorio
         +-- sources (fuentes rst)
         +-- docs
         |     +-- html (HTML generado)
         +-- Makefile
         +-- .gitignore
         +-- .git/
         +-- .github/

#. Un archivo ``.gitignore`` que contenga al menos la ruta ``docs/`` a fin de
   que los documentos generados no se sincronicen con el repositorio remoto. De
   este modo podremos generar en local la documentación y comprobar que queda a
   nuestro gusto, pero sin que suba a la rama *main* del repositorio de GitHub.

Configuración
-------------
#. Crear un archivo ``.github/workflows/docs.yaml`` con el siguiente
   contenido:

   .. code-block:: yaml

      name: Construir y desplegar el manual

      on:
        push:
          branches: [ main ]

      jobs:
        build-and-deploy:
          runs-on: ubuntu-latest
          steps:
            - name: Obtención de la documentación
              uses: actions/checkout@v4

            - name: Configurar Python
              uses: actions/setup-python@v5
              with:
                python-version: '3.x'

            - name: Instalar de dependencias.
              run: |
                pip install -r requirements.txt

            - name: Construir documentación 
              run: |
                make html
                touch docs/html/.nojekyll

            - name: Desplegarla en GitHub Page
              uses: peaceiris/actions-gh-pages@v3
              with:
                github_token: ${{ secrets.GITHUB_TOKEN }}
                publish_dir: ./docs/html

            - name: Notificación
              if: success()
              uses: actions/github-script@v6
              with:
                script: |
                  github.rest.repos.createCommitComment({
                    owner: context.repo.owner,
                    repo: context.repo.repo,
                    commit_sha: "${{ github.sha }}",
                    body: "¡Doc actualizada: https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }}/"
                  })

   Este documento define un *workflow* que se ejecuta al realizar un ``git push`` y
   se compone de varios trabajos que se desarrollan en un *runner* (contenedor) con
   Ubuntu:

   *Obtención de la documentación*
      Copia las nuestro archivos fuente en el *runner*.

   *Configurar Python*
      Prepara la última versión disponible de Python en el *runner*.

   *Instalar dependencias*
      Como su propio nombre indica descarga e instala las dependencias necesarias
      para poder generar la versión HTML de la documentación. Dado que todas las
      dependencias son paquetes de pip, debe crearse en el repositorio un archivo
      ``requirements.txt`` que las desglose.

   *Construir documentación*
      Genera la versión HTML de la documentación y crea el archivo `.nojekyll`
      necesario para que se puedan ver correctamente las páginas en `GitHub Pages`_.

   *Despliegue de la documentación*
      Se copia el contenido de ``docs/html`` en la rama *gh-pages* del
      repositorio. La rama no es necesario crearla de antemano, puesto que de no
      existir se creará automáticamente.
      
   *Notificación* (Opcional)
      Se añade un comentario automático al *commit*.

#. Crear un archivo ``requirements.txt`` con los paquetes de pip necesarios para
   generar la documentación que básicamente serán los que instalen el propio
   *sphinx* y las extensiones o temas adicionales. Por ejemplo:

   .. code-block:: none

      sphinx
      sphinx-book-theme
      sphinx-copybutton
      sphinx-togglebutton

#. Generar la documentación en local (``make html``) para comprobar que se
   genera correctamente y según deseamos y, hecha esta comprobación, subir los
   cambios al repositorio remoto con:

   .. code-block:: console

      $ git push

#. La primera vez que completemos todo deberemos además configurar en GitHub las
   `GitHub Pages`_ para que se muestre el directorio ``/`` del repositorio
   *gh-pages*:

   .. code-block:: none

      Settings>Pages
