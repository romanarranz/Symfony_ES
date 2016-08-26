Plantillas
===========================

Una plantilla es simplemente un archivo de texto para generar XML, HTML, LaTeX, CSV etc..., normalmente se venian usando 
plantillas creadas directamente con PHP.

Por ejemplo:

```php
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1><?php echo $page_title ?></h1>

        <ul id="navigation">
            <?php foreach ($navigation as $item): ?>
                <li>
                    <a href="<?php echo $item->getHref() ?>">
                        <?php echo $item->getCaption() ?>
                    </a>
                </li>
            <?php endforeach ?>
        </ul>
    </body>
</html>
```
Pero Symfony hace uso de Twig que nos ayuda a escribir plantillas de una forma más sencilla

```twig
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1>{{ page_title }}</h1>

        <ul id="navigation">
            {% for item in navigation %}
                <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
            {% endfor %}
        </ul>
    </body>
</html>
```

Twig define 3 tipos especiales en su sintaxis:
- `{{...}}` imprime una variable o el resultado de una expresión regular.
- `{%...%}` ejecuta reglas, como pueden ser bucles o estructuras de control.
- `{#...#}` comentarios.

Además Twig tiene la posibilidad de modificar el contenido en su renderizado con filtros.

**Pasar a mayúscula** la variable title:
```twig
{{title|uppercase}}
```

**Ejemplo** de como poner una **clase CSS** a los **divs pares(cursiva)** distinta de los **impares(normal)**.

```twig
{# Renderizamos 10 bloques #}
{% for i in 0..10 %}
	{# Llamamos a la funcion cycle para alternar resultados entre las clases odd y even #}
    <div class="{{ cycle(['odd', 'even'], i) }}">
      <!-- Contenido -->
    </div>
{% endfor %}
```

**Ejemplo de estructura de control if**:

```twig
<ul>
    {% for user in users if user.active %}
        <li>{{ user.username }}</li>
    {% else %}
        <li>No users found</li>
    {% endfor %}
</ul>
```

###Caché de Plantillas Twig
Quedan almacenadas en el directorio **var/cache/{environment}/twig** donde `{environment}` puede ser `dev`(desarrollo) o `prod`(produccion).

Cuando el modo `debug` está habilitado (entorno `dev`) cada vez que se hace un cambio en una plantilla Twig se recompila automaticamente, asi que veremos lo cambios sin necesidad de borrar caché.

Cuando el modo `debug` está deshabilitado (entorno `prod`) sin embargo tendremos que limpiar la caché para visualizar los nuevos cambios.

##Herencia de plantillas y diseños

Aqui hacemos referencia a las distintas secciones que dispone una página web como pueden ser la cabecera, cuerpo y pie.
Las **plantillas** pueden ser diseñadas **conteniendo** a **otras plantillas**, es decir, una plantilla puede extender de otra, podrá redefinir e implementar nuevos elementos.

**Plantilla padre**

```twig
{# app/Resources/views/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Test Application{% endblock %}</title>
    </head>
    <body>
        <div id="sidebar">
            {% block sidebar %}
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            {% endblock %}
        </div>

        <div id="content">
            {% block body %}{% endblock %}
        </div>
    </body>
</html>
```

**Plantilla hija** en la que implementamos el bloque `body` y redefinimos el bloque `title`
```twig
{# app/Resources/views/blog/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}My cool blog posts{% endblock %}

{% block body %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

**Importante**
Si encontramos contenido duplicado seguramente será porque hemos definido un `{% block %}` junto a su implementación tanto
en la plantilla padre como en la hija que hereda su contenido.

Si queremos obtener el contenido de un `{% block %}` de la plantilla padre podemos usar la función `{{ parent() }}`, esto
puede resultar **útil** si tan solo queremos **añadir contenido en lugar de machacarlo** con otro nuevo.

```twig
{% block sidebar %}
    <h3>Table of Contents</h3>

    {# ... #}

    {{ parent() }}
{% endblock %}
```

##Nombres de plantillas y sus Ubicaciones

Por defecto las plantillas pueden alojarse en:
- **app/Resources/views/**, estas plantillas tienen preferencia ante las plantillas de terceros.
- **path/to/bundle/Resources/views/**, cuando queremos compartir un `Bundle` podemos compartir sus plantillas alojadas en este directorio y sus subdirectorios.

Normalmente las plantillas que usaremos estrán en **app/Resources/views/**, para extender una plantilla de ese directorio 
simplemente llamamos al recurso en cuestión usando rutas relativas.

Si queremos extender de `app/Resources/views/base.html.twig` extenderemos de `base.html.twig` porque el path es relativo,
o bien si queremos extender de un subdirectorio `app/Resources/views/blog/index.html.twig` lo hacemos con `blog/index.html.twig`.

###Referencias a plantillas en los Bundles

Symfony usa una sintaxis de esta forma `bundle:directory:filename` para las plantillas que se alojan en un *Bundle*.

Según si la plantilla está dentro de un subdirectorio o no, hay 2 referencias:

- `AcmeBlogBundle:Blog:index.html.twig`: hacemos referencia a una plantilla de una página específica.
	- `AcmeBlogBundle`es el bundle alojado en `src/Acme/BlogBundle`
	- `Blog` es el directorio alojado en `Resources/views`
	- `index.html.twig` es la plantilla
	- A modo de resumen la plantilla estará en `src/Acme/BlogBundle/Resources/views/Blog/index.html.twig`

- `AcmeBlogBundle::layout.html.twig`: en esta ocasión no hay un subdirectorio especifico, por lo tanto la plantilla se aloja en la propia raiz del bundle que estará en `Resources/views/layout.html.twig`

###Sufijos de plantillas

|Nombre de archivo     	| Formato  	| Engine
|:---------------------:|:---------:|:----------:
blog/index.html.twig   	| HTML  	| TWIG
blog/index.html.php 	| HTML  	| PHP
blog/index.css.twig 	| CSS  		| TWIG

Formato es el formato de salida del archivo y el engine es el parser.

##Utilidades para crear Plantillas

###Incluir unas plantillas en otras

Cuando vamos a repetir un contenido N veces en distintos sitios se suele crear una plantilla para incluirla donde necesitemos.

```twig
{# app/Resources/views/article/article_details.html.twig #}
<h2>{{ article.title }}</h2>
<h3 class="byline">by {{ article.authorName }}</h3>

<p>
    {{ article.body }}
</p>
```

Ejemplo para **incluir una plantilla en otra** haciendo una llamada a la función **include()**:

```twig
{# app/Resources/views/article/list.html.twig #}
{% extends 'layout.html.twig' %}

{% block body %}
    <h1>Recent Articles<h1>

    {% for article in articles %}
        {{ include('article/article_details.html.twig', { 'article': article }) }}
    {% endfor %}
{% endblock %}
```
La variable `article` que utiliza la plantilla `article_details.html.twig` tiene que ser especificada a través de un hashmap que establecemos como segundo parámetro en el cual podemos especificar más pares clave - valor si fuera necesario.

###Embebiendo Controladores

Supongamos que tenemos un sidebar que contiene las 3 noticias más recientes, esto implica que hay una conexión a una BD
que en una plantilla de Twig no se puede hacer. Por lo tanto lo que haremos es que el controler renderice la salida de una plantilla Twig.

```php
// src/AppBundle/Controller/ArticleController.php
namespace AppBundle\Controller;

// ...

class ArticleController extends Controller
{
    public function recentArticlesAction($max = 3)
    {
        // CONEXION A LA BASE DE DATOS
        // OBTENER LOS ULTIMOS $max ARTICULOS
        $articles = ...;

        return $this->render(
            'article/recent_list.html.twig',
            array('articles' => $articles)
        );
    }
}
```

Su plantilla Twig:

```twig
{# app/Resources/views/article/recent_list.html.twig #}
{% for article in articles %}
	{# Disponer la URL de esta forma es una mala práctica #}
    <a href="/article/{{ article.slug }}">
        {{ article.title }}
    </a>
{% endfor %}
```

Ahora vamos a **embeber el Controlador Enterno en una plantilla Twig** usando la sintaxis que vimos anteriormente

```twig
{# app/Resources/views/base.html.twig #}

{# ... #}
<div id="sidebar">
    {{ render(controller(
        'AppBundle:Article:recentArticles',
        { 'max': 3 }
    )) }}
</div>
```

###Carga de contenido Asíncrona

Los controladores pueden ser embebidos de forma asíncrona usando [hinclude.js](http://mnot.github.io/hinclude/), la librería debe de ser incluida en su correspondiente etiqueta dentro de la página

```html
<script src="hinclude.js"></script>
```

Symfony ofrece una función de renderizado usando esta librería

```twig
{{ render_hinclude(controller('...')) }}
{{ render_hinclude(url('...')) }}
```

**Nota**: cuando usamos un controlador en lugar de una URL tenemos que habilitar `fragments` en la configuración.

```yml
# app/config/config.yml
framework:
    # ...
    fragments: { path: /_fragment }
```

Podemos cargar una plantilla por defecto

```twig
{{ render_hinclude(controller('...'),  {
    'default': 'default/content.html.twig'
}) }}
```

o incluso un string

```twig
{{ render_hinclude(controller('...'), {'default': 'Loading...'}) }}
```

###Enlaces a Páginas

Para esta tarea usaremos la función `path` de Twig o las reglas `@Route` dentro de los controladores PHP para generar
las URL.

**Ejemplo**, vamos a crear un **enlace a** la página **_welcome**

En el controlador PHP
```php
// src/AppBundle/Controller/WelcomeController.php

// ...
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class WelcomeController extends Controller
{
    /**
     * @Route("/", name="_welcome")
     */
    public function indexAction()
    {
        // ...
    }
}
```

Y su correspondiente configuración en `routing`
```yml
# app/config/routing.yml
_welcome:
    path:     /
    defaults: { _controller: AppBundle:Welcome:index }
```

Para hacer un enlace a esta página `_welcome` desde una plantilla Twig usamos `path`, lo que terminará generando la URL `/`

```twig
<a href="{{ path('_welcome') }}">Home</a>
<!-- También es valido usar la función url -->
<a href="{{ url('_welcome') }}">Home</a>
```

**Ejemplo Avanzado con parámetros**

```php
// src/AppBundle/Controller/ArticleController.php

// ...
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class ArticleController extends Controller
{
    /**
     * @Route("/article/{slug}", name="article_show")
     */
    public function showAction($slug)
    {
        // ...
    }
}
```

Su configuración en `routing`
```yml
# app/config/routing.yml
article_show:
    path:     /article/{slug}
    defaults: { _controller: AppBundle:Article:show }
```

En la parte Twig tendremos que llamar a `path` con un parámetro para `{slug}`

```twig
{# app/Resources/views/article/recent_list.html.twig #}
{% for article in articles %}
    <a href="{{ path('article_show', {'slug': article.slug}) }}">
        {{ article.title }}
    </a>
{% endfor %}
```

###Enlazando Recursos

Para enlazar imágenes, archivos CSS, JS etc..., usaremos la función `assets` que proporciona Twig.

```twig
<!-- Enlace de Imagen -->
<img src="{{ asset('images/logo.png') }}" alt="Symfony!" />
<!-- Enlace de un archivo CSS -->
<link href="{{ asset('css/blog.css') }}" rel="stylesheet" />
```

`assets` se encarga de obtener la ruta relativa del recurso, de manera que si nosotros tuvieramos la aplicación dentro de un subdirectorio `http://example.com/my_app` cada assets obtiene el recurso relativamente es decir si queremos enlazar a logo.png obtenemos `/my_app/images/logo.png`.

Si necesitamos una URL absoluta para el recurso podemos usar la función `absolute_url`

```twig
<img src="{{ absolute_url(asset('images/logo.png')) }}" alt="Symfony!" />
```

###Incluyendo CSS y JS en las plantillas Twig

Para esta tarea vamos a definir dos bloques en la plantilla base de Twig, uno para el CSS en el head y otro para el JS al final del body, como en la mayoria de sitios web.

```twig
{# app/Resources/views/base.html.twig #}
<html>
    <head>
        {# ... #}

        {% block stylesheets %}
            <link href="{{ asset('css/main.css') }}" rel="stylesheet" />
        {% endblock %}
    </head>
    <body>
        {# ... #}

        {% block javascripts %}
            <script src="{{ asset('js/main.js') }}"></script>
        {% endblock %}
    </body>
</html>
```

Si tuvieramos una plantilla `contact.html.twig` que hereda esta plantilla `base.html.twig` y queremos añadirle un estilo propio CSS haríamos lo siguiente:

```twig
{# app/Resources/views/contact/contact.html.twig #}
{% extends 'base.html.twig' %}

{# Hacemos un Override del bloque stylesheets del padre, incluimos el contenido que tuviera y añadimos el nuevo #}
{% block stylesheets %}
    {{ parent() }}

    <link href="{{ asset('css/contact.css') }}" rel="stylesheet" />
{% endblock %}

{# ... #}
```

Incluso podríamos acceder a recursos que se encuentran dentro de otros Bundles de terceros `/vendor/symfony/symfony/src/Symfony/Bundle/FrameworkBundle/Resources/public` ejecutando el siguiente comando:

```bash
# Crea un enlace simbolico a los recursos del bundle
# target por defecto es web
$ php bin/console assets:install target [--symlink]
```

Ahora podríamos acceder a esos recursos

```twig
<link href="{{ asset('bundles/acmedemo/css/contact.css') }}" rel="stylesheet" />
```

###Variables Globales en Plantillas

Cada vez que Symfony recibe una petición de un recurso se genera una variable global llamada `app` que pertenece a la clase [GlobalVariables](http://api.symfony.com/3.0/Symfony/Bundle/FrameworkBundle/Templating/GlobalVariables.html), se genera tanto en el motor Twig como en PHP.

- `app.user` objeto del usuario actual
- `app.request` objeto de la petición
- `app.session` objeto de la sesión
- `app.environment` objeto del entorno (dev, prod, etc)
- `app.debug` true si está en modo debug

En Twig:
```twig
<p>Username: {{ app.user.username }}</p>
{% if app.debug %}
    <p>Request method: {{ app.request.method }}</p>
    <p>Application Environment: {{ app.environment }}</p>
{% endif %}
```

En PHP:
```php
<p>Username: <?php echo $app->getUser()->getUsername() ?></p>
<?php if ($app->getDebug()): ?>
    <p>Request method: <?php echo $app->getRequest()->getMethod() ?></p>
    <p>Application Environment: <?php echo $app->getEnvironment() ?></p>
<?php endif ?>
```

##Configuración y uso del servicio de plantillas

El núcleo del sistema de plantillas en Symfony es el `Engine` que es el encargado de renderizar las plantillas.

Mediante atajos:

```php
return $this->render('article/index.html.twig');
```

O de forma explicita:

```php
use Symfony\Component\HttpFoundation\Response;

$engine = $this->container->get('templating');
$content = $engine->render('article/index.html.twig');

return $response = new Response($content);
```

Incluso podemos especificar que tipo de motor queremos dentro de los parámetros de configuración

```yml
# app/config/config.yml
framework:
    # ...
    templating: { engines: ['twig'] }
```

##Sobreescribiendo plantillas de Bundles de terceros

Supongamos el caso en el que queramos modificar el aspecto de una plantilla de un blog.

Controlador:
```php
public function indexAction()
{
    // OBTENER LISTA DE BLOGS
    $blogs = ...;

    $this->render(
        'AcmeBlogBundle:Blog:index.html.twig',
        array('blogs' => $blogs)
    );
}
```

Cuando se renderiza `AcmeBlogBundle:Blog:index.html.twig` Symfony busca la plantilla en dos localizaciones:

1. `app/Resources/AcmeBlogBundle/views/Blog/index.html.twig `
2. `src/Acme/BlogBundle/Resources/views/Blog/index.html.twig`

Lo que debemos hacer para poder sobreescribir la plantilla es copiarla a la carpeta de `views`:

```bash
# Creamos la estructura de directorios
$ mkdir app/Resources/AcmeBlogBundle
$ mkdir app/Resources/AcmeBlogBundle/views
$ mkdir app/Resources/AcmeBlogBundle/views/Blog

# Copiamos la plantilla
$ cp src/Acme/BlogBundle/Resources/views/Blog/index.html.twig app/Resources/AcmeBlogBundle/views/Blog/index.html.twig

# Limpiamos la caché para que reindexe los cambios
$ php bin/console cache:clear
```

**¿Y si esa plantilla heredaba de otra?**, tendremos que copiar los archivos de los que heredaba a la carpeta `views`.
Si heredaba de `AcmeBlogBundle::layout.html.twig` lo copiamos y podremos editar ambos archivos al gusto.

##Niveles de Herencia

Ejemplo de 3 niveles en la linea de herencia:
- Creamos un archivo `app/Resources/views/base.html.twig` que sería el layout principal de nuestra aplicación.
- Creamos una plantilla por cada sección de nuestro sitio, por ejemplo las secciones de un blog `blog/layout.html.twig`

```twig
{# app/Resources/views/blog/layout.html.twig #}
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Blog Application</h1>

    {% block content %}{% endblock %}
{% endblock %}
```

- Creamos una plantilla para cada entrada del blog, por ejemplo para la entrada de inicio `blog/index.html.twig`
```twig
{# app/Resources/views/blog/index.html.twig #}
{% extends 'blog/layout.html.twig' %}

{% block content %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

##Escape del Output, evitando ataques XSS

Cuando se genera un HTML usando plantillas hay un riesgo de ataque [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting), consideremos el siguiente ejemplo:

```twig
Hello {{ user_name }}
```

Imaginemos que el usuario introduce como nombre de usuario la cadena `<script>alert('hello!')</script>`, el resultado del renderizado de la plantilla sería:

```html
Hello <script>alert('hello!')</script>
```

Esto ejecutaría una sentencia en JS, lo cual podría llegar a ser un potencial ataque.

La **solución** ante este problema es usar símbolos de escape, habilitando `output escaping` la salida sería una **cadena** que **no puede ejecutar un código JS**:

```html
Hello &lt;script&gt;alert(&#39;hello!&#39;)&lt;/script&gt;
```

Escape de cadenas en Twig:

```twig
{{ article.body|raw }}
```

##Debug

Cuando usamos `dump()` para ver el contenido de una variable en **PHP** la salida aparece en la barra de herramientas de la web de la version desarrollo.

```php
// src/AppBundle/Controller/ArticleController.php
namespace AppBundle\Controller;

// ...

class ArticleController extends Controller
{
    public function recentListAction()
    {
        $articles = ...;
        dump($articles);

        // ...
    }
}
```

En **Twig**
```twig
{# app/Resources/views/article/recent_list.html.twig #}
{{ dump(articles) }}

{% for article in articles %}
    <a href="/article/{{ article.slug }}">
        {{ article.title }}
    </a>
{% endfor %}
```

##Comprobando Sintaxis

Usando el siguiente comando veremos si los archivos twig tienen errores sintácticos.
```bash
# Comprobar un archivo
$ php bin/console lint:twig app/Resources/views/article/recent_list.html.twig

# Comprobar un directorio
$ php bin/console lint:twig app/Resources/views
```

##Formatos de Plantillas

Si quisieramos renderizar la salida en otro formato distinto al habitual, como por ejemplo podría ser JS, CSS, XML,  etc..., podemos hacerlo en el propio controlador.

Supongamos el caso de que una ruta `/contact` nos ofrece una respuesta **HTML** y nosotros queremos que nos ofrezca una respuesta **XML**, si establecemos la ruta a `/contact.xml` como vimos en la entrada del [capitulo 4](https://github.com/romanarranz/LFV/tree/master/tutorial/4_enrutamiento#ejemplo-avanzado) obtendremos el resultado esperado.

```php
public function indexAction(Request $request)
{
    $format = $request->getRequestFormat();

    return $this->render('article/index.'.$format.'.twig');
}
```

Un **enlace a este recurso en PDF** podría ser de la siguiente forma:

```twig
<a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
    PDF Version
</a>
```

##Conclusiones

El contenido de la aplicación o sitio web será dispuesta a través de las plantillas, éstas pueden tener distintos formatos XML, HTML, o cualquier otro formato. Las plantillas son una manera de generar el contenido de salida de un controlador usando el objeto `Response`.

```php
// objeto Response que devuelve el renderizado de una plantilla
$response = $this->render('article/index.html.twig');

// objeto Response que solo devuelve un string
$response = new Response('response content');
```

Los motores para crear plantillas con Symfony son **PHP** y **Twig**, ambos soportan herencia de plantillas, aunque por **simplicidad** destaca más **Twig**.

##Referencias

- [Lista de filtros Twig](http://twig.sensiolabs.org/doc/filters/index.html)
- [Lista de tags Twig](http://twig.sensiolabs.org/doc/tags/index.html)
- [Lista de variables globales](https://symfony.com/doc/current/cookbook/templating/global_variables.html)
- [Escape de cadenas](http://twig.sensiolabs.org/doc/api.html#escaper-extension)
- [Función dump()](https://symfony.com/doc/current/components/var_dumper/introduction.html#components-var-dumper-dump)

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
