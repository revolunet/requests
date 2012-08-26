Requests: HTTP pour les Humains
===============================


.. image:: https://secure.travis-ci.org/kennethreitz/requests.png?branch=develop
        :target: https://secure.travis-ci.org/kennethreitz/requests

Requests est une librairie sous licence ISC, écrite en Python, pour les être humains.

La plupart des modules Python existants pour envoyer des requêtes HTTP sont
lourds et très verbeux. Le module Python intégré urllib2 fournit la plupart
des fonctionnalités HTTP dont nous avons besoin, mais l'API est complètement
moisie. Cela demande beaucoup trop de travail (voir même des écrasements de
méthodes) pour réaliser les tâches plus simples.

Cela ne devrait pas se passer comme ça. Pas en Python.

::

    >>> r = requests.get('https://api.github.com', auth=('user', 'pass'))
    >>> r.status_code
    204
    >>> r.headers['content-type']
    'application/json'
    >>> r.text
    ...

Voir `le même code, sans Requests <https://gist.github.com/973705>`_.

Requests vous permet d'envoyer des requêtes HTTP/1.1. Vous pouvez ajouter des
en-têtes, des données de formulaire, des fichiers multipart, et des paramêtres
avec de simples dictionnaires Python, et accéder aux données de la réponse de
la même manière. Requests est basé sur httplib et
`urllib3 <https://github.com/shazow/urllib3>`_, mais fait tout le travail
compliqué et les bidouilles improbables pour vous.

Fonctionnalités
---------------

- Gestion domaines et URLS internationales
- Keep-Alive & Groupement de connections (Pooling)
- Sessions et Cookies persistants
- Verification SSL
- Authentifications Basic/Digest ou personnalisées
- Gestion élégante des Cookies clé/valeur
- Décompression automatique
- Corps des réponses en unicode
- Upload de fichiers multiparts
- Timeouts de connexion
- supprt de ``.netrc``
- Thread-safe.


Installation
------------

Pour installer requests, il suffit de lancer: ::

    $ pip install requests

Où, si vraiment c'est obligé: ::

    $ easy_install requests

Mais ça, vous ne devriez pas.



Contriber
---------

#. Vérifiez les tickets ouvertes ou ouvrez-en, un nouveau pour commencer une discussion autour d'une idée de fonctionnalité ou d'un bug. Il y a un tag 'Contributor Friendly' pour les issues idéales pour ceux qui ne sont pas encore trop familiers avec le codebase.
#. Forkez le `repository GitHub`_ pour commencer à faire vos changements sur la branche **develop** (ou une branche qui en est issue).
#. Ecrivez un test qui montre que le bug a été fixé ou que la fonction marche bien comme prévue.
#. Envoyez une pull request et insistez auprès du mainteneur jusqu'à ce qu'il merge et publie. :) Ajoutez vous au fichier AUTHORS_.

.. _`repository GitHub`: http://github.com/kennethreitz/requests
.. _AUTHORS: https://github.com/kennethreitz/requests/blob/develop/AUTHORS.rst
