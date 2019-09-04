**What is this repository about?**

This repository serves as a demo setup illustrating the basic workflow of
testing a Django & Postgres web app with Travis (continuous integration) &
deployment to Heroku (continuous deployment).

---

**Prerequisite:**

This post assumes that the reader has accounts with Github, Travis & Heroku &
already has the accounts configured. For example, linking Travis with Github,
setting up Postgres server in Heroku & setting OS environment variables in
Travis & Heroku websites.

---

See [here](https://chuacheowhuan.github.io/DPDTH) for a Dockerized
version.

---

**Which copy of Postgres to use during the different stages in the workflow?**

During development, the local Postgres database server is used. When testing in
Travis, we'll used Travis's copy of Postgres & when deploying, we'll have to
use Heroku's copy.

---

**makemigrations**

Always run ```python manage.py makemigrations``` before deployment to Heroku
or in our case, before pushing to Github.

The actual ```python manage.py migrate``` for the Postgres server addon from
Heroku will be run by the deploy section in the ```.travis.yml``` file.

---

**Listing files & directories in a tree:**

```
cd web_app_DPTH

$ tree -a -I "CS50_web_dev|staticfiles|static|templates|LICENSE|README.md|__init__.py|settings_DPTH_.py|urls.py|wsgi.py|db.sqlite3|airline4_tests_.py|apps.py|migrations|views.py|models.py|flights.csv|manage.py|wait-for-it.sh|admin.py|.git|.travis_DPTH_.yml|Dockerfile|docker-compose.yml"  

.
├── .travis.yml
├── Procfile
├── airline
│   └── settings.py
├── flights
│   └── tests.py
└── requirements.txt
```

As shown in the tree above, the 5 files that matter in the workflow:

1) tests.py

2) settings.py

3) requirements.txt

4) .travis.yml

5) Procfile

We will look at the contents of each of the 5 files in the sections below.

---

**tests.py**

This is the test file that Travis will use for testing the app.
You write whatever test you want for Travis to run with.

```
from django.db.models import Max
from django.test import Client, TestCase

from .models import Airport, Flight, Passenger

# Create your tests here.
class FlightsTestCase(TestCase):

    def setUp(self):

        # Create airports.
        a1 = Airport.objects.create(code="AAA", city="City A")
        a2 = Airport.objects.create(code="BBB", city="City B")

        # Create flights.
        Flight.objects.create(origin=a1, destination=a2, duration=100)
        Flight.objects.create(origin=a1, destination=a1, duration=200)
        Flight.objects.create(origin=a2, destination=a1, duration=300)

    # 1
    def test_departures_count(self):
        a = Airport.objects.get(code="AAA")
        self.assertEqual(a.departures.count(), 2)

    # 2
    def test_arrivals_count(self):
        a = Airport.objects.get(code="AAA")
        self.assertEqual(a.arrivals.count(), 2)

    # 3
    def test_valid_flight(self):
        a1 = Airport.objects.get(code="AAA")
        a2 = Airport.objects.get(code="BBB")
        f = Flight.objects.get(origin=a1, destination=a2)
        self.assertTrue(f.is_valid_flight())
```

---

**settings.py**

Under the database section, the database credentials, as OS environment
variables, has to be made available to Travis & Heroku. They can be set in
their respective websites.

Add and/or edit the following to the ```settings.py``` file:

```
import django_heroku
import dj_database_url
```

```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',

    'whitenoise.middleware.WhiteNoiseMiddleware',  # new

    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ['DATABASE_NAME'],
        'USER': os.environ['DATABASE_USER'],
        'PASSWORD': os.environ['DATABASE_PASSWORD'],
        'HOST': os.environ['DATABASE_HOST'],
        'PORT': os.environ['DATABASE_PORT'],
    }
}
```

```
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

```
django_heroku.settings(locals())

```

---

**requirements.txt**

This file lets Travis know what packages are needed for the app.

```
django>=2.0.11
psycopg2
psycopg2-binary
dj-database-url==0.5.0
gunicorn
whitenoise
django-heroku
pytz
sqlparse
```

---

**.travis.yml**

This file contains instructions for Travis & is needed when Travis starts
running. ```$HEROKU_API_KEY``` can be generated from the Heroku website &
stored as an OS environment variable in the Travis website.
The test is done with the **Travis's copy** of Postgres.

```
language: python
python:
    - 3.6
services:
    - postgresql
install:
    - pip install -r requirements.txt
script:
    - python manage.py test
deploy:
    provider: heroku
    api_key: $HEROKU_API_KEY
    app: webapp-dpth
    run: python manage.py migrate
    on: master
```

---

**Procfile**

This file is for Heroku. It tells Heroku to deploy the web app using Gunicorn
as the production server.

Note that ```airline``` is the Django project name.

```
web: gunicorn airline.wsgi
```

---

**The deployed web app**

With the above files in place, push to Github & Travis will start testing.
After all tests passed, deployment starts. If there isn't any failures,
the web app will be running on:

[https://webapp-dpth.herokuapp.com](https://webapp-dpth.herokuapp.com)

This [link](https://webapp-dpth.herokuapp.com/admin) brings you to the admin
page. It is using the **Heroku's copy** of Postgres.

This [link](https://travis-ci.com/ChuaCheowHuan/web_app_DPTH/builds/125456399)
brings you to my built log in Travis.com which shows how a successful
test/deploy built looks like.

---

**Web security:**

Please note that web security has not been throughly consider in this basic
workflow describe above. **Do NOT** simply use the above workflow for
production.

For example the ```SECRET_KEY``` in the ```settings.py``` isn't dealt with at all
and web security is really beyond the scope of this post.

---

<br>
