.. _install:

Installation
============

Cette partie de la documentation traite de l'installation de Requests.
La première étape pour utiliser une librairie est de l'installer correctement.


Distribute & Pip
----------------

Requests s'installe simplement avec `pip <http://www.pip-installer.org/>`_::

    $ pip install requests

ou, avec `easy_install <http://pypi.python.org/pypi/setuptools>`_::

    $ easy_install requests

Mais, ce n'est `pas conseillé <http://www.pip-installer.org/en/latest/other-tools.html#pip-compared-to-easy-install>`_.


Miroir Cheeseshop
------------------

Si Cheeseshop n'est pas accessible, vous pouvez également installer Requests 
depuis le  `miroir personnel Cheeseshop <http://pip.kennethreitz.com/>`_ de 
Kenneth Reitz:: 

    $ pip install -i http://pip.kennethreitz.com/simple requests


Obtenir le code
---------------

Requests est activement développé sur GitHub, ou le code est
`toujours disponible <https://github.com/kennethreitz/requests>`_.

Vous pouvez cloner le dépot public::

    git clone git://github.com/kennethreitz/requests.git

Télécharger le `tarball <https://github.com/kennethreitz/requests/tarball/master>`_::

    $ curl -OL https://github.com/kennethreitz/requests/tarball/master

Ou, télécharger le `zipball <https://github.com/kennethreitz/requests/zipball/master>`_::

    $ curl -OL https://github.com/kennethreitz/requests/zipball/master


Une fois que vous avez une copie de la source, vous pouvez l'intégrer dans votre package
Python, ou l'installer facilement dans votre dossier site-packages::

    $ python setup.py install
