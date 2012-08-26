.. _advanced:

Utilisation avancée
===================

Ce document traite de quelques fonctionnalités avancées de Requests.


Objets Session
--------------

L'objet Session vous permet de conserver des paramètres entre plusieurs
requêtes. Il permet également de conserver les cookies entre toutes les 
requêtes de la même instance Session.

Un objet Session a toutes les methodes de l'API Requests principale.

Pour conserver des cookies entre les requêtes::

    s = requests.session()

    s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
    r = s.get("http://httpbin.org/cookies")

    print r.text
    # '{"cookies": {"sessioncookie": "123456789"}}'


Les Sessions peuvent aussi être utilisées pour fournir des valeurs par défaut
aux requêtes::

    headers = {'x-test': 'true'}
    auth = ('user', 'pass')

    with requests.session(auth=auth, headers=headers) as c:

        # 'x-test' et 'x-test2' sont envoyés
        c.get('http://httpbin.org/headers', headers={'x-test2': 'true'})


Tous les dictionnaires que vous passez aux methodes de requête sont fusionnés
avec les valeur de session déja définies. Les paramètres de la méthode 
surchargent les paramètres de session.

.. admonition:: Supprimer une valeur d'un paramètre

    Parfois vous voudrez supprimer des paramètres de session lors de vos
    requêtes. Pour cela, il suffit d'envoyer lors de l'appel de la méthode
    un dictionnaire dont les clés seraient les paramètres a supprimer et les
    valeurs seraient ``None``. Ces paramètres seront alors automatiquement
    omis.

Toutes les valeurs contenues dans la session sont directement accessibles.
Pour en savoir plus, cf :ref:`Session API Docs <sessionapi>`.


Objets Request et Response
--------------------------

Lorsqu'un appel à requests.* est effectué, vous faites deux choses. Premièrement,
vous construisez un object ``Request`` qui va être envoyé au serveur pour récupérer
ou intérroger des ressources. Dès que l'objet ``requests`` reçoit une réponse du
serveur, un object de type ``Response`` est généré. L'objet ``Response``contient
toutes les informations retournées par le serveur mais aussi l'object ``Request``
que vous avz crée initialement. Voici une requête simple pour obtenir des
informations depuis les serveurs Wikipedia::

    >>> response = requests.get('http://en.wikipedia.org/wiki/Monty_Python')

Si nous voulons accéder aux en-têtes renvoyées par le serveur, nous faisons::

    >>> response.headers
    {'content-length': '56170', 'x-content-type-options': 'nosniff', 'x-cache':
    'HIT from cp1006.eqiad.wmnet, MISS from cp1010.eqiad.wmnet', 'content-encoding':
    'gzip', 'age': '3080', 'content-language': 'en', 'vary': 'Accept-Encoding,Cookie',
    'server': 'Apache', 'last-modified': 'Wed, 13 Jun 2012 01:33:50 GMT',
    'connection': 'close', 'cache-control': 'private, s-maxage=0, max-age=0,
    must-revalidate', 'date': 'Thu, 14 Jun 2012 12:59:39 GMT', 'content-type':
    'text/html; charset=UTF-8', 'x-cache-lookup': 'HIT from cp1006.eqiad.wmnet:3128,
    MISS from cp1010.eqiad.wmnet:80'}

Toutefois, si nous souhaitons récupérer les en-têtes que nous avions envoyées au
serveur, nous accédons simplement à la requête, et aux en-têtes de la requête::

    >>> response.request.headers
    {'Accept-Encoding': 'identity, deflate, compress, gzip',
    'Accept': '*/*', 'User-Agent': 'python-requests/0.13.1'}


Verifications certificats SSL
-----------------------------

Requests peut vérifier les certificats SSL sur les requêtes HTTPS, comme n'importe quel navigateur web. Pour vérifier le certificat d'un serveur, vous pouvez utiliser l'argument ``verify``::

    >>> requests.get('https://kennethreitz.com', verify=True)
    requests.exceptions.SSLError: hostname 'kennethreitz.com' doesn't match either of '*.herokuapp.com', 'herokuapp.com'

SSL n'est pas configuré sur ce domaine, donc cela génère une erreur. Parfait. Par contre, GitHub en a un::

    >>> requests.get('https://github.com', verify=True)
    <Response [200]>

Vous pouvez aussi passer au paramètre ``verify`` le chemin vers un fichier ``CA_BUNDLE`` pour les certificats privés. Vous pouvez également définir la variable d'environnement ``REQUESTS_CA_BUNDLE``.

Requests can also ignore verifying the SSL certficate if you set ``verify`` to False.

::

    >>> requests.get('https://kennethreitz.com', verify=False)
    <Response [200]>

By default, ``verify`` is set to True. Option ``verify`` only applies to host certs.

You can also specify the local cert file either as a path or key value pair::

    >>> requests.get('https://kennethreitz.com', cert=('/path/server.crt', '/path/key'))
    <Response [200]>

If you specify a wrong path or an invalid cert::

    >>> requests.get('https://kennethreitz.com', cert='/wrong_path/server.pem')
    SSLError: [Errno 336265225] _ssl.c:347: error:140B0009:SSL routines:SSL_CTX_use_PrivateKey_file:PEM lib


Workflow du contenu des réponses
--------------------------------

Par défaut, lorsque vous effectuez une requête, le corps de la réponse n'est pas téléchargé automatiquement. Les en-têtes sont téléchargés, mais le contenu lui-même n'est téléchargé que lorsque vous accédez à l'attribut  :class:`Response.content`.

Exemple::

    tarball_url = 'https://github.com/kennethreitz/requests/tarball/master'
    r = requests.get(tarball_url)


La requête a été effectuée, et la connexion est toujours ouverte. Le corps de la réponse n'est pas encore été téléchargé.::

    r.content

Le contenu est téléchargé et mis en cache à ce moment-là.

Vous pouvez modifier ce comportement par défaut avec le paramètre ``prefetch``::

    r = requests.get(tarball_url, prefetch=True)
    # Appel bloquant jusqu'à reception du corps de la réponse


Configurer Requests
--------------------

Vous pouvez avoir envie de configurer une requête pour personnaliser son comportement.
Pour faire cela vous pouvez passer un dictionnaire ``config`` à une requête ou une session.
Pour en savoir plus, cf :ref:`Configuration API Docs <configurations>` to learn more.


Keep-Alive
----------

Bonne nouvelle - grâce à urllib3, le keep-alive est 100% automatique pendant une session! Toutes les requêtes que vous ferez à travers une session réutiliseront automatiquement la connexion appropriée!

A noter que les connexions ne sont libérées pour réutilisation seulement lorsque les données ont été lues. Faites attention à bien mettre ``prefetch`` à ``True`` ou toujours accéder à la propriété ``content`` de l'object ``Response``.

Si vous souhaitez désactiver le keep-alive, vous pouvez définir l'attribut de configuration ``keep_alive`` à ``False``::

    s = requests.session()
    s.config['keep_alive'] = False


Requêtes asynchrones
--------------------

``requests.async`` a été supprimé de requests et dispose maintenant de son propre repository nommé `GRequests <https://github.com/kennethreitz/grequests>`_.


Hooks d'évenements
------------------

Requests dispose d'un système de 'hooks' que vous pouvez utiliser pour
manipuler des portions du processus de requêtage ou signaler des évènements.

Hooks disponibles:

``args``:
    Un dictionnaire d'arguments prêts à être envoyés à Request().

``pre_request``:
    L'objet Request, juste avant d'être envoyé.

``post_request``:
    L'objet Request, juste après avoir été envoyé.

``response``:
    La réponse générée après une requête.


Vous pouvez assigner une fonction de hook par requête, en passant au 
paramètre ``hooks`` de la Request un dictionnaire de hooks 
``{hook_name: callback_function}``::

    hooks=dict(args=print_url)

La fonction ``callback_function`` recevra un bloc de données en premier 
argument.

::

    def print_url(args):
        print args['url']

Si une exception apparait lors de l'éxecution du callback, un warning est
affiché.

Si le callback renvoie une valeur, on suppose que cela remplace les données
qui lui ont été passées. Si la fonction ne renvoie rien, alors rien n'est
affecté.

Affichons quelques arguments a la volée::

    >>> requests.get('http://httpbin.org', hooks=dict(args=print_url))
    http://httpbin.org
    <Response [200]>

Cette fois-ci, modifions les arguments avec un nouveau callback::

    def hack_headers(args):
        if args.get('headers') is None:
            args['headers'] = dict()

        args['headers'].update({'X-Testing': 'True'})

        return args

    hooks = dict(args=hack_headers)
    headers = dict(yo=dawg)

Et essayons::

    >>> requests.get('http://httpbin.org/headers', hooks=hooks, headers=headers)
    {
        "headers": {
            "Content-Length": "",
            "Accept-Encoding": "gzip",
            "Yo": "dawg",
            "X-Forwarded-For": "::ffff:24.127.96.129",
            "Connection": "close",
            "User-Agent": "python-requests.org",
            "Host": "httpbin.org",
            "X-Testing": "True",
            "X-Forwarded-Protocol": "",
            "Content-Type": ""
        }
    }


Authentification personnalisée
------------------------------

Requests vous permet de spécifier vos propres mécanismes d'authentification.

N'importe quel 'callable' à qui l'on passe l'argument ``auth`` pour une méthode
de requête a l'opportunité de modifier la requête avant de la dispatcher.

Les implémentations d'authentification doivent hériter de la classe
``requests.auth.AuthBase``, et sont très faciles à définir. Request fournit
deux modèles communs d'authentification dans ``requests.auth``: ``HTTPBasicAuth``
et ``HTTPDigestAuth``.

Admettons que nous ayons un webservice qui réponde uniquement si le header ``X-Pizza``
est présent et défini avec un certain mot de passe. Peu de chance que cela arrive,
mais voyons voir ce que cela pourrait donner.

::

    from requests.auth import AuthBase
    class PizzaAuth(AuthBase):
        """Attache l'authentification HTTP Pizza à un object Request."""
        def __init__(self, username):
            # setup any auth-related data here
            self.username = username

        def __call__(self, r):
            # modify and return the request
            r.headers['X-Pizza'] = self.username
            return r

On peut alors faire une requête qui utilise notre authentification Pizza::

    >>> requests.get('http://pizzabin.org/admin', auth=PizzaAuth('kenneth'))
    <Response [200]>


Requête en streaming
--------------------

Avec la méthode ``requests.Response.iter_lines()`` vous pouvez facilement itérer sur des
API en streaming comme par exemple la `Twitter Streaming API <https://dev.twitter.com/docs/streaming-api>`_.

Pour utiliser la Twitter Streaming API et pister le mot-clé "requests"::

    import requests
    import json

    r = requests.post('https://stream.twitter.com/1/statuses/filter.json',
        data={'track': 'requests'}, auth=('username', 'password'))

    for line in r.iter_lines():
        if line: # filtre les lignes vides (keep-alive)
            print json.loads(line)


Logging verbeux
---------------

Si vous voulez avoir une bonne vision des requêtes HTTP qui sont envoyées
par votre application, vous pouvez activer le logging verbeux.

Pour cela, configurez Requests avec un stream où ecrire les logs::

    >>> my_config = {'verbose': sys.stderr}
    >>> requests.get('http://httpbin.org/headers', config=my_config)
    2011-08-17T03:04:23.380175   GET   http://httpbin.org/headers
    <Response [200]>


Proxys
-------

Si vous avez besoin d'utiliser un proxy, vous pouvez configurer individuellement
les requêtes avec l'argument ``proxies`` dans toutes les méthodes::

    import requests

    proxies = {
      "http": "10.10.1.10:3128",
      "https": "10.10.1.10:1080",
    }

    requests.get("http://example.org", proxies=proxies)

Vous pouvez aussi définir des proxys avec les variables d'environnement
``HTTP_PROXY`` et ``HTTPS_PROXY``.

::

    $ export HTTP_PROXY="10.10.1.10:3128"
    $ export HTTPS_PROXY="10.10.1.10:1080"
    $ python
    >>> import requests
    >>> requests.get("http://example.org")

To use HTTP Basic Auth with your proxy, use the `http://user:password@host/` syntax::

    proxies = {
        "http": "http://user:pass@10.10.1.10:3128/",
    }


Compatibilité
----------

Requests est destiné à être conforme avec toutes les spécifications et RFC
pertinentes, tant que cela ne cause pas de difficultés pour l'utilisateur.
Cette attention aux spécifications peut mener à des comportements qui 
peuvent sembler inhabituels pour ceux qui n'en sont pas familiers.

Encodages
^^^^^^^^^

Lorsque vous recevez une réponse, Requests devine l'encodage à utiliser pour 
décoder la réponse quand vous accéder à ``Response.text``. Requests commence
par vérifier l'encodage dans l'en-tête HTTP, et si aucun n'est présent,
Request utilisera le module `chardet <http://pypi.python.org/pypi/chardet>`_
pour tenter de deviner l'encodage.

Le seul cas ou Requests ne suivra pas cette méthode est quand l'en-tête charset
n'est pas présent et l'en-tête ``Content-Type`` contient ``text``. Dans ce cas,
la `RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.7.1>`_
spécifie que le jeu de caractères par défaut doit être ``ISO-8859-1``. Requests
suit donc les spécifications dans ce cas. Si vous avez besoin d'un encodage
différent, vous pouvez définir manuellement la propriété ``Response.encoding``
ou utiliser la réponse brute avec ``Request.content``.


Methodes (verbes) HTTP
----------------------

Requests fournit l'accès à toute la gamme des verbes HTTP: GET, OPTIONS,
HEAD, POST, PUT, PATCH et DELETE. Vous trouverez ci dessous divers exemples
d'utilisation de ces verbes avec Requests, en utilisant l'API GitHub.

Nous commençons avec les verbes les plus utilisé : GET. La methode HTTP GET est
une méthode idempotente qui retourne une ressource pour une URL donnée. C'est
donc ce verbe que vous allez utiliser pour tenter de récupérer des données
depuis le web. Un exemple d'usage serait de récupérer les informations d'un
commit spécifique sur GitHub. Admettons que nous souhaitions récupérer le
commit ``a050faf`` de Requests. On peut le récupérer de cette façon::

    >>> import requests
    >>> r = requests.get('https://api.github.com/repos/kennethreitz/requests/git/commits/a050faf084662f3a352dd1a941f2c7c9f886d4ad')

On devrait confirmer que GitHub a répondu correctement. Si c'est le cas on peut
alors travailler avec le contenu reçu. Voici comment faire::

    >>> if (r.status_code == requests.codes.ok):
    ...     print r.headers['content-type']
    ...
    application/json; charset=utf-8

Donc, GitHub renvoie du JSON. C'est super, on peut alors utiliser le module
JSON pour convertir le résultat en object Python. Comme GitHub renvoie de
l'UTF-8, nous devons accéder à ``r.text`` et pas ``r.content``. ``r.content``
renvoie un bytestring, alors que ``r.text``renvoie une chaîne encodée en
unicode.

::

    >>> import json
    >>> commit_data = json.loads(r.text)
    >>> print commit_data.keys()
    [u'committer', u'author', u'url', u'tree', u'sha', u'parents', u'message']
    >>> print commit_data[u'committer']
    {u'date': u'2012-05-10T11:10:50-07:00', u'email': u'me@kennethreitz.com', u'name': u'Kenneth Reitz'}
    >>> print commit_data[u'message']
    makin' history

Tout simple. Poussons un peu plus loin sur l'API GitHub. Maintenant, nous
pouvons regarder la documentation, mais ce serait plus fun d'utiliser Requests
directement. Nous pouvons tirer profit du verbe HTTP OPTIONS pour consulter
quelles sont les methodes HTTP supportées sur une URL.

::

    >>> verbs = requests.options(r.url)
    >>> verbs.status_code
    500

Comment ça? Cela ne nous aide pas du tout. Il se trouve que GitHubn comme
beaucoup de fournisseurs d'API n'implémente pas la méthode HTTP OPTIONS.
C'est assez embétant mais ca va aller, on peut encore consulter la
documentation. Si GitHub avait correctement implémenté la méhode OPTIONS,
elle retournerait la liste des méthodes autorisées dans les en-têtes, par
exemple.

::

    >>> verbs = requests.options('http://a-good-website.com/api/cats')
    >>> print verbs.headers['allow']
    GET,HEAD,POST,OPTIONS

En regardant la documentation, on découvre que la seule autre méthode HTTP
autorisée est POST, pour créer un nouveau commit. Comme nous utilisons le
repository Requests, nous devrions éviter d'envoyer des requêtes assemblées
manuellement. Nous allons plutôt jouter avec les Issues de GitHub.

Cette documentation a été ajotuée en réponse à l'issue #482. Sachant que cette
issue existe encore, nous allons l'utiliser en exemple. Commençons par la
récupérer.

::

    >>> r = requests.get('https://api.github.com/repos/kennethreitz/requests/issues/482')
    >>> r.status_code
    200
    >>> issue = json.loads(r.text)
    >>> print issue[u'title']
    Feature any http verb in docs
    >>> print issue[u'comments']
    3

Cool, nous avons 3 commentaires. Regardons le dernier.

::

    >>> r = requests.get(r.url + u'/comments')
    >>> r.status_code
    200
    >>> comments = json.loads(r.text)
    >>> print comments[0].keys()
    [u'body', u'url', u'created_at', u'updated_at', u'user', u'id']
    >>> print comments[2][u'body']
    Probably in the "advanced" section

Bon, le commentaire à l'air stupide. Ajoutons un commentaire pour en informer
son auteur. D'ailleurs, qui est-il ?

::

    >>> print comments[2][u'user'][u'login']
    kennethreitz

OK, donc disons à ce Kenneth que l'on pense que cet exemple devrait plutôt aller
dans la section quickstart. D'après la doc de l'API GitHub, il faut utiliser la
méthode POST pour ajouter un commentaire. allons-y.

::

    >>> body = json.dumps({u"body": u"Sounds great! I'll get right on it!"})
    >>> url = u"https://api.github.com/repos/kennethreitz/requests/issues/482/comments"
    >>> r = requests.post(url=url, data=body)
    >>> r.status_code
    404

Mince, c'est bizarre. On doit avoir besoin d'une authentification. Ca va pas être
simple, hein ? Non. Requests rend très simple tout sortes d'authentification,
comme la très classique Basic Auth.

::

    >>> from requests.auth import HTTPBasicAuth
    >>> auth = HTTPBasicAuth('fake@example.com', 'not_a_real_password')
    >>> r = requests.post(url=url, data=body, auth=auth)
    >>> r.status_code
    201
    >>> content = json.loads(r.text)
    >>> print content[u'body']
    Sounds great! I'll get right on it.

Parfait. Hum, en fait non! j'aimerai modifier mon commentaire. Si seulement je
pouvais l'éditer! Heureusement, GitHub nous permet d'utiliser un autre verbe,
PATCH, pour éditer ce commentaire. Essayons.

::

    >>> print content[u"id"]
    5804413
    >>> body = json.dumps({u"body": u"Sounds great! I'll get right on it once I feed my cat."})
    >>> url = u"https://api.github.com/repos/kennethreitz/requests/issues/comments/5804413"
    >>> r = requests.patch(url=url, data=body, auth=auth)
    >>> r.status_code
    200

Excellent. Bon finalement, juste pour embéter ce Kenneth, j'ai décidé de
le laisser attendre et de ne pas lui dire que je travaille sur le problème.
Donc je veux supprimer ce commentaire. GitHub nous permet de supprimer des
commentaire unqiuement avec le verbe bien nommé DELETE. Allons-y.

::

    >>> r = requests.delete(url=url, auth=auth)
    >>> r.status_code
    204
    >>> r.headers['status']
    '204 No Content'

Parfait. Plus rien. La dernière chose que je voudrais savoir c'est combien
j'ai consommé de mon taux de requêtes autorisées. GitHub envoie cette
information dans les en-têtes HTTP, donc au lieu de télécharger toute la page,
on peut simplement envoyer une requête HEAD pour récupérer uniquement les
en-têtes.

::

    >>> r = requests.head(url=url, auth=auth)
    >>> print r.headers
    ...
    'x-ratelimit-remaining': '4995'
    'x-ratelimit-limit': '5000'
    ...

Excellent. Il est temps d'écrire un programme Python qui abuse de l'API GitHub
de toutes les façons possibles, encore 4995 fois :)


Liens dans les en-têtes
-----------------------

De nombreuses APIs HTTP fournissent des liens dans les en-têtes (Link headers). Ceci rend les
APIs plus auto-descriptives et détéctables.

GitHub les utilise dans son API pour la `pagination <http://developer.github.com/v3/#pagination>`_, par exemple::

    >>> url = 'https://api.github.com/users/kennethreitz/repos?page=1&per_page=10'
    >>> r = requests.head(url=url)
    >>> r.headers['link']
    '<https://api.github.com/users/kennethreitz/repos?page=2&per_page=10>; rel="next", <https://api.github.com/users/kennethreitz/repos?page=6&per_page=10>; rel="last"'

Requests analyse automatiquement ces liens d'entête et les rends facilement utilisables::

    >>> r.links['next']
    'https://api.github.com/users/kennethreitz/repos?page=2&per_page=10'

    >>> r.links['last']
    'https://api.github.com/users/kennethreitz/repos?page=6&per_page=10'

