.. _quickstart:

Quickstart
==========

.. module:: requests.models

Impatient de commencer? Cette page vous donne une bonne introduction pour démarrer
avec Requests. Ceci implique que vous ayez déja Requests installé. Si ce n'est pas
la cas, suivez la section  :ref:`Installation <install>`.

Premièrement, vérifier que:

* Requests est :ref:`installé <install>`
* Requests est :ref:`à jour <updates>`


Commençons avec des exemples et des cas simples.


Créer une requête
-----------------

Créer une requête standard avec Request est très simple.

Commençons par import le module Requests::

    >>> import requests

Maintenant, essayons de récupérer une page web. Pour cette exemple, récupérons
la timeline publique de github::

    >>> r = requests.get('https://github.com/timeline.json')

Nous récupérons alors un objet :class:`Response` appelé ``r``. Celui-ci contient
toutes les informations dont nous avons besoin.

L'API simple de Requests permet d'effectuer toute sorte de requête HTTP très 
simplement. Par exemple, pour faire une requete HTTP POST::

    >>> r = requests.post("http://httpbin.org/post")

Pratique, non? 

Et pour les autres types de requêtes: PUT, DELETE, HEAD et OPTIONS ?

C'est tout aussi simple::

    >>> r = requests.put("http://httpbin.org/put")
    >>> r = requests.delete("http://httpbin.org/delete")
    >>> r = requests.head("http://httpbin.org/get")
    >>> r = requests.options("http://httpbin.org/get")

Jusqu'ici tout va bien, et c'est juste un petit aperçu de ce que Requests peut faire.


Passer des paramètres dans les URLs
-----------------------------------

Il est fréquent d'avoir besoin de passer des données dans les URLs sous forme
de paramètres. En construisant l'URL à la main, ces données devraient être 
fournies sous forme de clé/valeur dans l'URL après un point d'interrogation,
par exemple ``httpbin.org/get?key=val``. Requests vous permet de fournir ces
arguments sous forme de dictionnaire, en utilisant l'argument ``params``. Par
exemple, si vous souhaitez passer ``key1=value1`` et ``key2=value2`` à
``httpbin.org/get``, vous pouvez utiliser le code suivant::

    >>> payload = {'key1': 'value1', 'key2': 'value2'}
    >>> r = requests.get("http://httpbin.org/get", params=payload)

Vous pouvez constater que l'URL a été correctement encodée en imprimant l'URL::

    >>> print r.url
    u'http://httpbin.org/get?key2=value2&key1=value1'


Contenu de la réponse
---------------------

Nous pouvons lire le contenu de la réponse du serveur. Pour reprendre l'exemple
de la timeline GitHub::

    >>> import requests
    >>> r = requests.get('https://github.com/timeline.json')
    >>> r.text
    '[{"repository":{"open_issues":0,"url":"https://github.com/...

Lorsque vous effectuez une requête, Requests devine l'encodage de la réponse en
fonction des en-têtes HTTP. Le texte décodé selon cet encodage est alors
accesible via ``r.text``. Pour consulter l'encoding que Requests a utilisé, et
le modifier, utilisez la propriété ``r.encoding``::

    >>> r.encoding
    'utf-8'
    >>> r.encoding = 'ISO-8859-1'

Lorsque vous modifiez cette propriété, Requests l'utilise la nouvelle valeur
quand vous accédez à ``r.text``.

Requests sait également utiliser des encodages personnalisés si jamais vous en
avez besoin. Si vous avez crée votre propre encodage et l'avez enregistré avec
le module ``codecs``, utilisez simplement le nom du codec comme valeur de
``r.encoding`` et Requests gérera le décodage pour vous.


Réponse binaire
---------------

Pour les requêtes non-texte (fichiers binaires), vous pouvez accéder au 
contenu de la réponse sous forme de bytes::

    >>> r.content
    b'[{"repository":{"open_issues":0,"url":"https://github.com/...

Les réponse avec l'en-tête transfer-encoding à ``gzip`` et ``deflate`` sont 
automatiquement décodés pour vous.

Par exemple, pour créer une image à partir de données recues par une requête, vous
pouvez utiliser le code suivant:

    >>> from PIL import Image
    >>> from StringIO import StringIO
    >>> i = Image.open(StringIO(r.content))


Réponse JSON
------------

Si vous devez travailler avec des données JSON, Requests dispose dun décodeur intégré::

    >>> import requests
    >>> r = requests.get('https://github.com/timeline.json')
    >>> r.json
    [{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...

Si jamais le décodage échoue, ``r.json`` renverra simplement ``None``.


Réponse brute
-------------

Dans de rares cas, si vous avez besoin d'accéder au contenu brut de la 
réponse du serveur, vous pouvez y accéder directement via ``r.raw``::

    >>> r.raw
    <requests.packages.urllib3.response.HTTPResponse object at 0x101194810>

    >>> r.raw.read(10)
    '\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'


En-têtes personnalisées
-----------------------

Si vous souhaitez ajouter des headers HTTP à une requête, passez simpement un
object de type ``dict`` au pramètre ``headers``.

Par exemple, pour spécifier un content-type dans l'exemple précédent::

    >>> import json
    >>> url = 'https://api.github.com/some/endpoint'
    >>> payload = {'some': 'data'}
    >>> headers = {'content-type': 'application/json'}


Requêtes POST avancées
----------------------

Typiquement, vous avec besoin d'envoyer des données encodées comme
par exemple un formulaire HTML. Pour cela, on passe simplement un
dictionnaire avec l'argument `data`. Votre dictionnaire de données
sera automatiquement encodé comme un formulaire au moment de la requête::

    >>> payload = {'key1': 'value1', 'key2': 'value2'}
    >>> r = requests.post("http://httpbin.org/post", data=payload)
    >>> print r.text
    {
      // ...snip... //
      "form": {
        "key2": "value2",
        "key1": "value1"
      },
      // ...snip... //
    }

Dans certains cas, vous ne souhaitez pas que les données soit encodées. 
Si vous passez une chaîne de caractères ``string`` à la place d'un objet 
``dict``, les données seront postées directement.

Par exemple, l'API GitHub v3 accepte les requêtes POST/PATCH avec des données 
JSON::

    >>> import json
    >>> url = 'https://api.github.com/some/endpoint'
    >>> payload = {'some': 'data'}

    >>> r = requests.post(url, data=json.dumps(payload))


POST de fichiers Multipart
--------------------------

Requests simplifie l'upload de fichiers encodés en MultiPart::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': open('report.xls', 'rb')}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      // ...snip... //
      "files": {
        "file": "<censored...binary...data>"
      },
      // ...snip... //
    }

Pour forcer le nom du fichier explicitement::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': ('report.xls', open('report.xls', 'rb'))}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      // ...snip... //
      "files": {
        "file": "<censored...binary...data>"
      },
      // ...snip... //
    }

Vous pouvez également envoyer des chaînes de caractères en tant que fichier ::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      // ...snip... //
      "files": {
        "file": "some,data,to,send\\nanother,row,to,send\\n"
      },
      // ...snip... //
    }


Codes de retour des réponses (status)
-------------------------------------

Nous pouvons vérifier le code de retour d'une réponse::

    >>> r = requests.get('http://httpbin.org/get')
    >>> r.status_code
    200

Requests fournit également un code de statut interne pour faciliter
les vérifications :

    >>> r.status_code == requests.codes.ok
    True

Si nous faisons une mauvaise requête (code de retour autre que 200), nous
pouvons lever une exception avec :class:`Response.raise_for_status()`::

    >>> bad_r = requests.get('http://httpbin.org/status/404')
    >>> bad_r.status_code
    404

    >>> bad_r.raise_for_status()
    Traceback (most recent call last):
      File "requests/models.py", line 832, in raise_for_status
        raise http_error
    requests.exceptions.HTTPError: 404 Client Error


Mais comme notre ``status_code`` pour ``r`` était ``200``, lorsque l'on
appele ``raise_for_status()`` nous obtenons::

    >>> r.raise_for_status()
    None

Tout va bien.


En-têtes des réponses
---------------------

On peut accéder aux en-têtes HTTP (headers) de la réponse du serveur via
une simple dictionnaire Python::

    >>> r.headers
    {
        'status': '200 OK',
        'content-encoding': 'gzip',
        'transfer-encoding': 'chunked',
        'connection': 'close',
        'server': 'nginx/1.0.4',
        'x-runtime': '148ms',
        'etag': '"e1ca502697e5c9317743dc078f67693f"',
        'content-type': 'application/json; charset=utf-8'
    }

Ce dictionnaire est cependant particulier : Il est spécifique aux en-têtes HTTP.
En effet, selon la `RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html>`_, 
les en-têtes HTTP ne doivent pas être sensibles à la casse.

Donc, nous pouvons accéder aux en-têtes quelque soit la casse utilisée::

    >>> r.headers['Content-Type']
    'application/json; charset=utf-8'

    >>> r.headers.get('content-type')
    'application/json; charset=utf-8'

Si l'en-tête n'existe pas dans la Response, la valeur par défaut est ``None``::

    >>> r.headers['X-Random']
    None


Cookies
-------

Si la résponse contient des Cookies, vous pouvez y accéder rapidement::

    >>> url = 'http://httpbin.org/cookies/set/requests-is/formidable'
    >>> r = requests.get(url)

    >>> r.cookies['requests-is']
    'formidable'

Pour envoyer vos propres cookies au serveur, vous pouvez utiliser le
paramètre ``cookies``::

    >>> url = 'http://httpbin.org/cookies'
    >>> cookies = dict(cookies_are='working')

    >>> r = requests.get(url, cookies=cookies)
    >>> r.text
    '{"cookies": {"cookies_are": "working"}}'


Authentification basique
------------------------

La plupart des services web nécessitent une authentification. Il y a
différents types d'authentification, mais la plus commune est 
l'authentification HTTP basique.

Utiliser l'authentification basique avec Requests est extrêmement simple::

    >>> from requests.auth import HTTPBasicAuth
    >>> requests.get('https://api.github.com/user', auth=HTTPBasicAuth('user', 'pass'))
    <Response [200]>

Comme l'authentification HTTP basique est le standard le plus répandu, Requests 
fournit un raccourci pour cette méthode d'authentification::

    >>> requests.get('https://api.github.com/user', auth=('user', 'pass'))
    <Response [200]>

Fournir de cette manière un tuple d'authentification au paramètre `auth` 
équivaut à utiliser l'exemple ``HTTPBasicAuth`` ci-dessus.


Authentification Digest 
-----------------------

Une autre forme populaire de protection des web services est l'authentification Digest::

    >>> from requests.auth import HTTPDigestAuth
    >>> url = 'http://httpbin.org/digest-auth/auth/user/pass'
    >>> requests.get(url, auth=HTTPDigestAuth('user', 'pass'))
    <Response [200]>


Authentification OAuth
----------------------

Le projet `requests-oauth <http://pypi.python.org/pypi/requests-oauth>`_ de Miguel Araujo fournit une interface
simple pour établir des connexions OAuth. La documentation et des exemples peuvent être trouvées sur `git repository <https://github.com/maraujop/requests-oauth>`_.


Redirections et Historique
--------------------------

Requests effectue automatiquement les redirections lorsque vous utilisez les
méthodes GET et OPTIONS.

GutHub par exemple redirige tout le traffic HTTP vers HTTPS. Nous pouvons
utiliser la méthode ``history`` de l'object Response pour tracker les
redirections. Regardons ce qu'il se passe pour Github::

    >>> r = requests.get('http://github.com')
    >>> r.url
    'https://github.com/'
    >>> r.status_code
    200
    >>> r.history
    [<Response [301]>]

La liste :class:`Response.history` contient la liste des objets 
:class:`Request` qui ont été crées pour compléter la requête. Cette liste est triée de la plus ancienne à la plus récente.

Si vous utilisez les methodes GET ou OPTIONS, vous pouvez désactiver la 
gestion des redirections avec le paramètre ``allow_redirects``::

    >>> r = requests.get('http://github.com', allow_redirects=False)
    >>> r.status_code
    301
    >>> r.history
    []

Si vous utilisez POST, PUT, PATCH, DELETE ou HEAD vous pouvez également
autoriser explicitement les redirections::

    >>> r = requests.post('http://github.com', allow_redirects=True)
    >>> r.url
    'https://github.com/'
    >>> r.history
    [<Response [301]>]


Timeouts
--------

Vous pouvez demander à Requests d'arrêter d'attendre après un certains nombre de secondes 
avec le paramètre ``timeout``::

    >>> requests.get('http://github.com', timeout=0.001)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    requests.exceptions.Timeout: Request timed out.

.. admonition:: Note:

    ``timeout`` affecte uniquement le délai de connection, pas le temps de téléchargment
    des réponses.


Erreurs et Exceptions
---------------------

Dans le cas de problèmes de réseau (e.g. erreurs DNS, connexions refusées, etc),
Requests lévera une exception :class:`ConnectionError`.

Dans les rares cas de réponses HTTP invalides, Requests lévera une exception 
:class:`HTTPError`.

Si une requête dépasse le temps d'attente, une exception :class:`Timeout` est levée.

Si une requête dépasse le nombre maximum de redirections possibles configuré,
une exception :class:`TooManyRedirects` est levée.

Toutes les exceptions levées par Requests héritent de :class:`requests.exceptions.RequestException`.

Vous pouvez vous réferer à :ref:`Configuration API Docs <configurations>` si vous souhaitez toujours lever 
des exceptions :class:`HTTPError` avec l'option ``danger_mode``, ou laisser Requests attraper la majorité 
des :class:`requests.exceptions.RequestException` avec l'option ``safe_mode``.

-----------------------

Prêt pour aller plus loin ? Visitez la section :ref:`avancée <advanced>`.
