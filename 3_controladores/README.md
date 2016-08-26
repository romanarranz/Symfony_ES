Controladores
===========================

##¿Qué es un controlador?

Un controlador es una llamda a PHP que se encarga de gestionar una petición (request) HTTP y proporciona
una respuesta a dicha solicitud (response) en formato HTTP, JSON, XML, una imagen, una página de error 404 etc...

El objetivo principal es crear una respuesta a una petición.

##Peticiones, respuestas y ciclo de vida.

1. Cada petición ejecuta un controlador frontal (app.php o app_dev.php para producción) de Symfony que se encarga de llamar al Kernel del framework.
2. El Kernel consulta los enrutadores para saber a cual pasarle la solicitud.
3. El router captura la URL y el controlador pasa a gestionar la respuesta ante tal solicitud.
4. Se devuelve el objeto Response creado por el controlador al cliente.

##Trabajando con controladores

Un ejemplo simple del clásico **Hola mundo!** podría ser el siguiente:

```php
<?php
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;

class HelloController
{
    public function indexAction($name)
    {
        return new Response('<html><body>Hello '.$name.'!</body></html>');
    }
}
?>
```

**Importante:**
- Los controladores se llaman de la siguiente manera 'nombre'Controller, esto es así porque en los archivos
de enrutamiento se harán referencias a ellos únicamente indicando el nombre del controlador que en este caso es **'Hello'**.
- Cada acción de un controlador debe tener como sufijo **Action** que también puede ser referenciada por los enrutadores
mediante el nombre, en nuestro caso la referencia sería **index**.
- Y siempre tenemos que devolver un objeto **Response**, pero puede ser de muchos tipos como se mencionó anteriormente.

###Mapeando URLs a los controladores

Para ello tendremos que crear un archivo de enrutamiento que mapea la URL y el parámetro de entrada que vamos a pasarle
al controlador, una vez tenemos esto el controlador tendrá que devolver un Response, por ejemplo un HTML.

Archivo de **enrutamiento**
```yml
# app/config/routing.yml
hello:
    path:      /hello/{name}
    # uses a special syntax to point to the controller - see note below
    defaults:  { _controller: AppBundle:Hello:index }
```

Nuestro **Controlador**

```php
<?php
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class HelloController
{
    /**
     * @Route("/hello/{name}", name="hello")
     */
    public function indexAction($name)
    {
        return new Response('<html><body>Hello '.$name.'!</body></html>');
    }
}
?>
```

Si accedemos a http://localhost:8000/hello/roman Symfony ejecutará el método *indexAction($name)* del controlador y le
pasará como parámetro la cadena 'roman'.

###Paso de parámetros desde el enrutador al controlador

Veamos un ejemplo de como enviarle 2 parámtros utilizando las variables que recoge el enrutador y que este se las pase
al controlador.

Archivo **controlador**
```php
<?php
// src/AppBundle/Controller/HelloController.php
// ...

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class HelloController
{
    /**
     * @Route("/hello/{firstName}/{lastName}", name="hello")
     */
    public function indexAction($firstName, $lastName)
    {
        // ...
    }
}
?>
```

Archivo de **enrutamiento**

```yml
# app/config/routing.yml
hello:
    path:      /hello/{firstName}/{lastName}
    defaults:  { _controller: AppBundle:Hello:index }
```

**Consideraciones:**
- El orden de los parámetros recogidos por el router no es relevante en los métodos de php.
- Los métodos se encargan de implementar los parámetros del router. Si en el routing.yml le enviamos el parámetro
firstName y lastName, en el controlador deben aparecer estos de forma obligatoria y además cabe la posibilidad de añadir un parámetro opcional.
- También podemos elegir incluir un parámetro o bien excluirlo.

```php
<?php
// ... NO VALIDO
public function indexAction($firstName, $lastName, $foo)
{
    // ...
}

// ... VALIDO porque $foo es opcional
public function indexAction($firstName, $lastName, $foo = 'bar')
{
    // ...
}

// ... EXCLUSION de parametro
public function indexAction($firstName)
{
    // ...
}
?>
```
###Controlador base

Symfony nos permite también la opción de que podamos extender una clase controladora directamente de la clase Controller.php, de esta manera heredamos métodos útiles que contiene esta clase a los que llamamos **Servicios**. Entre ellos se encuentran renderizadores de plantillas de Twig, mensajes predefinidos etc...

```php
<?php
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class HelloController extends Controller
{
    // ...
}
?>
```

####Generar URLs

Podemos hacerlo llamando al método **generateUrl()** que genera una URL en función a una ruta.

####Redireccionamientos

Si queremos redireccionar a un usuario hacia otra página podemos usar el método **redirectToRoute()**, por defecto
esto método realiza un redireccionamiento de tipo **302 (temporal)**, para realizar un redireccionamiento de tipo **301 (permanente)** tenemos que añadir hasta un tercer parámetro.

Si quisiéramos redireccionar al usuario a una página externa usaremos **redirect()**

```php
<?php
public function indexAction()
{
	// redirectToRoute es equivalente a usar redirect() y generateUrl() a la vez
	// return $this->redirect($this->generateUrl('homepage'));

	// redireccionamiento temporal 302
    return $this->redirectToRoute('homepage');

    // redireccionamiento permanente 301
    return $this->redirectToRoute('homepage', array(), 301);

    // redireccionamiento a una página externa
    return $this->redirect('http://symfony.com/doc');
}
?>
```

En realidad lo que está haciendo **redirectToRoute()** es crear un objeto respuesta bajo una generación de URL.
Porque como hemos dicho antes, en Symfony siempre hay que devolver una respuesta (Response) ante una solicitud del usuario, ejemplo:
```php
<?php
use Symfony\Component\HttpFoundation\RedirectResponse;

public function indexAction()
{
    return new RedirectResponse($this->generateUrl('homepage'));
}
?>
```

####Renderizando Plantillas

Seguramente si vamos a mostrar contenido HTML haremos uso de plantillas y para que estas plantillas recojan todos los
parámetros necesarios para su correcto renderizado tenemos que pasarle estos contenidos, para ello usaremos **render()**:

```php
<?php
// renders app/Resources/views/hello/index.html.twig
return $this->render('hello/index.html.twig', array('name' => $name));
?>
```
Las **plantillas** se localizan siempre **dentro de la carpeta de 'view'** (vistas) pero pueden estar en subdirectorios, estos deben tener como nombre el nombre del bundle, es decir, si mi controlador se llama HelloController su carpeta en será *app/Resources/view/hello*.
```php
<?php
// renders app/Resources/views/hello/greetings/index.html.twig
return $this->render('hello/greetings/index.html.twig', array(
    'name' => $name
));
?>
```

####Accediendo a los Servicios
Como hemos mencionado anteriormente cuando extendemos de Controller.php heredamos sus **Servicios** , la lista de
servicios podemos encontrarla con el siguiente comando:

```bash
$ php bin/console debug:container
```

Podemos usar servicios de mail, base de datos, renderizado de plantillas etc...
```php
$templating = $this->get('templating');

$router = $this->get('router');

$mailer = $this->get('mailer');
```

####Manejando páginas de Error 404

Cuando no se encuentra un recurso normalmente se ofrece una respuesta HTTP bajo un código de respuesta 404, esto a fin
de cuentas es la gestión de una excepción.

```php
<?php
public function indexAction()
{
    // obtener un dato de la BD
    $product = ...;
    if (!$product) {
        throw $this->createNotFoundException('The product does not exist');
    }

    return $this->render(...);
}
?>
```
**createNotFoundException()** es un atajo para crear un objeto **NotFoundException()** que dispara una respuesta de error
404.

Aunque somos libres de elegir el código de error que más nos convenga enviando un código de error 500 (internal sever
error) y detallando el error

```php
<?php
throw new \Exception('Something went wrong!');
?>
```

Más adelante veremos como personalizar las páginas de error.

###Peticiones como parámetros del Controlador

Si necesitamos leer parámetros de una consulta, capturar una cabecera HTTP o acceder a una subida de archivo, podemos
manejar esos datos con el objeto **Request** añadiendolo como parámetro a un método de nuestro controlador:

```php
<?php
use Symfony\Component\HttpFoundation\Request;

public function indexAction($firstName, $lastName, Request $request)
{
    $page = $request->query->get('page', 1);

    // ...
}
?>
```

###Manejando Sesiones

Symfony por defecto guarda los atributos en una cookie usando las sesiones nativas de PHP. Las sesiones nos permitirán
manejar la infrmación de sesión de un usuario usando su navegador, un bot o un servicio web.

Para almacenar y capturar la información de sesión usaremos el método **getSessions()** del objeto **Request**, este
método devuelve una **SessionInterface** que nos permite gestionar la sesión.

```php
<?php
use Symfony\Component\HttpFoundation\Request;

public function indexAction(Request $request)
{
    $session = $request->getSession();

    // store an attribute for reuse during a later user request
    $session->set('foo', 'bar');

    // get the attribute set by another controller in another request
    $foobar = $session->get('foobar');

    // use a default value if the attribute doesn't exist
    $filters = $session->get('filters', array());
}
?>
```

###Notificaciones para el Usuario

Esto es lo que se denomina **mensajes 'flash'** en Symfony, se utilizan una vez sólo para la sesión del usuario y cuando expira desaparecen.

Por ejemplo si el usuario enviase datos a través de un formulario. Después de procesar el formulario creamos una notifcación para la sesión de usuario y luego lo redireccionamos a otra ruta. Ahora cuando el usuario acceda a la nueva
ruta queremos mostrarle ese mensaje, para esto último usamos una plantilla de Twig.

```php
<?php
use Symfony\Component\HttpFoundation\Request;

public function updateAction(Request $request)
{
    $form = $this->createForm(...);

    $form->handleRequest($request);

    if ($form->isValid()) {
        // procesamiento

		// $this->addFlash es equivalente a $this->get('session')->getFlashBag()->add
        $this->addFlash(
            'notice',						// notice, warning, error o la clave que queramos
            'Your changes were saved!'		// cadena con el mensaje
        );

        return $this->redirectToRoute(...);
    }

    return $this->render(...);
}
?>
```

Plantilla con el **mensaje**

```twig
{% for flash_message in app.session.flashbag.get('notice') %}
    <div class="flash-notice">
        {{ flash_message }}
    </div>
{% endfor %}
```

###Objetos Petición(Request) y Respuesta (Response)

La clase **Request** contiene una gran cantidad de atributos y métodos con visibilidad pública de la cual podemos obtener información:

```php
<?php
use Symfony\Component\HttpFoundation\Request;

public function indexAction(Request $request)
{
    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(array('en', 'fr'));

    // retrieve GET and POST variables respectively
    $request->query->get('page');
    $request->request->get('page');

    // retrieve SERVER variables
    $request->server->get('HTTP_HOST');

    // retrieves an instance of UploadedFile identified by foo
    $request->files->get('foo');

    // retrieve a COOKIE value
    $request->cookies->get('PHPSESSID');

    // retrieve an HTTP request header, with normalized, lowercase keys
    $request->headers->get('host');
    $request->headers->get('content_type');
}
?>
```

La clase **Response** es una abstracción de una respuesta HTTP, y también es una interfaz ya que encapsula el objeto **ResponseHeaderBag** que es el que contiene los métodos y atributos públicos al igual que Request.

Ejemplo de respuesta HTTP:

```php
<?php
use Symfony\Component\HttpFoundation\Response;

// create a simple Response with a 200 status code (the default)
$response = new Response('Hello '.$name, Response::HTTP_OK);

// create a JSON-response with a 200 status code
$response = new Response(json_encode(array('name' => $name)));
$response->headers->set('Content-Type', 'application/json');
?>
```

También como se menciono en la [entrada 2](https://github.com/romanarranz/LFV/tree/master/tutorial/2_creando_paginas#rutas-y-controladores) podemos devolver diferentes tipos de respuestas:

- Para json, [JsonResponse](https://symfony.com/doc/current/components/http_foundation/introduction.html#component-http-foundation-json-response)
- Para archivos, [BinaryFileResponse](https://symfony.com/doc/current/components/http_foundation/introduction.html#component-http-foundation-serving-files)
- Para flujos, [StreamedResponse](https://symfony.com/doc/current/components/http_foundation/introduction.html#streaming-response)

###Redireccionamiento a otro Controlador

Si queremos delegar la gestión en otro controlador (aunque no es usual) podemos en lugar de redirigir al navegador del usuario a otra página lo que haremos será una sub-petición "interna" que será enviada a otro controlador. Para ello usamos el método **forward()** que cómo no... devuelve un objeto **Response** del resultado de la llamada al otro controlador.

```php
<?php
public function indexAction($name)
{
    $response = $this->forward('AppBundle:Something:fancy', array(
        'name'  => $name,
        'color' => 'green',
    ));

    // ... further modify the response or return it directly

    return $response;
}
?>
```

El método del controlador destino podría ser algo como esto:

```php
public function fancyAction($name, $color)
{
    // ... create and return a Response object
}
```

Y al igual que con las rutas, el orden de los argumentos no es importante, tan solo su nombre de variable.

###Validar un token CSRF

En ocasiones cuando se usamos protección CSRF en los envíos de **formularios** y no queremos usar la funcionalidad básica de Symfony, como podría ser el caso de un **DELETE** podemos comprobar si es válido ese token usando un atajo al modulo de seguridad csrf:

```php
if ($this->isCsrfTokenValid('token_id', $submittedToken)) {
    // ... do something, like deleting an object
}

// isCsrfTokenValid() es igual que:
// $this->get('security.csrf.token_manager')->isTokenValid(
//     new \Symfony\Component\Security\Csrf\CsrfToken\CsrfToken('token_id', $token)
// );
```

Esto es una forma de hacer una llamada a un servicio de Symfony sin necesidad de incluir el use del módulo que en este
caso sería Symfony\Component\Security\Csrf\CsrfTokenManager.          

##Conclusiones

Siempre que creamos una página la lógica la contendrá el **Controlador**. En cada método del controlador podremos hacer
cualquier cosa siempre y cuando devolvamos una respuesta **Response** al usuario.

Para hacernos la vida más fácil es conveniente que los controladores extiendan de **Controller.php** para obtener
los métodos y atributos de utilidad para las tareas más elementales como por ejemplo renderizar las plantillas de
las páginas HTML con *render()*.

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
