# Magento para Desarrolladores
## Parte 1 - Introducción a Magento

#### Otros articulos en esta serie

* [Parte 1 - Introducción a Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-1)
* [Parte 2 - Configuración de Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-2)
* [Parte 3 - Controllers en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-3)
* [Parte 4 - Layouts, Blocks y Templates en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-4)
* [Parte 5 - Models y ORM básicos en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-5)
* [Parte 6 - Recursos de instalación en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-6)
* [Parte 7 - ORM avanzados: Entidad Atributo Valor](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-7)
* [Parte 8 - Varien Data Collections](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-8)

¿Que es Magento? Es la plataforma de comercio electrónico más potente del universo y está cambiando la cara del comercio electrónico para siempre. :-)

Por supuesto, ya lo sabes. Lo que quizás no sepas es que Magento también es un framework PHP orientado a objetos que puede usarse para desarrollar aplicaciones web modernas y dinámicas que aprovechan las poderosas carácteristcas de comercio electrónico de Magento.

Este es el primero de una serie de artículos en los que vamos a realizar tour a través de las características del Framework de Magento. No te preocupes si no entiendes todo inmediatamente. Al estudiar el sistema más todo lo que está en este artículo, todo comenzará a tener sentido, y pronto serás la envidia de tus colegas que estarán pegados trabajando con sistemas más primitivos de PHP.

#### En este articulo...

* [Código organizado en Módulos](#código-organizado-en-módulos)
* [MVC Básado en la Configuración (Configuration-Based)](#mvc-basado-en-la-configuración-configuration-based)
* [Controllers](#controllers)
* [Carga de Modelos URI Basado en Contexto (Context-Based)](#carga-de-modelos-uri-basado-en-contexto-context-based)
* [Models](#models)
* [Helpers](#helpers)
* [Layouts](#layouts)
* [Observers](#observers)
* [Sustitución de Clases](#sustitución-de-clases)
* [Conclusión](#conclusión)

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




## Controllers

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

Ahora que estamos en el punto de entrada del Action Method, queremos comenzar a instanciar las clases que hacen las cosas. Magento ofrece una forma especial de instanciar Models, Helpers y Blocks usando métodos estáticos los cuales están definidos en la clase global Mage. Por ejemplo:

```php
Mage::getModel('catalog/product');
Mage::helper('catalog/product');
```

El string 'catalog/product' se denomina un nombre de clase agrupado (Grouped Class Name). También se llama a menudo URI. La primera parte de cualquier nombre de clase agrupado (en este caso, catalog) se utiliza para buscar en qué Módulo reside la clase. La segunda parte ("product") se utiliza para determinar qué clase debe cargarse.

Por lo tanto, en ambos ejemplos anteriores, 'catalog' determina el Módulo `app/code/core/Mage/Catalog`.

Lo que significa que nuestro nombre de clase comenzará con `Mage_Catalog`.

A continuación, se agrega `product` para obtener el nombre final de la clase

```php
Mage::getModel('catalog/product');
Mage_Catalog_Model_Product

Mage::helper('catalog/product');
Mage_Catalog_Helper_Product
```

Estas reglas están vinculadas a cada módulo en su archivo de configuración. Cuando creas tu propio Módulo personalizado, tendrá tus propios nombres de clase agrupados (también llamados Grupos de Clases) para utilizar `Mage::getModel('myspecialprefix/modelname');`.

No es necesario utilizar nombres de clase agrupados (Grouped Class Name) para instanciar las clases. Sin embargo, como veremos más adelante, existen ciertas ventajas al hacerlo.



## Models

Magento, al igual que la mayoría de los frameworks actuales, ofrece un sistema de asignación de objetos relacional (Object Relational Mapping, ORM). Utilizando ORMs te olvidas de escribir SQL y te permiten manipular un almacén de datos puramente a través del código PHP. Por ejemplo:

```php
$model = Mage::getModel('catalog/product')->load(27);
$price = $model->getPrice();
$price += 5;
$model->setPrice($price)->setSku('SK83293432');
$model->save();
```

En el ejemplo anterior estamos llamando a los métodos "getPrice" y "setPrice" en nuestro product. Sin embargo, la clase Mage_Catalog_Model_Product no tiene métodos con estos nombres. Esto se debe a que el ORM de Magento usa el método mágico *__call* de PHP para implementar getters y setters.

Llamando al método `$product->getPrice();` obtendras el atributo "price" del Modelo.

Llamando ` $product->setPrice();` setearemos el atributo "price" del Modelo.

Con esto asumiremos que la clase Model no debería tiener los métodos llamados getPrice o setPrice. Si los tuviera, los métodos mágicos serán anulados. Si tienes interes en como están implementados estos métodos, puedes mirar la clase *Varien_Object*, de la cual heredan todos los Models.

Si quieres obtener todos los datos disponibles en un Modelo, lo puedes hacer con `$product->getData();` Obtendras un array con todos los atributos.

También podrás ver que es posible encadenar varias llamadas al método set: `$model->setPrice($price)->setSku('SK83293432');`

Esto se debe a que cada método "set" devuelve una instancia del Modelo. Este es un patrón que verás en gran parte del código base de Magento.

El ORM de Magento también contiene una forma de consultar varios objetos a través de una interfaz de colecciones. La sentencia a continuación nos devolverá una colección de todos los productos que cuestan $5.00

```php
$products_collection = Mage::getModel('catalog/product')
->getCollection()
->addAttributeToSelect('*')
->addFieldToFilter('price','5.00');
```

Una vez más, verás que Magento implementó una interfaz de encadenamiento. Las colecciones utilizan la biblioteca estándar de PHP para implementar objetos que tienen atributos de tipo array.

```php
foreach($products_collection as $product)
{
    echo $product->getName();
}
```

Puedes preguntarte para qué es el método "addAttributeToSelect". Magento tiene dos tipos de objetos Modelo. El primero es "Un Objeto, Una Tabla" el tradicional Modelo del tipo Active Record. Al instanciar estos Modelos, todos los atributos se seleccionan automáticamente.

El segundo tipo es un modelo definido como Entidad Atributo Valor (EAV). Los modelos de EAV distribuyen datos a través de varias tablas diferentes en la base de datos. Esto le da a Magento la flexibilidad de ofrecer un sistema de gestión para los atributos de los productos flexible sin tener que hacer un cambio en el esquema d elas tablas cada vez que quieras agregar un nuevo atributo. Al crear una colección de objetos EAV, Magento es conservador en el número de columnas que consultará, por lo que puedes utilizar "addAttributeToSelect" para obtener las columnas que quieras o `addAttributeToSelect ('*')` para obtener todas las columnas.



## Helpers

Los Helpers (Ayudantes) de Magento contienen métodos de utilidad que te permitirán realizar tareas comunes en objetos y variables. Por ejemplo:

```php
$helper = Mage::helper('catalog');
```

Notarás que hemos dejado la segunda parte del nombre agrupado de la clase. Cada Módulo tiene una clase Helper Data. lo cual equivale a:

```php
$helper = Mage::helper('catalog/data');
```

La mayoría de los Helpers heredan de Mage_Core_Helper_Abstract, de donde obtienen varios métodos útiles por defecto.

```php
$translated_output =  $helper->__('Magento is Great'); //gettext style translations
if($helper->isModuleOutputEnabled()): //is output for this module on or off?
```



## Layouts

Por lo tanto, hemos visto Controllers, Models y Helpers. En un típico sistema PHP MVC, después de haber manipulado nuestros Modelos, tendriamos que:

1. Establecer algunas variables para nuestra view
2. El sistema cargaría un Layout HTML exterior por defecto
3. Entonces el sistema cagaría nuestra vista dentro de ese Layout HTML exterior

Sin embargo, si analizamos un método de acción (Action Method) típico en algún Controller de Magento, no veras nada de esto:

```php
/**
 * View product gallery action
 */
public function galleryAction()
{
    if (!$this->_initProduct()) {
        if (isset($_GET['store']) && !$this->getResponse()->isRedirect()) {
            $this->_redirect('');
        } elseif (!$this->getResponse()->isRedirect()) {
            $this->_forward('noRoute');
        }
        return;
    }
    $this->loadLayout();
    $this->renderLayout();
}
```
En su lugar, la acción del controlador termina con dos llamadas

```php
$this->loadLayout();
$this->renderLayout();
```

Por lo tanto, la "V" en Magento MVC ya difiere de lo que probablemente estás acostumbrado, ya que aquí es necesario explícitamente iniciar la representación del diseño.

El Layout mismo también difiere. Un Layout en Magento es un objeto que contiene una colección anidada/árbol de objetos "Blocks". Cada objeto Block renderizará una parte específica del HTML. Los objetos Block hacen esto a través de una combinación de código PHP, e incluyendo archivos de Plantilla .phtml.

Los objetos Blocks están hechos para interactuar con el sistema Magento y obtener datos de los Modelos, mientras que los archivos de plantilla phtml producirán el HTML necesario para una página.

Por ejemplo, el Block encargado del encabezado de página `app/code/core/Mage/Page/Block/Html/Head.php` utiliza el archivo head.phtml ubicado en `page/html/head.phtml`.

Otra forma de ver esto es que las clases Blocks son casi como pequeños mini-controladores, y los archivos .phtml son las vistas.

Por defecto, cuando tu llamas a

```php
$this->loadLayout();
$this->renderLayout();
```

Magento cargará un Layout con la estructura esqueleto del sitio. Alli estarán los Structure Blocks (Bloques de Estructura) para generar el `html`, `head` y `body`, así como también configurará el Layout HTML para una o varias columnas. Además, habrá algunos Content Blocks (Bloques de Contenido) para la navegación, el mensaje de bienvenida predeterminado, etc.

"Estructura" y "Contenido" son designaciones arbitrarias en el sistema de Layout. Un bloque no sabe programáticamente si es estructura o contenido, pero es útil pensar en un bloque como uno u otro.

Para agregar contenido a un Layout, necesitas decirle a Magento algo así como:

```
"Oye, Magento, agrega estos Bloques adicionales abajo del Bloque 'content' del esqueleto"
```
o

```
"Oye, Magento, agrega estos Bloques adicionales debajo del Bloque 'left column' del esqueleto"
```

Esto se puede escribir en código dentro de la Acción del Controllador de la siguiente forma

```php
public function indexAction()
{
    $this->loadLayout();
    $block = $this->getLayout()->createBlock('adminhtml/system_account_edit')
    $this->getLayout()->getBlock('content')->append($block);
    $this->renderLayout();
}
```

Pero más comúnmente (al menos en la aplicación frontend cart), es el uso del sistema XML Layout.

```xml
<catalog_category_default>
    <reference name="left">
        <block type="catalog/navigation" name="catalog.leftnav" after="currency" template="catalog/navigation/left.phtml"/>
    </reference>
</catalog_category_default>
```

Le estamos diciendo que en el Módulo `catalog`, en el Controlador `category`, en la Acción `default`, inserte el Bloque `catalog/navigation` dentro del Bloque Estructura `left`, usando la plantilla `catalog/navigation/left.phtml`.

Otra cosa importante sobre los bloques. A menudo verás código en plantillas que se ve así:

`$this->getChildHtml('order_items')`

Así es como un bloque procesa un bloque anidado. Sin embargo, un bloque sólo puede representar un bloque secundario si el bloque secundario **se incluye como un Bloque anidado en el archivo XML del Layout**. En el ejemplo anterior, nuestro bloque de `catalog/navigation` no tiene bloques anidados. Esto significa que cualquier llamada a `$this->getChildHtml()` en `left.phtml` se mostrará en blanco.

En caso contrario, si tuvieramos algo así:

```xml
<catalog_category_default>
    <reference name="left">
        <block type="catalog/navigation" name="catalog.leftnav" after="currency" template="catalog/navigation/left.phtml">
            <block type="core/template" name="foobar" template="foo/baz/bar.phtml"/>
        </block>
    </reference>
</catalog_category_default>
```

Entonces ahora sí, podríamos llamar a `$this->getChildHtml('foobar');` dentro de `left.phtml`



## Observers

Como cualquier buen sistema orientado a objetos, Magento implementa un patrón de evento/observador para que los usuarios finales puedan utilizarlo. Como ciertas acciones ocurren durante una solicitud de página (un Modelo se guarda, un usuario inicia sesión, etc.), Magento emitirá una señal de evento.

Al crear tus propios Módulos, puedes "escuchar" estos eventos. Digamos que quisieras obtener un correo electrónico cada vez que un usuario inicia sesión en la tienda. Entonces, para ello deberías escuchar el evento "customer_login" (configurado en config.xml)

```xml
<events>
    <customer_login>
        <observers>
            <unique_name>
                <type>singleton</type>
                <class>mymodule/observer</class>
                <method>iSpyWithMyLittleEye</method>
            </unique_name>
        </observers>
    </customer_login>
</events>
```

Y luego escribir algún código que se ejecute cada vez que un usuario ha iniciado sesión:

```php
class Packagename_Mymodule_Model_Observer
{
    public function iSpyWithMyLittleEye($observer)
    {
        $data = $observer->getData();
        //code to check observer data for our user,
        //and take some action goes here
    }
}
```



## Sustitución de Clases

Por último, el sistema Magento ofrece la posibilidad de reemplazar las clases Model, Helper y Block de los Módulos nativos por tus propias clases. Esta es una característica similar a "Duck Typing" o "Monkey Patching" en un lenguaje como Ruby o Python.

Un ejemplo para ayudarte a entender. La clase Modelo para un producto es `Mage_Catalog_Model_Product`.

```php
$product = Mage::getModel('catalog/product');
```

Este es un patrón *Factory*

Lo que el sistema de Susutición de Clases de Magento hace es permitir decirle al sistema


```
"Hey, cuando alguien solicite catalog/product, en lugar de darles un Mage_Catalog_Model_Product,
entregales un Packagename_Modulename_Model_Foobazproduct".
```

Entonces, si lo deseas, tu clase Packagename_Modulename_Model_Foobazproduct puede extender de la clase product original

```php
class Packagename_Modulename_Model_Foobazproduct extends Mage_Catalog_Model_Product
{
}
```

Lo cual te permitirá cambiar el comportamiento de cualquier método de la clase, pero mantener la funcionalidad de los métodos existentes.

```php
class Packagename_Modulename_Model_Foobazproduct extends Mage_Catalog_Model_Product
{
    public function validate()
    {
        //add custom validation functionality here
        return $this;
    }

}
```

Como es de esperar, esta sustitución (o reescritura) se realiza en el archivo config.xml.

```xml
<models>
    <!-- does the override for catalog/product-->
    <catalog>
        <rewrite>
            <product>Packagename_Modulename_Model_Foobazproduct</product>
        </rewrite>
    </catalog>
</models>
```

Una cosa que es importante tener en cuenta aquí es que las clases individuales en **tu** Módulo están reemplazando clases individuales en **otros** Módulos. Sin embargo, no está sustituyendo todo el Módulo. Esto te permite cambiar el comportamiento del método específico sin tener que preocuparte por lo que está haciendo el resto de los metodos de la clase.



## Conclusión

Esperamos que hayas disfrutado de este tour relámpago a través de algunas de las características que el sistema Magento eCommerce ofrece a los desarrolladores. Puede ser un poco abrumador al principio, especialmente si esta es tu primera experiencia con un sistema PHP moderno orientado a objetos. Si comienzas a sentirte frustrado, respira hondo, recuerda que esto es nuevo, y las cosas nuevas son difíciles, pero al final del día es sólo una forma diferente de codificar. Una vez que sobrepases la curva de aprendizaje, te sentirás reacio a volver a otros sistemas menos potentes.
