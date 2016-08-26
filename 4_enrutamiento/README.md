Enrutamiento
===========================

Con Symfony es sencillo tener rutas bonitas como `/read/intro-to-symfony` en lugar de feas como `index.php?article_id=57`.

## Enrutamiento en acción

Una ruta es un mapeo de un *path* a un controlador (indica qué controlador y qué método ejecuta). Por ejemplo:

```php
<?php

// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class BlogController extends Controller
{
  /**
  * @Route("/blog/{slug}", name="blog_show")
  */
  public function showAction($slug)
  {
    // ...
  }
}

?>
```

El camino para cualquier página del blog sería `/blog/slug` donde `slug` sería la variable que utilizaría el controlador.

## Crear rutas

Symfony carga todas las rutas de `app/config/routing.yml` pero se puede cambiar para que sea otro archivo (incluso en otras extensiones como PHP o XML) cambiando en el archivo de configuración `app/config/config.yml` el valor de `resource` de `router`.

#### Configuración básica de una ruta

Definir una ruta es sencillo y normalmente una aplicación tiene muchas rutas. Una ruta contiene dos partes: el `path` (patrón que debe cumplir la URL) y un array de opciones llamado `defaults`.

En YAML:

```yaml
# app/config/routing.yml

_welcome:
    path:      /
    defaults:  { _controller: AppBundle:Main:homepage }
```

y en PHP:

```php
<?php

// src/AppBundle/Controller/MainController.php
// ...
class MainController extends Controller
{
  /**
  * @Route("/")
  */
  public function homepageAction()
  {
    // ...
  }
}

?>
```

#### Enrutando con variables

Ya se ha visto en el primer ejemplo de este capítulo.

#### Variables opcionales y obligatorias

Si se accediese a `/blog` veríamos que Symfony no ejecutaría el controlador. Eso es porque **todas las variables deben tener algún valor**. Aunque se puede solucionar dando **valores por defecto** a las variables en el array `defaults`.

Por ejemplo, si queremos que la variable sea el índice de la página y que cuando no escribamos nada vaya a la primera página tendríamos que hacer algo como esto:

```yaml
# app/config/routing.yml

blog:
    path:      /blog/{page}
    defaults:  { _controller: AppBundle:Blog:index, page: 1 }
```

URL     | Ruta  | Parámetros
------- | ----- | ----------
/blog   | blog  | {page} = 1
/blog/1 | blog  | {page} = 1
/blog/2 | blog  | {page} = 2

**ADVERTENCIA**: Se pueden definir varias variables opcionales, pero con una restricción, todas las variables que vayan detrás de una variable opcional también tiene que ser opcional. Así que para la ruta `/{page}/blog`, la variable `page` debe ser obligatoria, por tanto `/blog` no sería una URL válida.

**ADVERTENCIA:** Las rutas con variables opcionales al final no son válidas cuando la URL solicitada por el usuario tiene una `/` al final. Por lo que `/blog/` no sería una URL válida.

#### Requisitos en las variables

Imaginemos que tenemos el siguiente controlador donde la acción `index` devuelve una página y la acción `show` un artículo:

```php
<?php

// src/AppBundle/Controller/BlogController.php
// ...

class BlogController extends Controller
{
  /**
  * @Route("/blog/{page}", defaults={"page" = 1})
  */
  public function indexAction($page)
  {
    // ...
  }

  /**
  * @Route("/blog/{slug}")
  */
  public function showAction($slug)
  {
    // ...
  }
}

?>
```

Si quisiésemos acceder al artículo mediante `/blog/my-blog-post` daría lugar un fallo debido a que el enrutador coge la primera ruta que coincide y cogería la de mostrar una página, fallando porque la variable de entrada no es un número.

URL                 | Ruta  | Parámetros
------------------- | ----- | ----------
/blog/2             | blog  | {page} = 2
/blog/my-blog-post  | blog  | {page} = "my-blog-post"

La solución a este problema sería añadir un requisito a la variable y decirle que para las paǵinas solo capte enteros:

En PHP:

```php
<?php

// src/AppBundle/Controller/BlogController.php
// ...

/**
* @Route("/blog/{page}", defaults={"page": 1}, requirements={"page": "\d+"})
*/
public function indexAction($page)
{
  // ...
}

?>
```

En YAML:

```yaml
# app/config/routing.yml

blog:
    path:      /blog/{page}
    defaults:  { _controller: AppBundle:Blog:index, page: 1 }
    requirements:
        page:  \d+
```

URL                   | Ruta      | Parámetros
--------------------- | --------- | ----------
/blog/2               | blog      | {page} = 2
/blog/my-blog-post    | blog_show | {slug} = my-blog-post
/blog/2-my-blog-post  | blog_show | {slug} = 2-my-blog-post

Como el parámetro de los requisitos es una expresión regular, esto da mucha flexibilidad ya que si por ejemplo tenemos la página principal en dos idiomas y queremos que se muestre en uno u otro según los parámetros de la URL:

```php
<?php

// src/AppBundle/Controller/MainController.php
// ...

class MainController extends Controller
{
  /**
  * @Route("/{_locale}", defaults={"_locale": "es"}, requirements={"_locale": "es|en"})
  */
  public function homepageAction($_locale)
  {
    // ...
  }
}

?>
```

URL | Parámetros
--- | ---------
/   | {_locale} = "es"
/es | {_locale} = "es"
/en | {_locale} = "en"
/fr | *No daría ningún resultado*

#### Requisitos sobre el método HTTP

Podríamos crear una API para el blog con dos rutas, una para mostrar un post con peticiones `GET` o `HEAD` y otra para actualizarlo con `PUT`. Esto lo podríamos solucionar con la siguiente configuración en PHP:

```php
<?php

// src/AppBundle/Controller/MainController.php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
// ...

class BlogApiController extends Controller
{
    /**
     * @Route("/api/posts/{id}")
     * @Method({"GET","HEAD"})
     */
    public function showAction($id)
    {
        // ... return a JSON response with the post
    }

    /**
     * @Route("/api/posts/{id}")
     * @Method("PUT")
     */
    public function editAction($id)
    {
        // ... edit a post
    }
}
?>
```

y en YAML:

```yaml
# app/config/routing.yml
api_post_show:
    path:     /api/posts/{id}
    defaults: { _controller: AppBundle:BlogApi:show }
    methods:  [GET, HEAD]

api_post_edit:
    path:     /api/posts/{id}
    defaults: { _controller: AppBundle:BlogApi:edit }
    methods:  [PUT]
```

#### Condiciones personalizadas mediante expresiones

Se puede dotar de una flexibilidad casi ilimitada a las rutas usando condiciones:

```yaml
# app/config/routing.yml
contact:
    path:     /contact
    defaults: { _controller: AcmeDemoBundle:Main:contact }
    condition: "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'"
```

El valor de la propiedad `condition` es una expresión cuya sintaxis se puede consultar [aquí](http://symfony.com/doc/current/components/expression_language/syntax.html). Con esta configuración la ruta se ejecutará solo si el método es `GET` o `HEAD` y se navega desde Firefox.

La expresión puede ser tan compleja como se necesite y dentro se encuentran dos variables:
  * `context`: una instancia de la clase `Symfony/Component/Routing/RequestContext` y contiene toda la información básica de la ruta.
  * `request`: objeto del tipo `Symfony/Component/HttpFoundation/Request` que representa la petición del usuario.

**ADVERTENCIA:** Las condiciones no se tienen en cuenta al generar las URL a partir de las rutas.

#### Ejemplo avanzado

En PHP:

```php
<?php

// src/AppBundle/Controller/ArticleController.php

// ...
class ArticleController extends Controller
{
    /**
     * @Route(
     *     "/articles/{_locale}/{year}/{title}.{_format}",
     *     defaults={"_format": "html"},
     *     requirements={
     *         "_locale": "es|en",
     *         "_format": "html|rss",
     *         "year": "\d+"
     *     }
     * )
     */
    public function showAction($_locale, $year, $title)
    {
    }
}

?>
```

En YAML:

```yaml
# app/config/routing.yml
article_show:
  path:     /articles/{_locale}/{year}/{title}.{_format}
  defaults: { _controller: AppBundle:Article:show, _format: html }
  requirements:
      _locale:  es|en
      _format:  html|rss
      year:     \d+
```

Esta ruta solo se ejecutará si el valor de `_locale` es español o inglés, cuando la variable `year` sea un número. Además, en esta ruta se muestra **cómo utilizar un punto (`.`) para separar dos variables entre sí**, en vez de la barra (`/`). En resumen cualquiera de las siguientes URL se ejecutarían:
  * `/articles/en/2010/my-post`
  * `/articles/es/2010/my-post.rss`  
  * `/articles/en/2013/my-latest-post.html`

#### Variables de enrutamiento especiales

Symfony añade tres variables especiales de enrutamiento:
  * `_controller`: determina el controlador asociado a la ruta que se ejecuta.
  * `_format`: establece el formato de la petición del usuario.
  * `_locale`: establece el idioma en la petición del usuario.

## Nombre lógico de los controladores

Todas las rutas definen una variable `_controller` que indican el controlador que se debe ejecutar. El valor indica el controlador utilizando la siguiente notación `bundle:controlador:acción`. Por ejemplo un `_controller` con valor `AppBundle:Blog:show` significa

Bundle    | Clase del controlador | Nombre del método
--------- | --------------------- | -----------------
AppBundle | BlogController        | showAction

Symfony añade automáticamente la cadena `Controller` al controlador y `Action` a la acción.

El controlador sería como este:

```php
<?php

// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class BlogController extends Controller
{
    public function showAction($slug)
    {
        // ...
    }
}

?>
```

También puede referirse a este controlador como: `AppBundle\Controller\BlogController::showAction`. Pero el otro método es más conciso y permite una mayor flexibilidad.

**NOTA:** También se puede definir refiriéndose al controlador como un servicio, pero eso ya se verá más adelante.

## Variables de la ruta y argumentos del controlador

Todas las variables del array `defaults` se combinan con las variables de la ruta para formar un solo array disponible para los argumentos del método del controlador.

En el ejemplo avanzado anterior, cualquier combinación de las siguientes variables en cualquier orden se podría utilizar como argumentos para el método:
  * `$_locale`
  * `$year`
  * `$title`
  * `$_format`
  * `$_controller`

**NOTA:** Una variable especial adicional que siempre está disponible es `$_route` que indica el nombre de la ruta que se está ejecutando.

## Importar recursos de enrutamiento externos

Todas las rutas son cargadas por un solo archivo de configuración, normalmente `app/config/routin.yml`. Sin embargo, si se usan anotaciones de enrutamiento, necesitarás que el router apunte a los controladores con las anotaciones. Esto puede hacerse importando directorios en la configuración de enrutamiento:

```yaml
# app/config/routing.yml
app:
    resource: '@AppBundle/Controller/'
    type:     annotation # required to enable the Annotation reader for this resource
```

En este caso `resource` es un directorio por lo que cargaría todos los archivos de este.

También se podría apuntar a un solo archivo de configuración de enrutamiento:

```yaml
# app/config/routing.yml
app:
    resource: '@AcmeOtherBundle/Resources/config/routing.yml'
```

#### Prefijar las rutas importadas

También puedes dar un prefijo para las rutas importadas. Por ejemplo, supón que quieres prefijar todas las rutas en el AppBundle con `/site` (por ejemplo `/site/blog/{slug}`en lugar de`/blog/{slug}`):

```yaml
# app/config/routing.yml
app:
    resource: '@AppBundle/Controller/'
    type:     annotation
    prefix:   /site
```

El path de cada ruta cargado por el nuevo enrutamiento será prefijado con la cadena `/site`.

## Visualizar y depurar rutas

Conforme se añaden y configuran rutas, puede venir bien obtener información detellada sobre todas o alguna ruta específica. La mejor manera de ver todas las rutas que se han dfinido es ejecutando el siguiente comando en la carpeta raíz del proyecto:

```bash
$ php bin/console debug:router
```

Este comando imprimirá una lista de todas las rutas configuradas en tu aplicación:

```bash
homepage              ANY       /
contact               GET       /contact
contact_process       POST      /contact
article_show          ANY       /articles/{_locale}/{year}/{title}.{_format}
blog                  ANY       /blog/{page}
blog_show             ANY       /blog/{slug}
```

También se puede obtener información más específica de una ruta incluyendo el nombre de la ruta después del comando:

```bash
$ php bin/console debug:router article_show
```

Asímismo, si quires comprobar que una URL coincide con una ruta dada puedes usar:

```bash
$ php bin/console router:match /blog/my-latest-post
```

## Generar URL

El enrutamiento es un sistema bidireccional: convierte una URL con sus variables en un controlador y después convierte una ruta y varias variables en una URL. Este sistema se basa en los métodos `match()` y `generate()`. El siguietne código emplea la ruta `blog_show` definida con anterioridad:

```php
<?php

$params = $this->get('router')->match('/blog/my-blog-post');
// array(
//     'slug'        => 'my-blog-post',
//     '_controller' => 'AppBundle:Blog:show',
// )

$uri = $this->get('router')->generate('blog_show', array(
    'slug' => 'my-blog-post'
));
// /blog/my-blog-post

?>
```

Para generar URL, se necesita especificar el nombre de la ruta y el valor de todas las variables de la ruta (por ejemplo, `slug = my-blog-post`). Con esta información, puedes generar fácilmente cualquier URL.

```php
<?php

class MainController extends Controller
{
    public function showAction($slug)
    {
        // ...

        $url = $this->generateUrl(
            'blog_show',
            array('slug' => 'my-blog-post')
        );
    }
}

?>
```

Si el *front-end* de tu aplicación usa peticiones AJAX, seguramente necesitarás generar URL de Symfony desde el propio código JavaScript. Para ello debes utilizar un bundle desarrollado por terceros llamado `FOSJsRoutingBundle`:

```js
var url = Routing.generate(
    'blog_show',
    {"slug": 'my-blog-post'}
);
```

#### Generar URL con parámetros

El array que se pasa al método `generate` contiene los valores de las variables de la ruta, pero también puede contener variables adicionales que se añaden a la URL generarda en forma de *query-string*:

```php
<?php

$router->generate('blog', array('page' => 2, 'category' => 'Symfony'));
// /blog/2?category=Symfony

?>
```

#### Generar URL desde una plantilla

La mayoría de URL se generan en las propias plantillas para poder enlazar entre sí páginas de la aplicación. El formato es muy parecido al explicado anteriormente, pero en el caso de las plantillas Twig, se utilza una función especial llamada `path()`:

```twig
<a href="{{ path('blog_show', { 'slug': 'my-blog-post' }) }}">
  Read this blog post.
</a>
```

#### Generar URL absolutas

Por defecto, el sistema de enrutamiento genera URL relativas (`/blog`). Para generar URL absolutas, pasa el valor `true` como tercer parámetro del método `generate()`:

```php
<?php

$router->generate('blog_show', array('slug' => 'my-blog-post'), true);
// http://www.example.com/blog/my-blog-post

?>
```
En una plantilla, se generan de la siguiente forma:

```twig
<a href="{{ url('blog_show', { 'slug': 'my-blog-post' }) }}">
  Read this blog post.
</a>
```

**NOTA:** El host utilizado en las URL absolutas es el mismo de la petición actual y su valor se detecta automáticamente en función de la información proporcionada por PHP.

## Resumen

El enrutamiento es un sistema que permite mapear las URL de las peticiones a los controladores de tu aplicación que procesarán la petición. Gracias a este sistema se generar URL limpias y se desacopla el funcionamiento de la aplicación respecto a las URL.

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
