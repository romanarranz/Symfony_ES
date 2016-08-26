Creando Páginas en Symfony
===========================

##Rutas y Controladores

Necesitaremos crear una *ruta* que es la parte que especifíca como queda la URL de entrada al controlador y el propio *controlador*

Accedemos a la carpeta my_project/src/AppBundle y creamos el archivo *LuckyController.php*

```php
<?php
// src/AppBundle/Controller/LuckyController.php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Response;

class LuckyController
{
    /**
     * @Route("/lucky/number")
     */
    public function numberAction()
    {
        $number = rand(0, 100);

        return new Response(
            '<html><body>Lucky number: '.$number.'</body></html>'
        );
    }

    /**
     * @Route("/api/lucky/number")
     */
    public function apiNumberAction()
    {
        $data = array(
            'lucky_number' => rand(0, 100),
        );

        return new Response(
            json_encode($data),
            200,
            array('Content-Type' => 'application/json')
        );
    }

    /**
     * @Route("/api/jsonresponse/lucky/number")
     */
    public function apiNumberActionJsonResponse()
    {
        $data = array(
            'lucky_number' => rand(0, 100),
        );

        // calls json_encode and sets the Content-Type header
        return new JsonResponse($data);
    }
}
?>
```

Formas de acceder a las rutas que hemos definido:
- http://localhost:8000/lucky/number obtendremos como respuesta un número aleatorio.
- http://localhost:8000/api/lucky/number obtendremos una respuesta en formato json
- http://localhost:8000/api/jsronresponse/lucky/number obtendremos una jsonresponse

##Rutas con parametros de entrada en la URL

Para llevar a cabo esta operacion necesitamos definir unas reglas en el archivo de rutas.
Vamos a usar la opcion de implementar esta tarea usando YAML.

```yaml
# app/config/routing.yml
lucky_number:
    path:     /lucky/number/{count}
    defaults: { _controller: AppBundle:Lucky:number }
```

Ahora, nuevamente actualizamos el controller y añadimos una nueva ruta con un parametro al final.

```php
<?php
// src/AppBundle/Controller/LuckyController.php
// ...

class LuckyController
{

	//...

    /**
     * @Route("/lucky/number/{count}")
     */
    public function numberActionParameter($count)
    {
        $numbers = array();
        for ($i = 0; $i < $count; $i++) {
            $numbers[] = rand(0, 100);
        }
        $numbersList = implode(', ', $numbers);

        return new Response(
            '<html><body>Lucky numbers: '.$numbersList.'</body></html>'
        );
    }

    // ...
}
?>
```

Para acceder a la nueva ruta que hemos definido:
- http://localhost:8000/lucky/number/10 y obtendremos como resultado 10 numeros aleatorios.

##Creando Plantillas

Las plantillas tienen como valor de retorno HTML de forma general. Para esta labor haremos que la clase
extienda de Controller y enlazaremos en el proceso una plantilla hecha con Twig.

Procedemos a crear la clase LuckyControllerTemplate.php

```php
<?php
// src/AppBundle/Controller/LuckyControllerTemplate.php

namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class LuckyControllerTemplate extends Controller
{
	/**
     * @Route("/luckytemplate/number/{count}")
     */
    public function numberAction($count)
    {
        $numbers = array();
        for ($i = 0; $i < $count; $i++) {
            $numbers[] = rand(0, 100);
        }
        $numbersList = implode(', ', $numbers);

        $html = $this->container->get('templating')->render(
            'lucky/number.html.twig',
            array('luckyNumberList' => $numbersList)
        );

        return new Response($html);
    }
}

?>
```

Ahora necesitaremos crear la **vista** de la plantilla que en este caso es **lucky/number.html.twig**.

```twig
{# app/Resources/views/lucky/number.html.twig #}

{# Extiende del fichero app/Resources/views/base.html.twig que es una plantilla de html basica #}
{% extends 'base.html.twig' %}

{% block body %}
    <h1 style="color:red">Lucky Numbers: {{ luckyNumberList }}</h1>
{% endblock %}
```

Para acceder a la nueva ruta que hemos definido:
- http://localhost:8000/luckytemplate/number/10 y obtendremos como resultado 10 numeros aleatorios.


##Sobre los directorios

Nosotros principalmente trabajaremos en las carpetas:

####app/
Contiene aspectos de configuracion y plantillas con las vistas de las páginas.

####src/
Contiene los bundles que usemos en nuestro proyecto (*como plugins*), normalmente el código reside en la carpeta AppBundle.

####web/
Raiz de documentos donde residen los archivos accesibles como el CSS, imágenes y controladores del front-end que ejecutan
la aplicacion (app_dev.php y app.php)

####tests/
Test de unidad automaticos (parecido a JUnit).

####bin/
Archivos binarios como puede ser la consola de comandos de Symfony.

####var/
Directorio donde se crean automaticamente ficheros, estos pueden ser ficheros de caché (var/cache) y registros (var/logs/).

####vendor/
Librerias de terceros, paquetes y bundles son descargados aqui gracias a nuestro gestor de dependencias Composer. Nunca
se debería editar nada de este directorio.

##Configurando la Aplicación

Como podemos ver dentro del archivo *app/AppKernerl.php* podemos encontrar una lista de bundles que se instancian
en tiempo de ejecución y es posible añadir más.

El archivo de configuración principal de los bundles es *app/config/config.yml*.

```yml
# app/config/config.yml

# ...
framework:
    secret: '%secret%'
    router:
        resource: '%kernel.root_dir%/config/routing.yml'
    # ...

twig:
    debug:            '%kernel.debug%'
    strict_variables: '%kernel.debug%'

# ...
```
En este archivo podemos observar como la clave **framework** configura FrameworkBundle y del mismo modo **twig** configura
TwigBundle.

Si queremos comprobar que la configuración que está cargada es correcta podemos ejecutar el siguiente comando:

```bash
$ php bin/console config:dump-reference framework
```

Detallaremos más acerca de la configuración de Symfony en otra entrada.

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
