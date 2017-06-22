# Magento para Desarrolladores
## Parte 1 - Introducción to Magento

#### Otros articulos en esta serie

* Parte 1 - Introducción a Magento
* Parte 2 - Configuración de Magento
* Parte 3 - Controllers en Magento
* Parte 4 - Layouts, Blocks y Templates en Magento
* Parte 5 - Models y ORM básicos en Magento
* Parte 6 - Recursos de instalación en Magento
* Parte 7 - ORM avanzados: Entidad Atributo Valor
* Parte 8 - Varien Data Collections

¿Que es Magento? Es la plataforma de comercio electrónico más potente del universo y está cambiando la cara del comercio electrónico para siempre. :-)

Por supuesto, ya lo sabes. Lo que quizás no sepas es que Magento también es un framework PHP orientado a objetos que puede usarse para desarrollar aplicaciones web modernas y dinámicas que aprovechan las poderosas carácteristcas de comercio electrónico de Magento.

Este es el primero de una serie de artículos en los que vamos a realizar tour a través de las características del Framework de Magento. No te preocupes si no entiendes todo inmediatamente. Al estudiar el sistema más todo lo que está en este artículo, todo comenzará a tener sentido, y pronto serás la envidia de tus colegas que estarán pegados trabajando con sistemas más primitivos de PHP.

#### En este articulo...

* Código organizado en Módulos
* MVC Básado en la Configuración (Configuration-Based)
* Controllers
* Carga de Modelos URI Basado en Contexto (Context-Based)
* Models
* Helpers
* Layouts
* Observers
* Sobre-escritura de Clases
* Wrap up

Or for the more visually oriented [Magento_MVC.pdf](http://devdocs.magento.com/common/images/m1x/Magento_MVC.pdf/).



## Código organizado en Módulos

Magento organiza su código en módulos individuales. En una típica aplicación [Model-View-Controller (MVC)](https://es.wikipedia.org/wiki/modelo-vista-controlador), todos los Controladores estarán en una carpeta, todos los Modelos en otra, etc. En Magento, los archivos se agrupan basados en su funcionalidad, la cual se denomina **modules** (modulos) en Magento.

### Código fuente de Magento

Por ejemplo, podrás encontrar Controllers, Models, Helpers, Blocks, etc. relacionados con la funcionalidad "checkout" de Magento en:

<pre><code>app/code/core/Mage/Checkout</code></pre>

Encontrarás Controllers, Models, Helpers, Blocks, etc. relacionados con la funcionalidad de "Google Checkout" de Magento en:

<pre><code>app/code/core/Mage/GoogleCheckout</code></pre>

### Tu propio código fuente

Si deseas personalizar o ampliar Magento, en lugar de editar directamente los archivos principales, o incluso si deseas colocar tus propios Controllers, Models, Helpers, Blocks, etc. junto al código de Magento, debes crear tus propios **Módulos** en:

<pre><code>app/code/local/Package/Modulename</code></pre>

**Package** (también conocido como un **Namespace**) es un nombre único que identifica a tu empresa u organización. La intención es que cada miembro de la comunidad Magento mundial utilice su propio nombre de **Package** al crear módulos para evitar duplicidades con el código de otro usuario.

Cuando crees un nuevo Módulo, debes informarle a Magento. Esto se hace agregando un archivo XML a la carpeta:

<pre><code>app/etc/modules</code></pre>

Hay dos tipos de archivos en esta carpeta, el primero habilita un módulo individual y se denomina en la forma: `Package_Modulename.xml`

El segundo es un archivo que te permitirá habilitar múltiples Módulos de un Package/Namespace, y se llama en la forma: `Package_All.xml`. Ten en cuenta que esto sólo es utilizado por el nucleo de Magento con el archivo `Mage_All.xml`. **No se recomienda** habilitar varios módulos en un solo archivo, ya que esto rompe la modularidad de Magento y los módulos.



## MVC Basado en la Configuración (Configuration-Based)

Magento es un sistema MVC basado en la configuración (**configuration-based**). La alternativa a esto sería un sistema MVC basado en convenciones (**convention-based**).

En un sistema MVC basado en convenciones (**convention-based**), si quisieras agregar, digamos, un nuevo controlador o quizás un nuevo modelo, simplemente crearías el archivo/clase y el sistema lo reconocería automáticamente.

En un sistema basado en la configuración (**configuration-based**), como Magento, además de agregar el nuevo archivo/clase al código base, a menudo es necesario explicar explícitamente al sistema acerca de la nueva clase o el nuevo grupo de clases. En Magento, cada módulo tiene un archivo denominado ``config.xml.`` Este archivo contiene toda la configuración relevante para un módulo de Magento. En tiempo de ejecución, todos estos archivos se cargan en un árbol de configuración grande.

Por ejemplo, ¿quieres usar Modelos en tu Módulo personalizado? Tendrás que agregar algún código a ``config.xml`` que le indique a Magento que quieres utilizar Modelos, así como el nombre de la clase base para todos tus Modelos.
```xml
<models>
    <packagename>
        <class>Package_Modulename_Model</class>
    <packagename>
</models>
 ```

 Lo mismo ocurre con Helpers, Blocks, Routes for your Controllers, Event Handlers, y muchos más. Casi siempre que quieras aprovechar el poder del sistema Magento, tendrás que hacer algún cambio o adición a tu archivo de configuración.

## Controlles

En cualquier sistema PHP, el punto de entrada principal de PHP sigue siendo un archivo PHP. Magento no es diferente, y ese archivo es ``index.php``.

Sin embargo, nunca escribas código en index.php. En un sistema MVC, index.php contendrá código/llamadas al código que hace lo siguiente:

1. Examinar la URL
2. Basado en un conjunto de reglas, convierte esta URL en una clase Controller y un método Action (denominado Routing)
3. Instancia la clase Controller y llama al método Action (llamado dispatching)

Esto significa que el punto de entrada práctico en Magento (o cualquier sistema basado en MVC) es un método en un archivo Controller. Considere la siguiente URL:

<pre><code>http://example.com/catalog/category/view/id/25</code></pre>

Cada parte de la ruta después del nombre del servidor se analiza de la siguiente manera.

### Front Name - ``catalog``

 La primera parte de la URL se llama *front name*. Esto, más o menos, indica a magento en qué Módulo puede encontrar un Controlador. En el ejemplo anterior, el *front name* es *catalog*, que corresponde al Módulo ubicado en:

 <pre><code>app/code/core/Mage/Catalog</code></pre>

### Controller Name - ``category``

La segunda parte de la URL le dice a Magento qué controlador debe usar. Cada módulo con controladores tiene una carpeta especial denominada 'controllers' que contiene todos los controladores de un módulo. En el ejemplo anterior, la categoría de la parte URL se traduce al archivo Controller

<pre><code>app/code/core/Mage/Catalog/controllers/CategoryController.php</code></pre>

La cual se verá así

```php
class Mage_Catalog_CategoryController extends Mage_Core_Controller_Front_Action
{
}
```

Todas las clases *Controller* de la aplicación Magento Cart extienden desde Mage_Core_Controller_Front_Action.

### Action Name - ``view``

El tercero en nuestra URL es el nombre de la acción. En nuestro ejemplo, es la palabra "view". La palabra "view" se utiliza para crear el Action Method. Así, en nuestro ejemplo, "view" se convertiría en "viewAction"

```php
class Mage_Catalog_CategoryController extends Mage_Core_Controller_Front_Action
{
    public function viewAction()
    {
        //main entry point
    }
}
```

Las personas familiarizadas con Zend Framework reconocerán la nomenclatura.

### Parameter/Value - ``id/25``

Cualquier parte de la ruta después del nombre de la acción (action name) seran consideradas como variables GET de la forma key/value. Así, en nuestro ejemplo, el "id/25" significa que obtendrá una variable GET llamada "id", con un valor de "25".

Como se mencionó anteriormente, si deseas que tu Módulo utilice Controladores, deberás configurarlos previamente. A continuación se muestra el bloque de configuración que habilita los Controladores del Módulo de ``catalog``

```xml
<frontend>
    <routers>
        <catalog>
            <use>standard</use>
            <args>
                <module>Mage_Catalog</module>
                <frontName>catalog</frontName>
            </args>
        </catalog>
    </routers>
</frontend>
```

No te preocupes demasiado por los detalles en este momento, pero observa el ``<frontName>catalog</frontName>``

Esto es lo que enlaza un Módulo con el *frontname* de la URL. La mayoría de los Módulos del núcleo de Magento eligen un *frontname* que tenga el mismo nombre que sus Módulos, pero esto no es necesario.

### Múltiples enrutamientos

El enrutamiento descrito anteriormente es para la aplicación Magento cart (a menudo llamada el frontend). Si Magento no encuentra un Controller/Action válido para una URL, intenta de nuevo, esta vez utilizando un segundo conjunto de reglas de enrutamiento para la aplicación de administración. Si Magento no encuentra un **Admin** Controller/Action válido, utiliza un Controlador especial llamado ``Mage_Cms_IndexController``.

El CMS Controller comprueba el sistema de gestión de contenido de Magento para ver si hay contenido que debe cargarse. Si encuentra alguno, lo cargará, de lo contrario se mostrará al usuario una página 404.

Por ejemplo, la página principal de magento "index" utiliza el CMS Controller, que a menudo puede lanzar a los recién llegados a un bucle.



## Carga de Modelos URI Basado en Contexto (Context-Based)
