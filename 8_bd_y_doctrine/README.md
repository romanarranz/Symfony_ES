Bases de Datos y Doctrine
===========================

Symfony por defecto integra [Doctrine](http://www.doctrine-project.org) que es una librería que tiene como objetivo proporcionar un conjunto de herramientas para gestionar bases de datos.

Doctrine ORM nos permite hacer mapeos de objetos a una base de datos relacional como MySQL, PostgreSQL, Microsoft SQL o MongoDB.

## Ejemplo básico de Doctrine: `Producto`

Vamos a enumerar una serie de pasos para realizar una gestión de productos con Doctrine.

### Configurar la BD

Lo primero será configurar el conector a la base de datos en el archivo `app/config/parameters.yml`:

```yml
# app/config/parameters.yml
parameters:
    database_driver:    pdo_mysql
    database_host:      localhost
    database_name:      test_project
    database_user:      root
    database_password:  password

# ...
```

Esto se hace por convenio ya que una vez que se importe el archivo `app/config/parameters.yml` en el archivo de configuración global `app/config/config.yml` se referencian las variables.

```yml
# app/config/config.yml
imports:
    - { resource: parameters.yml }

//...

doctrine:
    dbal:
        driver:   '%database_driver%'
        host:     '%database_host%'
        dbname:   '%database_name%'
        user:     '%database_user%'
        password: '%database_password%'
```

Ahora que Doctrine se puede conectar a la BD el siguiente comando generará una base de datos `test_project` por nosotros a modo de prueba:

```bash
$ php bin/console doctrine:database:create
```

No debemos olvidar poner el charset a UTF8 en la BD si no queremos problemas con el juego de caracteres.
Para ello lo configuramos en el archivo `my.cnf`

```yml
[mysqld]
# Version 5.5.3 introduce "utf8mb4", que es mas recomendable
collation-server     = utf8mb4_general_ci # Reemplazamos utf8_general_ci
character-set-server = utf8mb4            # Reemplazamos utf8
```

Tenemos que indicarle también a Doctrine que las consultas las genere en función a esta codificación.

```yml
# app/config/config.yml
doctrine:
    charset: utf8mb4
    dbal:
        default_table_options:
            charset: utf8mb4
            collate: utf8mb4_unicode_ci
```

### Creando Entidades

Supongamos que queremos mostrar una serie de productos, tendremos que crear la clase Producto dentro del directorio `Entity` de nuestro `AppBundle`.

```php
// src/AppBundle/Entity/Product.php
namespace AppBundle\Entity;

class Product
{
    private $name;
    private $price;
    private $description;
}
```

Esta tarea la puede hacer también con el asistente de Doctrine

```bash
$ php bin/console doctrine:generate:entity
```

### Mapeando información

Doctrine nos permite hacer un mapeo directo de los datos en objetos y en vectores.

Los mapeos de información se pueden hacer en distintos formatos como YAML, XML o directamente en una clase PHP. Nosotros realizaremos los mapeos en PHP.

```php
// src/AppBundle/Entity/Product.php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="product")
 */
class Product
{
    /**
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=100)
     */
    private $name;

    /**
     * @ORM\Column(type="decimal", scale=2)
     */
    private $price;

    /**
     * @ORM\Column(type="text")
     */
    private $description;
}
```

**Importante:**

- Si vamos a crear un bundle no podemos tener mapeos mixtos es decir no podemos tener mapeos realizados con PHP y con YAML, sólo uno de ellos.
- El nombre de la tabla se obvia porque directamente se asocia al nombre de la clase.
- No debemos de poner palabras reservadas del SGBD como nombre de la clase ya que podemos obtener errores.

Para más información sobre consejos para hacer mapeos se dejan los siguientes enlaces.

### Generando Getters y Setters

Para encapsular la información necesitamos proporcionar en la clase `Producto` métodos `get` y métodos `set`.

Afortunadamente Symfony nos ofrece a través de su linea de comandos una utilidad para generar todos los get y set de una clase.

```bash
$ php bin/console doctrine:generate:entities --no-backup AppBundle/Entity/Product
```

Aunque lo ejecutemos n veces sólo nos generará los getters y setters necesarios sin generar duplicados.

**Nota:** obviamente si necesitamos añadir alguna funcionalidad extra a estos métodos tendremos que hacerlo manualmente.

**Más de doctrine:generate:entities**

- Genera un repositorio de clases con el nombre de las entidades con la anotación `@ORM\Entity(repositoryClass="...")`
- Genera los constructores en relación a la ariedad de sus relaciones 1:n , n:m.

Nos ahorra tiempo en caso de que tengamos que hacer un mapping de información en un bundle entero o en un namespace.

```php
# genera todas las entidades en el bundle AppBundle
$ php bin/console doctrine:generate:entities AppBundle

# genera todas las entidades en el namespace Acme
$ php bin/console doctrine:generate:entities Acme
```

## Crear bases de datos y tablas

Ahora mismo sabemos como gestionar la persistencia, pero en el caso en el que no tengamos definida la tabla `Producto` en el SGBD Doctrine nos proporciona una utilidad para esta realizar esta tarea a través del siguiente comando.

```bash
$ php bin/console doctrine:schema:update --force
```

Lo que hace el comando es comparar la base de datos con las entidades que tenemos definidas y ejecuta sentencias SQL para igualar las entidades a las tablas de la BD.
Por tanto si añadimos un nuevo atributo a `Producto` y ejecutamos el comando anterior se ejecutaran tantos "ALTER TABLE" como nuevas propiedades se tengan que añadir a la tabla.

Esto es realmente util si tuvieramos que hacer migraciones de un servidor a otro, ya que podriamos definir unas clases solo con los atributos y crear directamente las tablas con sus columnas con el comando en el nuevo servidor.

**Nota:** solo debe ser usado en el modo desarrollo, no en producción.

### Persistencia de Objetos en la BD

Dentro de un **controlador** es más simple definir las correspondencias entre la entidad y la tabla.

```php
// src/AppBundle/Controller/DefaultController.php

// ...
use AppBundle\Entity\Product;
use Symfony\Component\HttpFoundation\Response;

// ...
public function createAction()
{
    $product = new Product();
    $product->setName('Keyboard');
    $product->setPrice(19.99);
    $product->setDescription('Ergonomic and stylish!');

    $em = $this->getDoctrine()->getManager();

    // tells Doctrine you want to (eventually) save the Product (no queries yet)
    $em->persist($product);

    // actually executes the queries (i.e. the INSERT query)
    $em->flush();

    return new Response('Saved new product with id '.$product->getId());
}
```

**Nota:** para que este ejemplo funcione necesitamos crear una ruta al action que queremos ejecutar, dentro de routes.

**Importante:** cuando se llama al método `flush()` Doctrine mira todos los objetos que se les haya asignado para manejar su persistencia y comprueba si necesitan ser almacenados en la BD. Por ejemplo, si el objeto `$product` no existe en la BD, el **entity manager** ejecuta un `INSERT` y crea una nueva fila en la tabla `product` con la informacion.

Si tuvieramos 100 objetos `Product` y se llamara a `flush()`, Doctrine ejecutaria 100 sentencias `INSERT`, 1 por objeto.

### Captar objetos de la BD

## Referencias

- [Capa de abstracción de la BD: DBAL](https://symfony.com/doc/current/cookbook/doctrine/dbal.html)
- [Sentencias en strings sin ORM](https://symfony.com/doc/current/cookbook/doctrine/dbal.html)
- [Doctrine MongoDB](https://symfony.com/doc/current/bundles/DoctrineMongoDBBundle/index.html)
- [Juego de caracteres utf8mb4](https://dev.mysql.com/doc/refman/5.5/en/charset-unicode-utf8mb4.html)


**Mapping**
- [Tipado en Mapping](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#property-mapping)
- [Crear clases para la BD](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#creating-classes-for-the-database)
- [Palabras reservadas](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#quoting-reserved-words)
- [Mapeo adecuado](https://symfony.com/doc/current/book/doctrine.html#book-doctrine-field-types)

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
