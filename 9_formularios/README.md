Formularios
===========================

Trabajar con formularios es una de las tareas más comunes de un desarrollador web, Symfony tiene un componente llamado `Form` para trabajar con estos.

## Formulario sencillo

Imagina que estás construyendo una aplicación de lista de tareas. Los usuarios crearán y editarán tareas y para ello hará falta un formulario. Pero antes vamos a presentar la clase `Task` que representa y almacena los datos de una tarea:

```php
<?php
// src/AppBundle/Entity/Task.php
namespace AppBundle\Entity;

class Task
{
    protected $task;
    protected $dueDate;

    public function getTask()
    {
        return $this->task;
    }

    public function setTask($task)
    {
        $this->task = $task;
    }

    public function getDueDate()
    {
        return $this->dueDate;
    }

    public function setDueDate(\DateTime $dueDate = null)
    {
        $this->dueDate = $dueDate;
    }
}
?>
```

#### Construir el formulario

Ahora hay que crear y mostrar en el navegador el formulario HTML. En Symfony se hace construyendo un objeto de tipo `Form` y luego renderizándolo en una plantilla. Para empezar, se puede hacer estos dos pasos dentro de un controlador (más adelante se verá como hacerlo en una clase independiente que es lo recomendado):

```php
<?php
// src/AppBundle/Controller/DefaultController.php
namespace AppBundle\Controller;

use AppBundle\Entity\Task;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class DefaultController extends Controller
{
    public function newAction(Request $request)
    {
        // create a task and give it some dummy data for this example
        $task = new Task();
        $task->setTask('Write a blog post');
        $task->setDueDate(new \DateTime('tomorrow'));

        $form = $this->createFormBuilder($task)
            ->add('task', TextType::class)
            ->add('dueDate', DateType::class)
            ->add('save', SubmitType::class, array('label' => 'Create Task'))
            ->getForm();

        return $this->render('default/new.html.twig', array(
            'form' => $form->createView(),
        ));
    }
}
?>
```

Como se puede ver, crear un formulario con Symfony requiere poco código, porque los objetos `Form` se crean con un generador de formularios que se encarga del trabajo duro.

En el ejermplo, se han añadido dos campos al formulario (`task` y `dueDate`) que se corresponden con los atributos de la clase `Task`. También se asigna un tipo de dato a cada campo (`TextType` y `DateType`) para que Symfony sepa qué tipo de etiqueta HTML debe mostrar para cada campo. Finalmente, se añade un botón para enviar.

#### Mostrar el formulario

Una vez creado, hay que renderizarlo:

```twig
{# app/Resources/views/default/new.html.twig #}
{{ form_start(form) }}
{{ form_widget(form) }}
{{ form_end(form) }}
```

Tan solo hacen falta tres líneas para renderizarlo:
* `form_start(form)`: Renderiza la etiqueta de inicio del formulario, incluyendo el tipo de codificación cuando se usa para subir archivos.
* `form_widget(form)`: Renderiza todos los campos, lo que incluye el campo del elemento en sí, una etiqueta y cualquier mensaje de error en la validación.
* `form_end(form)`: Renderiza la etiqueta de final del formulario.

Más adelante se verá como renderizar cada campo de forma individual.

#### Procesar el envío del formulario

El segundo trabajo del formulario es traducir las entradas del usuario a propiedades del objeto. Para que esto ocurra, los datos enviados por el usuario deber ser escritos en el objeto `Form`.

```php
<?php
// src/AppBundle/Controller/DefaultController.php

// ...

use Symfony\Component\HttpFoundation\Request;

// ...

public function newAction(Request $request)
{
    // just setup a fresh $task object (remove the dummy data)
    $task = new Task();

    $form = $this->createFormBuilder($task)
        ->add('task', TextType::class)
        ->add('dueDate', DateType::class)
        ->add('save', SubmitType::class, array('label' => 'Create Task'))
        ->getForm();

    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        // ... perform some action, such as saving the task to the database

        return $this->redirectToRoute('task_success');
    }

    return $this->render('default/new.html.twig', array(
        'form' => $form->createView(),
    ));
}

// ...
?>
```

Este controlador sigue el patrón de envío de formularios que puede seguir uno de estos tres caminos:
1. Inicialmente solo se renderiza pues `handleRequest()` reconoce que `isSubmitted()` es `false` y por tanto no hace nada.
2. Cuando se envía `handleRequest()` comprueba si los datos son válidos. Si `isValid()` es `false` el formulario se renderiza con los errores de validación.
3. Cuando se envía y los datos son válidos se puede realizar cualquier acción con los datos.

#### Formulario con varios botones

Si el formulario tiene varios botones:

```php
<?php
$form = $this->createFormBuilder($task)
    ->add('task', TextType::class)
    ->add('dueDate', DateType::class)
    ->add('save', SubmitType::class, array('label' => 'Create Task'))
    ->add('saveAndAdd', SubmitType::class, array('label' => 'Save and Add'))
    ->getForm();
?>
```

Hay que comprobar dentro del controlador cuál se ha pulsado:

```php
<?php
if ($form->isValid()) {
    // ... perform some action, such as saving the task to the database

    $nextAction = $form->get('saveAndAdd')->isClicked()
        ? 'task_new'
        : 'task_success';

    return $this->redirectToRoute($nextAction);
}
?>
```

## Validación de formularios

La validación se realiza añadiendo un conjunto de reglas a una clase. Por ejemplo:

```yaml
# src/AppBundle/Resources/config/validation.yml
AppBundle\Entity\Task:
    properties:
        task:
            - NotBlank: ~
        dueDate:
            - NotBlank: ~
            - Type: \DateTime
```

Para más información, visitar [este enlace](http://symfony.com/doc/current/book/validation.html).

#### Grupos de validación

Suponer que tenemos una clase `User` usada tanto para cuando un usuario se registra como cuando actualiza su información:

```yaml
# src/AppBundle/Resources/config/validation.yml
AppBundle\Entity\User:
    properties:
        email:
            - Email: { groups: [registration] }
        password:
            - NotBlank: { groups: [registration] }
            - Length: { min: 7, groups: [registration] }
        city:
            - Length:
                min: 2
```

Con esta configuración hay tres grupos de configuración (`Default`, `User` y `registration` que solo tiene las restricciones del email y la contraseña).

Si usas grupos de validación, hay que usar:

```php
<?php
$form = $this->createFormBuilder($users, array(
    'validation_groups' => array('registration'),
))->add(...);
?>
```

Si además untilizas clases para los formularios (lo recomendado que se verá más adelante), hay que añadir el método `configureOptions()`:

```php
<?php
use Symfony\Component\OptionsResolver\OptionsResolver;

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => array('registration'),
    ));
}
?>
```

En ambos casos, solo el grupo devalidación `registration` será usado para validar el objeto.

#### Deshabilitar validación

A veces es útil suprimir la validación de todo el formulario:

```php
<?php
use Symfony\Component\OptionsResolver\OptionsResolver;

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => false,
    ));
}
?>
```

#### Grupos que dependen de los datos enviados

También se puede usar un *callback* en la opción `validation_groups` para determinar el grupo de validación que se aplica utilizando lógica PHP:

```php
<?php
use AppBundle\Entity\Client;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => function (FormInterface $form) {
            $data = $form->getData();

            if (Client::TYPE_PERSON == $data->getType()) {
                return array('person');
            }

            return array('company');
        },
    ));
}
?>
```

El código anterior hace que se ejecute el método estático `determineValidationGroups()` de la clase `Client` despues de que el procesamiento del formulario haya comenzado pero antes de que se realice la validación.

Este método recibe el objeto que representa el formulario como argumento. Se puede usar un `closure` en lugar del metodo de una clase para indicar toda la lógica:

```php
<?php
use AppBundle\Entity\Client;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => function (FormInterface $form) {
            $data = $form->getData();

            if (Client::TYPE_PERSON == $data->getType()) {
                return array('Default', 'person');
            }

            return array('Default', 'company');
        },
    ));
}
?>
```

#### Grupos que dependen del botón pulsado

Imagina que se tiene un asistente en el que se puede avanzar o volver atrás. Y si se vuelve atrás, los datos son guardados pero no validados. Los botones serían:


```php
<?php
$form = $this->createFormBuilder($task)
    // ...
    ->add('nextStep', SubmitType::class)
    ->add('previousStep', SubmitType::class)
    ->getForm();
?>
```

Se configuraría el botón de retroceder para que haga uso de un grupo de validación específico:


```php
<?php
$form = $this->createFormBuilder($task)
    // ...
    ->add('previousStep', SubmitType::class, array(
        'validation_groups' => false,
    ))
    ->getForm();
?>
```

## Tipos de campo predefinidos

Symfony inluye los siguientes tipos de campo:

* Campos de texto
  + TextType
  + TextareaType
  + EmailType
  + IntegerType
  + MoneyType
  + NumberType
  + PasswordType
  + PercentType
  + SearchType
  + UrlType
  + RangeType
* Campos de elección
  + ChoiceType
  + EntityType
  + CountryType
  + LanguageType
  + LocaleType
  + TimezoneType
  + CurrencyType
* Campos de fecha
  + DateType
  + DateTimeType
  + TimeType
  + BirthdayType
* Otros campos
  + CheckboxType
  + FileType
  + RadioType
* Campos de grupo
  + CollectionType
  + RepeatedType
* Campos ocultos
  + HiddenType
* Botones
  + ButtonType
  + ResetType
  + SubmitType
* Campos base
  + FormType

También se pueden [crear campos personalizados](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html).

#### Configurar campos del formulario

Cada tipo de campo tiene distintas opciones de configuración. Por ejemplo podemos hacer que se escriba la fecha en una caja de texto en lugar de en tres desplegables:

```php
<?php
->add('dueDate', DateType::class, array('widget' => 'single_text'))
?>
```

Dos de las opciones más usadas son `label` para el título del campo y `required` para establecer a falso si no se quiere que HTML5 valide el formulario.

## Deducir tipos de campo en el formulario

Cuando se añaden los datos de validación en la clase `Task`, Symfony puede deducir cual es el tipo de campo y no haría falta añadirlo en el `->add(...)`. Además del tipo de campo, también puede deducir algunas opciones de configuración.

## Renderizar formulario en una plantilla

Anteriormente se ha utilizado `form_widget(form)` para mostrar el formulario en una plantilla, pero normalmente se necesita más precisión:

```twig
{# app/Resources/views/default/new.html.twig #}
{{ form_start(form) }}
    {{ form_errors(form) }}

    {{ form_row(form.task) }}
    {{ form_row(form.dueDate) }}
{{ form_end(form) }}
```

Ya se explicaron `form_start(form)` y `form_end(form)`, pero qué significan los demás:
* `form_errors(form)`: Muestra errores globales del formulario (los específicos se muestran al lado de cada campo).
* `form_row(form.dueDate)`: Muestra el título, los errores si los hay y las etiquetas HTML necesarias para mostrar el campo indicado (`dueDate`). Por defecto, todos estos elementos se encierran dentro de un `<div>`.

Si se quiere que `form_row()` no muestre algunos de los elementos se tendrá que personalizar como se verá más adelante.

Para acceder a los datos del formulario se usa la notación `form.vals.value`. Por ejemplo `form.vars.value.task`.

#### Mostrar cada campo a mano

Para usar algo más preciso aún al `form_row()` se haría algo parecido a lo siguiente:

```twig
{{ form_start(form) }}
    {{ form_errors(form) }}

    <div>
        {{ form_label(form.task) }}
        {{ form_errors(form.task) }}
        {{ form_widget(form.task) }}
    </div>

    <div>
        {{ form_label(form.dueDate) }}
        {{ form_errors(form.dueDate) }}
        {{ form_widget(form.dueDate) }}
    </div>

    <div>
        {{ form_widget(form.save) }}
    </div>

{{ form_end(form) }}
```

Si el título generado automáticamente no es el que se quiere, se puede usar:

```twig
{{ form_label(form.task, 'Task Description') }}
```

En el siguiente ejemplo se le añade la calse `task_field` de CSS al elemento HTML para reperesentar el campo `task`:

```twig
{{ form_widget(form.task, {'attr': {'class': 'task_field'}}) }}
```

Para acceder a los valores individuales de cada campo (para `task`: `id`, `name` y `label`), se usaría, por ejemplo para el `id`:

```twig
{{ form.task.vars.id }}
```

Para obtener el valor del atributo name, usar mejor:

```twig
{{ form.task.vars.full_name }}
```

Para más información acerca de twig y los formularios, usar [este enlace](http://symfony.com/doc/2.0/reference/forms/twig_reference.html).

## Modificar la acción y el método de un formulario

Hasta ahora, se ha utilizado `form_start()` que por defecto redirige a la misma página y utiliza un método POST. Si se quiere cambiar esto se puede hacer de tres formas.

* **Primera opción - Si se crea el formulario en el controlador**:

```php
<?php
$form = $this->createFormBuilder($task)
    ->setAction($this->generateUrl('target_route'))
    ->setMethod('GET')
    ->add('task', 'text')
    ->add('dueDate', 'date')
    ->add('save', 'submit')
    ->getForm();
?>
```

* **Segunda opción - Si se usa una clase para el formulario** (recomendado):

```php
<?php
$form = $this->createForm(new TaskType(), $task, array(
    'action' => $this->generateUrl('target_route'),
    'method' => 'GET',
));
?>
```

* **Tercera opción - Desde la plantilla**:

```twig
{# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}
{{ form(form, {'action': path('target_route'), 'method': 'GET'}) }}

{{ form_start(form, {'action': path('target_route'), 'method': 'GET'}) }}
```

## Clases de formulario

Una clase para un formulario sería:


```php
<?php
// src/AppBundle/Form/Type/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->add('save', SubmitType::class)
        ;
    }
}
?>
```

Y se llamaría desde fuera así:


```php
<?php
// src/AppBundle/Controller/DefaultController.php
use AppBundle\Form\Type\TaskType;

public function newAction()
{
    $task = ...;
    $form = $this->createForm(TaskType::class, $task);

    // ...
}
?>
```

De esta forma se podría reutilizar el formulario.

**ADVERTENCIA**: Cuando se mapean los formularios a objetos, todos los campos son mapeados, así que si el formulario tiene un campo que no correponde al bojeto se produciría una excepción. Para poder lidiar con esto, por ejemplo cuando se añade la típica casilla de "Acepto los términos..." se teiene que especificar lo opción `mapped` a `false`:

```php
<?php
use Symfony\Component\Form\FormBuilderInterface;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add('task')
        ->add('dueDate', null, array('mapped' => false))
        ->add('save', SubmitType::class)
    ;
}
?>
```

Además si el formulario tiene campos que no se incluyen en los datos enviados por el usuario, a esos campos se le asignar el valor `null`. Se puede acceder a todos los datos de un controlador de la siguiente forma: `$form->get('dueDate')->getData();`. Además, los datos de cun campo no mapeado puede ser también modificado directamente: `$form->get('dueDate')->setData(new \DateTime());`.

#### Configurar el data_class

Todo formulario necesita conocer el nombre de la clase que se le pasa con los datos. No es necesario, pero cuando el proyecto se hace grande es una buena práctica añadir la opción `data_class` al formulario:

```php
<?php
use Symfony\Component\OptionsResolver\OptionsResolver;

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'data_class' => 'AppBundle\Entity\Task',
    ));
}
?>
```

## Formularios como servicios

Definir un formuario como servicio es una buena práctica que se puede realizar de la siguiente forma:

```php
<?php
// src/AppBundle/Form/Type/TaskType.php
namespace AppBundle\Form\Type;

use App\Utility\MyService;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class TaskType extends AbstractType
{
    private $myService;

    public function __construct(MyService $myService)
    {
        $this->myService = $myService;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // You can now use myService.
        $builder
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->add('save', SubmitType::class)
        ;
    }
}
?>
```

```yaml
# src/AppBundle/Resources/config/services.yml
services:
    app.form.type.task:
        class: AppBundle\Form\Type\TaskType
        arguments: ["@app.my_service"]
        tags:
            - { name: form.type }
```

## Formularios y Doctrine

El objetivo de un formulario es traducir datos de un tipo de objeto a un formulario HTML y después traducir las entradas del usuario en el objeto original. Así que la persistencia del objeto `Task` sería algo completamente ajeno a los formularios. No obstante, si has configurado la clase `Task` para persistirla a través de Doctrine (se le ha añadido metadatos para mapear las propiedades a la base de datos), entonces puedes persistirla despueś de comprobar que los datos de formulario son válidos:

```php
<?php
if ($form->isValid()) {
    $em = $this->getDoctrine()->getManager();
    $em->persist($task);
    $em->flush();

    return $this->redirect($this->generateUrl('task_success'));
}
?>
```

Si por alguna razón, no tienes acceso a tu objeto `$task` original, lo puedes recuperar a través de `$task = $form->getData();`.
