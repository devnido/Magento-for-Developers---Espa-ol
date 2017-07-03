# Magento para Desarrolladores
## Parte 4 - Layouts en Magento

#### Otros articulos en esta serie

* [Parte 1 - Introducción a Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-1)
* [Parte 2 - Configuración de Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-2)
* [Parte 3 - Controllers en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-3)
* [Parte 4 - Layouts, Blocks y Templates en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-4)
* [Parte 5 - Models y ORM básicos en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-5)
* [Parte 6 - Recursos de instalación en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-6)
* [Parte 7 - ORM avanzados: Entidad Atributo Valor](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-7)
* [Parte 8 - Varien Data Collections](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-8)

Los desarrolladores nuevos en Magento suelen confundirse con el sistema de presentación y visualización. Este artículo echará un vistazo al enfoque de Layout / Bloque de Magento y le mostrará cómo encaja en la cosmovisión de Magento MVC.

A diferencia de muchos sistemas MVC populares, el controlador de acciones de Magento no pasa un objeto de datos a la vista ni define propiedades en el objeto de vista (con algunas excepciones). En su lugar, el componente View hace referencia directa a modelos de sistema para obtener la información que necesita para su visualización.

Una consecuencia de esta decisión de diseño es que la vista se ha separado en bloques y plantillas. Los bloques son objetos PHP, las plantillas son archivos RAW "crudos" (con una extensión .phtml) que contienen una mezcla de HTML y PHP (donde PHP se usa como un lenguaje de plantilla). Cada bloque está vinculado a un único archivo de plantilla. Dentro de un archivo phtml, la palabra clave PHP $this contendrá una referencia al objeto Block de la plantilla.

## Ejemplo rápipo

Eche un vistazo a la plantilla de producto predeterminada en el archivo en

<pre>app/design/frontend/base/default/template/catalog/product/list.phtml</pre>

Verá el siguiente código de plantilla de PHP.

```php
<?php $_productCollection=$this->getLoadedProductCollection() ?>
<?php if(!$_productCollection->count()): ?> <div class="note-msg">
    <?php echo $this->__("There are no products matching the selection.") ?>
</div> <?php else: ?>
...
```

El método `getLoadedProductCollection` se puede encontrar en la clase Block de la plantilla, `Mage_Catalog_Block_Product_List` como se muestra a continuación:

<pre>File: app/code/core/Mage/Catalog/Block/Product/List.php</pre>

```php
...
public function getLoadedProductCollection()
{
    return $this->_getProductCollection();
}
...
```

La función `_getProductCollection` del bloque instancia los modelos y lee sus datos, devolviendo un resultado a la plantilla.

## Bloques Anidados

El verdadero poder de los Blocks/Templates viene con el método `getChildHtml`. Esto le permite incluir el contenido de un Bloque/Plantilla secundario dentro de un Bloque/Plantilla principal.

Bloques llaman Bloques llaman Bloques es cómo se crea el diseño HTML completo de la página. Echa un vistazo a la plantilla de una columna de diseño.

<pre>File: app/design/frontend/base/default/template/page/one-column.phtml</pre>


```HTML
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="?php echo $this->getLang() ?>" lang="<?php echo $this->getLang() ?>">
<head>
<?php echo $this->getChildHtml('head') ?>
</head>
<body class="page-popup <?php echo $this->getBodyClass()?$this->getBodyClass():'' ?>">
    <?php echo $this->getChildHtml('content') ?>
    <?php echo $this->getChildHtml('before_body_end') ?>
    <?php echo $this->getAbsoluteFooter() ?>
</body>
```

La propia plantilla tiene sólo 11 líneas de largo. Sin embargo, c*ada llamada a `$this->getChildHtml(...)` incluirá y renderizará otro bloque. Estos bloques, a su vez, utilizarán getChildHtml para renderizar otros bloques. Es bloques todo el camino hacia abajo.

## El Layout

Por lo tanto, los bloques y plantillas están bien y bien, pero probablemente se esté preguntando

1. ¿Cómo le digo a Magento qué bloques quiero usar en una página?
2. ¿Cómo le digo a Magento cuál Bloque debería comenzar a representar?
3. ¿Cómo especifico un bloque particular en `getChildHtml (...)`? Esas cadenas de argumentos no me parecen nombres de Bloque.

Aquí es donde el objeto de diseño entra en la imagen. El objeto de diseño es un objeto XML que definirá qué bloques se incluyen en una página y qué bloque (s) debe iniciar el proceso de representación.

La última vez que hicimos eco del contenido directamente de nuestros métodos de acción. Esta vez vamos a crear una sencilla plantilla HTML para nuestro módulo Hello World.

Primero, cree un archivo en

<pre>app/design/frontend/base/default/layout/local.xml</pre>

Con el siguiente contenido

```xml
<layout version="0.1.0">
    <default>
        <block type="page/html" name="root" output="toHtml" template="magentotutorial/helloworld/simple_page.phtml" />
    </default>
</layout>
```

A continuación, cree un archivo en

<pre>app/design/frontend/base/default/template/magentotutorial/helloworld/simple_page.phtml</pre>

Con el siguiente contenido

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Hello World</title>
    <style type="text/css">
        body {
            background-color:#f00;
        }
    </style>
</head>
<body>

</body>
</html>
```

Por último, cada controlador de acción es responsable de iniciar el proceso de disposición. Necesitaremos agregar dos llamadas de método al método de acción.

```php
public function indexAction() {
    //remove our previous echo
    //echo 'Hello Index!';
    $this->loadLayout();
    $this->renderLayout();
}
```

Borre la memoria caché de Magento y vuelva a cargar la página del controlador Hello World. Ahora debería ver un sitio web con un fondo rojo brillante y un código HTML que coincida con lo que está en simple_page.phtml.

## Que esta pasando

Así que eso es un montón de encantamientos vudú y crípticos. Echemos un vistazo a lo que está pasando.

En primer lugar, deseará instalar el módulo Layoutviewer. Este es un módulo similar al módulo Configviewer que construyó en el artículo de Hello World que nos permitirá echar un vistazo a algunos de los componentes internos de Magento.

Una vez que haya instalado el módulo (similar a cómo se configura el módulo Configviewer), vaya a la siguiente URL

<pre>http://example.com/helloworld/index/index?showLayout=page</pre>

Este es el formato xml para su página/solicitud. Se compone de etiquetas `<block/>`, `<reference/>` y `<remove/>`. Cuando llama al método loadLayout de su controlador de acciones, Magento

1. Generate this Layout XML
2. Instanciar una clase de bloque para cada etiqueta `<block />`, buscar la clase usando el atributo de tipo de la etiqueta como una ruta de configuración global y almacenarla en la matriz `_blocks` interna del objeto de diseño, utilizando el atributo de nombre de la etiqueta como clave de matriz.
3. Si la etiqueta `<block/>` contiene un atributo de salida, su valor se agrega a la matriz interna `_output` del objeto de diseño.

Luego, cuando llama al método `renderLayout` en su controlador de acciones, Magento itera sobre todos los bloques de la matriz `_output`, utilizando el valor del atributo de salida como un método de devolución de llamada. Esto es siempre `toHtml`, y significa que el punto de partida para la salida será la plantilla de Block.

Las secciones siguientes cubrirán la forma en que se instancian los bloques, cómo se genera este archivo de disposición y terminan con el inicio del proceso de salida.

## Instanciación de bloques

Por lo tanto, dentro de un archivo XML de diseño, un <block /> tiene un "tipo" que es en realidad un nombre de clase agrupada URI

```
<block type="page/html" ...
<block type="page/template_links" ...    
```

Por lo tanto, dentro de un archivo XML de diseño, un <block /> tiene un "tipo" que es en realidad un nombre de clase agrupada URI

Pasaremos por `page/html` como ejemplo. En primer lugar, Magento busca el nodo de configuración global en el archivo

<pre>app/code/core/Mage/Page/etc/config.xml</pre>

y encuentra

```xml
<page>
    <class>Mage_Page_Block</class>
</page>
```

Esto nos da nuestro prefijo de clase base `Mage_Page_Block`. A continuación, se agrega la segunda parte del URI (`html`) al nombre de la clase para darnos nuestro último nombre de clase de bloque `Mage_Page_Block_Html`. Esta es la clase que será instanciada.


Si creamos un bloque con el mismo nombre que un bloque ya existente, la nueva instancia de bloque reemplazará a la instancia original. Esto es lo que hemos hecho en nuestro archivo local.xml desde arriba.

```xml
<layout version="0.1.0">
    <default>
        <block type="page/html" name="root" output="toHtml" template="magentotutorial/helloworld/simple_page.phtml" />
    </default>
</layout>
```

El bloque llamado `root` ha sido reemplazado por nuestro bloque, que apunta a un archivo de plantilla phtml diferente.

## Usando referencias

<reference name="" /> enganchará todas las declaraciones XML contenidas en un bloque existente con el nombre especificado. Los nodos <block /> contenidos se asignarán como bloques secundarios al bloque principal referenciado.

```xml
<layout version="0.1.0">
    <default>
        <block type="page/html" name="root" output="toHtml" template="page/2columns-left.phtml">
            <!-- ... sub blocks ... -->
        </block>
    </default>
</layout>
```

En un archivo de diseño diferente:

```xml
<layout version="0.1.0">
    <default>
        <reference name="root">
            <!-- ... another sub block ... -->
            <block type="page/someothertype" name="some.other.block.name" template="path/to/some/other/template" />
        </reference>
    </default>
</layout>
```

Aunque el bloque raíz se declara en un archivo XML de diseño separado, el nuevo bloque se agrega como un bloque secundario. Magento inicialmente crea una `page/html` Bloque llamado `root`. Luego, cuando encuentre la referencia con el mismo nombre (`root`), asignará el nuevo bloque `some.other.block.name` como un hijo del bloque `root`.


## Cómo se generan los archivos de diseño

Por lo tanto, tenemos una comprensión un poco mejor de lo que está pasando con el XML de diseño, pero ¿de dónde viene este archivo XML? Para responder a esta pregunta, necesitamos introducir dos nuevos conceptos; Handles y la disposición del paquete.

### Handles

Cada solicitud de página en Magento generará varios identificadores únicos. El módulo de vista de diseño puede mostrarle estos identificadores utilizando una URL algo así como

<pre>http://example.com/helloworld/index/index?showLayout=handles</pre>

Debería ver una lista similar a la siguiente (dependiendo de su configuración)

1. `default`
2. `STORE_bare_us`
3. `THEME_frontend_default_default`
4. `helloworld_index_index`
5. `customer_logged_out`

Cada uno de estos es un identificador. Los Hadles se establecen en una variedad de lugares dentro del sistema Magento. Los dos a los que queremos prestar atención son `default` y `helloworld_index_index`. El Handle `default` está presente en cada solicitud en el sistema Magento. El Handle `helloworld_index_index` se crea combinando el nombre de la ruta (helloworld), el nombre del controlador de acciones (index) y el Metodo de Acción (index) en un solo string. Esto significa que cada método posible en un controlador de acción tiene un identificador asociado con él.

Recuerde que "index" es el valor predeterminado Magento para los controladores de acción y los métodos de acción, por lo que la siguiente solicitud

<pre>http://example.com/helloworld/?showLayout=handles</pre>

También producirá un identificador llamado `helloworld_index_index`

## Package Layout

Usted puede pensar en el diseño del paquete similar a la configuración global. Es un archivo XML grande que contiene **todas las configuraciones de diseño posibles** para una instalación de Magento en particular. Echemos un vistazo a él usando el módulo Layoutview

<pre>http://example.com/helloworld/index/index?showLayout=package</pre>

Esto puede tomar un tiempo para cargar. Si su navegador se está ahogando en la representación XML, intente el formato de texto

<pre>http://example.com/helloworld/index/index?showLayout=package&showLayoutFormat=text</pre>

Debería ver un archivo XML muy grande. Este es el diseño del paquete. Este archivo XML se crea combinando el contenido de todos los archivos de diseño XML para el tema actual (o paquete). Para la instalación predeterminada

<pre>app/design/frontend/base/default/layout/</pre>

Detrás de las escenas hay `<frontend><layout><updates />` y `<adminhtml><layout><updates />` secciones de la configuración global que contiene nodos con todos los nombres de archivo a cargar para el área respectiva. Una vez combinados los archivos enumerados en la configuración, Magento se fusionará en un último archivo xml, local.xml. Este es el archivo donde puede agregar sus personalizaciones a su instalación de Magento.

## Combinando Handles y Package Layout

Por lo tanto, si observa el Package Layout, verás algunas etiquetas conocidas como `<block />` y `<reference />`, pero todas están rodeadas de etiquetas que se ven así

```xml
<default />
<catalogsearch_advanced_index />
etc...
```

Estas son todas las etiquetas Handle. El diseño de una solicitud individual se genera mediante la captura de todas las secciones del diseño del paquete que coinciden con cualquier identificador de la solicitud. Por lo tanto, en nuestro ejemplo anterior, nuestro diseño se genera mediante la captura de etiquetas de las siguientes secciones


```xml
<default />
<STORE_bare_us />
<THEME_frontend_default_default />
<helloworld_index_index />
<customer_logged_out />
```

Hay una etiqueta adicional que debe tener en cuenta en la presentación del Package Layout. La etiqueta `<update />` le permite incluir las etiquetas de otro Handle. Por ejemplo

```xml
<customer_account_index>
    <!-- ... -->
    <update handle="customer_account"/>
    <!-- ... -->
</customer_account_index>
```

Está diciendo que las solicitudes con un identificador `customer_account_index` deben incluir `<blocks />` s del `<customer_account />` Handle.

## Aplicando lo que hemos aprendido

OK, eso es mucha teoría. Vamos a volver a lo que hicimos antes. Sabiendo lo que sabemos ahora, agregando

```xml
<layout version="0.1.0">
    <default>
        <block type="page/html" name="root" output="toHtml" template="magentotutorial/helloworld/simple_page.phtml" />
    </default>
</layout>
```

A `local.xml` significa que hemos reemplazado la etiqueta "root". Con un bloque diferente. Colocándolo en el `<default />` Handle nos hemos asegurado de que este reemplazo ocurra para **cada solicitud de página** en el sistema. Probablemente no es lo que queremos.

Si vas a cualquier otra página de tu sitio de Magento, notarás que están en blanco o tienen el mismo fondo rojo que tu página de mundo hola. Cambiemos el archivo `local.xml` para que solo se aplique a la página de Hello World. Lo haremos cambiando el valor predeterminado para usar el identificador de nombre de acción completo (`helloworld_index_index`).

```xml
<layout version="0.1.0">
    <helloworld_index_index>
        <block type="page/html" name="root" output="toHtml" template="magentotutorial/helloworld/simple_page.phtml" />
    </helloworld_index_index>
</layout>
```

Elimine su caché de Magento y restaure el resto de sus páginas.

En este momento esto sólo se aplica a nuestro índice Método de Acción. Añadámoslo también al Método de Acción de Adiós. En el controlador de acciones, modifique la acción de despedida para que parezca

```php
public function goodbyeAction() {
    $this->loadLayout();
    $this->renderLayout();
}
```

Si carga la siguiente URL, notará que sigue recibiendo el diseño predeterminado de Magento.

<pre>http://example.com/helloworld/index/goodbye</pre>

Necesitamos agregar un Handle para el nombre completo de la acción (`helloworld_index_goodbye`) a nuestro archivo `local.xml`. En lugar de especificar un nuevo `<block />`, vamos a usar la etiqueta de actualización para incluir el handle `helloworld_index_index`.

```xml
<layout version="0.1.0">
    <!-- ... -->
    <helloworld_index_goodbye>
        <update handle="helloworld_index_index" />
    </helloworld_index_goodbye>
</layout>
```

Cargar las siguientes páginas (después de borrar la memoria caché de Magento) ahora debe producir resultados idénticos.

<pre>
http://example.com/helloworld/index/index
http://example.com/helloworld/index/goodbye
</pre>

## Inicio de salida y getChildHtml

En una configuración estándar, la salida se inicia en el bloque denominado root (porque tiene un atributo de salida). Hemos anulado la plantilla de root con nuestra propia

<pre>template="magentotutorial/helloworld/simple_page.phtml"</pre>

Las plantillas se hacen referencia desde la carpeta raíz del tema actual. En este caso, eso es

<pre>app/design/frontend/base/default</pre>

Así que necesitamos profundizar en nuestra página personalizada. La mayoría de las plantillas de Magento se almacenan en

<pre>app/design/frontend/base/default/templates</pre>

La combinación de esto nos da el camino completo

<pre>app/design/frontend/base/default/templates/magentotutorial/helloworld/simple_page.phtml</pre>

## Adición de bloques de contenido

Una simple página roja es bastante aburrida. Añada un poco de contenido a esta página. Cambie su `<helloworld_index_index />` Handle en `local.xml` para que se vea como el siguiente

```xml
<helloworld_index_index>
    <block type="page/html" name="root" output="toHtml" template="magentotutorial/helloworld/simple_page.phtml">
        <block type="customer/form_register" name="customer_form_register" template="customer/form/register.phtml"/>
    </block>
</helloworld_index_index>
```

Estamos agregando un nuevo bloque anidado dentro de nuestra raíz. Este es un bloque que se distribuye con Magento, y mostrará un formulario de registro de clientes. Anidando este bloque dentro de nuestro bloque de raíz, lo hemos hecho disponible para que lo insertemos en nuestra `plantilla simple_page.html`. A continuación, utilizaremos el método `getChildHtml` del bloque en nuestro archivo `simple_page.phtml`. Editar `simple_page.html` para que se vea así

```php
<body>
    <?php echo $this->getChildHtml('customer_form_register'); ?>
</body>
```

Borra tu caché de Magento y vuelve a cargar la página y deberías ver el formulario de registro del cliente en tu fondo rojo. Magento también tiene un bloque llamado top.links. Intentemos incluir eso. Cambie su archivo simple_page.html para que lea

```php
<body>
    <h1>Links</h1>
    <?php echo $this->getChildHtml('top.links'); ?>
</body>
```

Cuando vuelvas a cargar la página, te darás cuenta de que el título `<h1>Links</h1>` es renderizado, pero no hay nada que renderice para **top.links**. Eso es porque no lo agregamos a `local.xml`. El método `getChildHtml` sólo puede incluir bloques que se especifican como subbloque en el diseño. Esto permite que Magento sólo instancia los bloques que necesita y también le permite establecer plantillas de diferencia para bloques basadas en el contexto.

Vamos a agregar el bloque `top.links` a nuestro `local.xml`

```xml
<helloworld_index_index>
    <block type="page/html" name="root" output="toHtml" template="magentotutorial/helloworld/simple_page.phtml">
        <block type="page/template_links" name="top.links"/>
        <block type="customer/form_register" name="customer_form_register" template="customer/form/register.phtml"/>
    </block>
</helloworld_index_index>
```

Borre la caché y vuelva a cargar la página. Ahora debería ver el módulo **top.links**.

## Vamos a la acción

Hay un concepto más importante que cubrir antes de terminar esta lección, y esa es la etiqueta `<action />`. El uso de la etiqueta `<action />` nos permite llamar a los métodos PHP públicos de las clases de bloque. Así que en vez de cambiar la plantilla del bloque raíz reemplazando la instancia de bloque por la nuestra, podemos usar una llamada a `setTemplate`.

```xml
<layout version="0.1.0">
    <helloworld_index_index>
        <reference name="root">
            <action method="setTemplate">
                <template>magentotutorial/helloworld/simple_page.phtml</template>
            </action>
            <block type="page/template_links" name="top.links"/>
            <block type="customer/form_register" name="customer_form_register" template="customer/form/register.phtml"/>
        </reference>
    </helloworld_index_index>
</layout>
```

Este XML de diseño establecerá primero la propiedad de plantilla del bloque root y, a continuación, agregará los dos bloques que usamos como bloques secundarios. Una vez que borremos el caché, el resultado debería ser igual que antes. La ventaja de usar `<action />` es la misma instancia de bloque que se creó anteriormente y todas las otras asociaciones padre/hijo todavía existen. Por esta razón, esta es una forma más versátil de implementar nuestros cambios.

Todos los argumentos al método de la acción necesitan ser envueltos en un nodo hijo individual de la etiqueta `<action />`. El nombre de ese nodo no importa, solo el orden de los nodos. Podríamos haber escrito el nodo de acción del ejemplo anterior de la siguiente manera con el mismo efecto.

```xml
<action method="setTemplate">
    <some_new_template>magentotutorial/helloworld/simple_page.phtml</some_new_template>
</action>
```

Esto es sólo para ilustrar que los nombres de nodos de argumentos de la acción son arbitrarios.

## Conclusión

Eso cubre los fundamentos de Layout. Hemos cubierto las etiquetas `<block />`, `<reference />`, `<update />` y `<action />`, así como manejos de actualización de diseño como `<default />` y `<cms_index_index />`. Estos componen la mayor parte de la configuración de diseño utilizada en Magento. Si lo encuentra un poco desalentador, no se preocupe, rara vez tendrá que trabajar con diseños en un nivel tan fundamental. Magento proporciona una serie de diseños pre-construidos que pueden ser modificados y desollados para satisfacer las necesidades de su tienda. La comprensión de cómo funciona todo el sistema de Layout puede ser de gran ayuda cuando se están solucionando problemas de diseño o agregando nueva funcionalidad a un sistema Magento existente.
