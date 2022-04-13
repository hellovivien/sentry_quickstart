# Sentry

Sentry surveille l'execution de votre application et enregistre les **erreurs**, les **exceptions** et les 
métriques liées à la **performance**. Il est compatible avec la majorité des languages et framework existants.

![](https://i.imgur.com/qZzal33.png)

Sentry est proposé en version **cloud** ou **self hosted**.
La version cloud dispose d'une offre **gratuite** qui nécessite un compte pour avoir accès à son dashboard et les fonctions sont plus limitées mais largement suffisante pour le projet:

* 5K errors
* 10K transactions
* Alertes (sauf metrics de performance)

![](https://i.imgur.com/jCO7Xqi.png)


## Installation

Pour la version auto hébergée, suivre la doc:
https://develop.sentry.dev/self-hosted/

Pour la version cloud, créer un compte:
https://sentry.io/signup/

installation:
```bash
pip install --upgrade sentry-sdk
```

## Utilisation


Créer un nouveau projet et copier la clé DSN (Data Source Name) fournie ou retrouvez la dans [Project] > Settings > Client Keys (DSN).

Dans votre fichier app.py ou main.py

---

### Flask

```python    
import sentry_sdk
from flask import Flask
from sentry_sdk.integrations.flask import FlaskIntegration

sentry_sdk.init(
    dsn="https://e1dc8d22210d40a7a90b2dfd7155ac9c@o1196630.ingest.sentry.io/6321460",
    integrations=[FlaskIntegration()],
    traces_sample_rate=1.0,
    # release="myapp@1.0.0", # or SENTRY_RELEASE env var or git commit
)

app = Flask(__name__)
```

#### test

```python=
@app.route('/debug-sentry')
def trigger_error():
    division_by_zero = 1 / 0
```
---

### Django

```python
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn="https://examplePublicKey@o0.ingest.sentry.io/0",
    integrations=[DjangoIntegration()],

    traces_sample_rate=1.0,
    send_default_pii=True,
)
```

#### test

```python=
from django.urls import path

def trigger_error(request):
    division_by_zero = 1 / 0

urlpatterns = [
    path('sentry-debug/', trigger_error),
    # ...
]
```
---
### Streamlit

```python
import sentry_sdk
sentry_sdk.init(
    "https://e1dc8d22210d40a7a90b2dfd7155ac9c@o1196630.ingest.sentry.io/6321460",
    traces_sample_rate=1.0,
)
```

#### test

il faut capturer les erreurs manuellement car streamlit les intercepte.

```python
try:
    division_by_zero = 1 / 0
except Exception as e:
    sentry_sdk.capture_exception(e)
    raise
```
---
## Et après?

C'est tout, votre monitoring est opérationnel.

La navigation sur le dashboard de Sentry est plutôt intuitive


On peut créer des **hooks** si on veut enrichir ou réduire les données que va enregistrer Sentry.


![](https://i.imgur.com/l2H5Eb5.png)


On peut appeler la capture manuellement dans un bloc **try...except**.


![](https://i.imgur.com/DPd0SGq.png)


On peut aussi juste capturer un **message**.

```python
sentry_sdk.capture_message("You caught me!", "fatal")
```
---
## Performance

Sentry enregistre le temps d'execution de toutes les transactions.

Il propose plusieurs indicateurs de performance, tous se basent sur le **Response Time Threeshold** qui est le temps maximum en ms qu'un utilisateur devrait attendre pour avoir une réponse lors d'un appel à un endpoint.

Settings > [organisation] > [project name] > Performance

![](https://i.imgur.com/o4bbHrz.png)

La métrique la plus pertinente est l'**Apdex**.

C'est un standard de l'industrie utilisé pour mesurer la satisfaction basé sur le temps de réponse de l'application. L'Apdex se base sur un ratio Satisfaisant/Tolérable/Frustration du nombre de requêtes pour un endpoint particulier.

soit T le Threeshold

Satisfaisant : temps de répose <= T
Tolérable : T > temps de réponse >= 4T
Frustration : temps de réponse > 4T

Apdex = Nombre de requête satisfaisantes + (Nombre de requêtes tolérable / 2) / Nombre totale de requête
