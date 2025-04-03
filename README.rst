Vademécum
*********
Un *repositorio Git* es un conjunto estructurado de archivos (un proyecto, si se
prefiere) del que se conserva su historial de cambios (*commits*), de manera que
se puede restaurar uno o hacer comparaciones entre ellos. Este historial,
además, no es líneal, sino que puede bifurcarse en ramas que divergen o incluso
que llegado el caso acaban convergiendo después:

.. _figura 1:

.. image:: files/git.svg

Repositorio
===========
En la `figura 1`_ los boliches representan versiones del
proyecto que conserva el repositorio. Estas versiones se crean al realizar una
confirmación de cambios (*commit*) y se identifican por un *hash* (`af78aeb1`,
`e5c112ea`, etc).

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

**HEAD**
   Es el puntero que señala la versión (*commit*) activa del proyecto. Esto
   quiero decir que, si consultamos los archivos, nos encontraremos con los
   contenidos que tenían en esa versión.

**Branch** (rama)
   Es un puntero (una referencia dinámica) a un *commit*. Es dinámico en la
   medida en que, si estamos en una rama (o lo que es lo mismo, el *HEAD* apunta
   al mismo *commit* que dicha rama), al hacer cambios y generar un nuevo
   *commit*, tanto el *HEAD* como la rama apuntarán a ese nuevo *commit*. En el
   ejemplo, la rama activa es **main**, ya que el *commit* activo es `674e3172`.
   Si hacemos algumos cambios y generamos un nuevo *commit* `f932eb8`, tanto
   *HEAD* como **main** apuntarán automáticamente a este nuevo *commit*:

   .. _figura_2:

   .. image:: files/git2.svg

Como un *commit* debe tener un padre a partir del cual se originó, Git permite
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
referenciados de algún modo para que Git no los acabe desechando. Los padres
siempre están referenciados por los hijos, como acabamos de ver, por lo que en
realidad el problema podría estar en los *commits* terminales si no les hemos
asociado ninguna rama. Por ejemplo, en el gráfico, `ddbaebaa` es un nodo
terminal y está referenciado por la rama `experimental`, por lo que no se
perderá. Sin embargo, si no estuviera referenciado por esta rama, no tendría
referencia ninguna y a la postre Git lo desecharía a él y a `7172f279` y
`558302c9`.

Flujo de trabajo
================
