# Documentacion de la actividad 0

Nombre del alumno: Emanuel leiva

Primero nos aseguramos que tengamos instalado Django con el siguiente comando: 
```
$ python -m django --version
```
En mi caso use un entorno virtual para aislar dependencias entonces cada vez que quiero activar el entorno virtual en mi sistema operativo y poder ejecturar archivos .py uso:
```
$ source env/bin/activate
```
Como primer paso se crea el proyecto con el siguiente comando:
```
$ django-admin startproject mysite
```
Esto autogenerara un codigo que establezca un Django project osea un conjunto de ajustes para una instancia de Django

Hecho esto tendremos un directorio llamado mysite con la siguiente estructura
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```
Donde estos archivos se encargaran de:
- El directorio externo llamdo *mysite* que es el contenedor de todo el proyecto
- El archivo manage.py que permitira interactuar con el proyecto Django de difentes formas
- El paquete *mysite* que contendra todas las configuraciones del proyecto
- El archivo __init__.py indica que *mysite* debe ser considerado como un paquete que puede ser importado
- El archivo settings.py indica configuraciones para el proyecto django
- El archivo urls.py tendra las rutas que usara el proyecto 
- El archivo asgi.py es el punto de entrada para correr Django con un servidor asincronico
- El archivo wsgi.py es el punto de entrada para que los servidores web puedan servir al proyecto 

Ahora para que nosotros veamos si nuestro proyecto de Django funciona lo primero que haceremos es ejecutar 
```
$ python manage.py runserver
```

Una vez configurado el proyecto podremos crear nuestra aplicacion en el mismo directorio que esta el archivo manage.py. Entonces escribimos el siguiente comando 
```
$ python manage.py startapp polls
```
Dandonos la siguiente estructura
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

Ahora escribiremos la primera vista (Una vista se encargara de manejar una request y devolver una respuesta adecuada) 
Entonces en nuestro archivo views.py pondremos
```
from django.http import HttpResponse #Se importa el modulo que crea respuestas HTTP

def index(request): # Se define una vista que recibira una request y devolvera algo
    return HttpResponse("Hello, world. You're at the polls index.") #Es la respuesta que se le manda al usuario
```
Para acceder a esta vista vamos a asignarle una URL donde para ello debemos definirlas en el archivo polls/urls.py
```
from django.urls import path #Se importa la funcion path que sirve para definir rutas/URLs

from . import views # Se importan todas las vistas 

urlpatterns = [ #Se definira la lista de rutas que Django usara para decidir que vista ejecutar segun la URL que reciba
    path("", views.index, name="index"),
]
```

Como siguiente paso hay que configurar la URLconf global en el proyecto mysite para que incluya la URLconf definida en polls.urls
mysite/urls.py
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```
La funcion include() permite hacer referencia a otros URLconfs

Ahora que la vista index esta vinculada en los URLconfs se comprueba el funcionamiento con 
```
$ python manage.py runserver
```
Los argumentos de path() son 4 
- route: Cadena que contiene un patron de URL 
- view: Llama a la funcion de la vista especificada con un objeto httpRequest
- kwargs
- name: Da nombre a la URL esto permitira referirse a ella en otras parte de Django


Configuracion con la BD

Ahora se abre el archivo mysite/settings.py que contendra la configuracion de Django 
Por defecto la configuracion utiliza SQLite pero si se quiere cambiar a otra base de datos mas potente en el item DATABASES 'default' cambie la claves 
- ENGINE: por otra base de datos
- NAME: El nombre de tu base de datos
Tambien observe la configuracion de INSTALLED_APP que se encuentra en la parte superior del archivo. Aca se veran todos los nombres de todas las aplicaciones Django que estan activas en esta instancia de Django
Por defecto, INSTALLED_APPS contiene las siguientes aplicaciones y Django viene equipado con todas ellas:
- django.contrib.admin - El sitio administrativo.
- django.contrib.auth– Un sistema de autenticación.
- django.contrib.contenttypes– Un framework para los tipos de contenido.
- django.contrib.sessions – Un framework de sesión.
- django.contrib.messages – Un framework de mensajería.
- django.contrib.staticfiles – Un framework para la gestión de archivos estáticos
Estas aplicaciones utilizan al menos una tabla de base de datos entonces las crearemos con 
```
$ python manage.py migrate
```
Esta funcion examina la configuracion de INSTALLED_APP creando las tablas de la BD necesarias segun la configuracion de la BD en el archivo setting.py y las migraciones de BD incluidas con la aplicacion 

Empezaremos a crear los modelos en el archivo polls/models.py
```
from django.db import models


class Question(models.Model): //Nombre de la tabla que se creara
    question_text = models.CharField(max_length=200) #Texto de hasta 200 caracteres
    pub_date = models.DateTimeField("date published") #Fecha y hora en la que se publico la pregunta 


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE) # Relacion entre question y choice
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
Una vez hecho esto se activan los modelos. Para hacer esto tenemos que incluir la aplicacion de polls en la configuracion INSTALLED_APP. La clase PollsConfig esta en el archivo polls/app.py por lo que la ruta es 'polls.apps.PollsConfig' entonces se edita el archivo mysite/settings.py y se agrega esa ruta en INSTALLED_APPS
```
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

Ahora Django sabe incluir la aplicacion polls. entonce se ejecuta
```
$ python manage.py makemigrations polls
```
Al hacer makemigrations le indicas a Django que se realizaron cambios en los modelos y que se guardaran los cambios como una migracion
Para ver cual SQL ejecutaria esa migracion es 
```
$ python manage.py sqlmigrate polls 0001
```
Por ultimo creamos esas tablas modelos en su BD
```
$ python manage.py migrate
```

Para usar la API usamos: 
```
$ python manage.py shell
```

Ahora haremos que la respuesta de los modelos agregando __str__() que mostrara a los objetos de una manera mas linda 
```
from django.db import models


class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

Ahora para hacer que la aplicacion encuesta se pueda modificar en el sitio administrativo ponemos en polls/admin.py
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
Basicamente lo que hace esto es mostrar el modelo qustion en el panel de administracion

Una vista es un tipo de página web en tu aplicación Django que generalmente cumple una función específica y tiene una plantilla específica
Para llegar desde una URL a una vista Django se usa las URLconfs

Ahora vamos a escribir mas vistas 
polls/views.py
```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

Se tienen que unir estas vistas al modulo de polls.url anadiendo las siguientes llamadas path()
polls/urls.py
```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

Cada vista es responsable de hacer una de dos cosas:
- Devolver un objeto HttpResponse con el contenido de la pagina solicitada
- Levantar una excepcion como por ejemplo Http404

Ahora vamos a senalar una nueva vista index() que muestra en el sistema las 5 ultimas preguntas de la encuesta separadas por comas, segun su fecha de publicacion
polls/views.py
```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)


# Leave the rest of the views (detail, results, vote) unchanged
```
Pero de esta forma el diseno de la pagina esta codificado de forma fija en la vista. Entonces usaremos el sistema de plantillas de Django para separar el diseno de Python mediante la creacion de una plantilla que la vista pueda utilizar
Se creara un directorio templates en el directorio polls. Django buscara alli las plantillas 
Entonces la configuracion de TEMPLATES de su proyecto describe como Django cargara y creara las plantillas. 
Dentro del directorio templates crear el directorio polls y dentro el archivo index.html donde dentro de ese archivo tendremos 
polls/templates/polls/index.html
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
Ahora vamos a actualizar nuestra vista index en polls/views.py para usar la plantilla
```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
Esto lo que hara es cargar la plantilla creada y le pasa un contexto (El contexto es un diccionario que relaciona los nombres de variables de plantillas con objetos Python)

Una practica comun es cargar una plantilla, llenar un contexto y retornar un objeto HttpResponse con el resultado de la plantilla creada
polls/views.py
```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```
Una vez hecho esto no necesitaremos importar loader y HttpResponse ya que la funcion render hace las dos cosas por un lado carga la plantilla y devuelve un objeto HttpResponse
La funcion render() tomar el objeto de una solicitud como primer argumento, un nombre de plantilla como su segundo argumento y un diccionario como su tercer argumento opcional. Esto devolvera un objeto HttpResponse de la plantilla que se ha proporcionado representada con el contexto dado

Ahora se vera como levantar una excepcion Http404 si no existe una pregunta con la ID solicitada
polls/views.py
```
from django.http import Http404
from django.shortcuts import render

from .models import Question


# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```

Una practica comun es utilizar get() y levantar la excepcion Http404 si no existe el objeto. Django proporciona un atajo
```
from django.shortcuts import get_object_or_404, render

from .models import Question


# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```
La funcion get_object_or_404() toma un modelo Django como su primer argumento y un numero arbitrario de argumentos de palabra clave que pasa a la funcion get() del administrador del modelo. Se levantara una excepcion Http404 si no existee el objeto

Ahora volviendo a la vista detail()
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %} #La funcion devuelve un iterable de objetos choise que es recorrido con el for
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```
El sistema de plantillas utiliza la sintaxis de busqueda con puntos para acceder a los atributos de variables. {{ question.question_text }}, Django primero realiza una consulta en el diccionario sobre el objeto question. Si esto falla, intenta una búsqueda de atributos, que en este caso funciona. Si la búsqueda de atributos hubiera fallado, hubiera intentado una búsqueda de índice de lista.

Ahora nosotros teniamos el enlace de una pregunta en la plantilla polls/index.html parcialmente codificada de esta forma 
```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
El problema con este enfoque duro y estrechamente acoplado es que se vuelve difícil cambiar URLs en proyectos con una gran cantidad de plantillas. Pero como ahora nosotros definimos los names de las URL junto con la etiqueta {% url %} permitira no depender de escribir las URLs a mano 

```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

Ahora anadimos espacios de nombres en URLconf en polls/urls.py anadimos app_name para configurar el espacio de nombres de la aplicacion
Este app_name sirve para referenciar las URL de distintas aplicacionse que puede tener un proyecto. Por ejemplo la aplicacion polls tiene una vista details pero la aplicacion blog tambien puede tener una vista details entonces hay que especificar a URL pertenece
```
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
Ahora modificamos la plantilla polls/index.html desde
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
Ahora modificamos la vista de detalle con espacio de nombres
```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

Ahora se actulizar la plantilla de detalles de encuestas para contenga un <form>
```
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```
Ahora haremos una vista Django que maneje los datos enviados y haga algo con ellos
```
path("<int:question_id>/vote/", views.vote, name="vote"),
```
Tambien se crea una implementacion simulada de la funcion vote()
polls/views.py
```
from django.db.models import F
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"]) #Se intenta obetener la opcion seleccionada en el formulario 
    except (KeyError, Choice.DoesNotExist): #Si no se mando ningun dato o no existe la pregunta devuelve un mensaje de error
        # Redisplay the question voting form.
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else: #Si todo sale bien entonces
        selected_choice.votes = F("votes") + 1 #Le dice a django que aumente en 1 el numero de votos. F('votes') es mas seguro y permite concurrencia ya que va directo a la BD
        selected_choice.save() #Guarda los cambios
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,))) #Redirecciona ala pagina de resultados
```
Despues de que alguien vota en una pregunta, la vista vote() remite a la pagina de resultados de la pregunta
```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})
```
Ahora creamos una plantilla polls/resukt.html
```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```