Configuración del Entorno
===========================

Una **aplicación o sitio web se compone de** una **colección de bundles**, cada uno de estos bundles puede ser configurado en YAML, XML o PHP.

Los archivos de configuración residen en `app/config`

```yml
# app/config/config.yml
imports:
    - { resource: parameters.yml }
    - { resource: security.yml }

framework:
    secret:          '%secret%'
    router:          { resource: '%kernel.root_dir%/config/routing.yml' }
    # ...

# Twig Configuration
twig:
    debug:            '%kernel.debug%'
    strict_variables: '%kernel.debug%'

# ...
```

Cada nivel de configuración superior define la configuración de un bundle, por ejemplo, la clave `framework` define la **configuración de *FrameworkBundle*** e incluye su enrutamiento, plantillas etc...

##Volcado de configuración por defecto

Podemos ver la configuración por defecto de un bundle en YAML usando el comando:

```bash
$ php bin/console config:dump-reference FrameworkBundle
```

o bien **mediante su clave**

```bash
$ php bin/console config:dump-reference framework
```

##Entorno

Tenemos un entorno de desarrollo `dev`, otro de producción `prod` que sería el resultado final y por último uno de pruebas llamado `test`.

Los **errores y alertas** de PHP sólo aparecen en el entorno `dev` mientras que en `prod` sólo apareceran los **errores**.

Para ver la aplicación en los distintos entornos simplemente tenemos que cambiar el controlador frontal:

**dev**
```bash
http://localhost/app_dev.php/random/10
```

**prod**
```bash
http://localhost/app.php/random/10
```

El entorno `prod` está optimizado para obtener mayor velocidad ya que las plantillas Twig y configuración de rutas se cachean. Por lo tanto si queremos ver cambios en la versión `prod` tenemos que limpiar la caché.

```bash
$ php bin/console cache:clear --env=prod --no-debug
```

Es más si abrimos el controlador frontal de la versión `prod`, es decir `web/app.php`, podremos ver como está configurado por defecto entorno.

```php
$kernel = new AppKernel('prod', false);
```

**NOTA**: cuando arrancamos el servidor con el comando `server:run` por defecto se carga el controlador frontal del entorno `dev` en `http://localhost:8000`.

##Configuración del Entorno

La clase PHP responsable de cargar la configuración del entorno es `app/AppKernel.php`

```php
// app/AppKernel.php
public function registerContainerConfiguration(LoaderInterface $loader)
{
    $loader->load(
        __DIR__.'/config/config_'.$this->getEnvironment().'.yml'
    );
}
```

Como cada entorno se carga a través de un archivo de configuración podríamos considerar la opción de tener un archivo de configuración específico para cada entorno.

```yml
# app/config/config_dev.yml

# cargo lo que habia en config.yml
imports:
    - { resource: config.yml }

framework:
    router:   { resource: '%kernel.root_dir%/config/routing_dev.yml' }
    profiler: { only_exceptions: false }

# ...
```

##Referencias

- [Cargar la configuración dentro del propio Bundle](https://symfony.com/doc/current/cookbook/bundles/extension.html)
- [Parametros externos de configuración](https://symfony.com/doc/current/cookbook/configuration/external_parameters.html)

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
