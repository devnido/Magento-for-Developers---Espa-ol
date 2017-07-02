# Magento para Desarrolladores
## Parte 3 - Controllers en Magento

#### Otros articulos en esta serie

* [Parte 1 - Introducción a Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-1)
* [Parte 2 - Configuración de Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-2)
* [Parte 3 - Controllers en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-3)
* [Parte 4 - Layouts, Blocks y Templates en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-4)
* [Parte 5 - Models y ORM básicos en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-5)
* [Parte 6 - Recursos de instalación en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-6)
* [Parte 7 - ORM avanzados: Entidad Atributo Valor](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-7)
* [Parte 8 - Varien Data Collections](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-8)

La arquitectura Model-View-Controller (MVC) remonta sus orígenes al lenguaje de programación Smalltalk y Xerox Parc. Desde entonces, ha habido muchos sistemas que describen su arquitectura como MVC. Cada sistema es ligeramente diferente, pero todos tienen el objetivo de separar el acceso a los datos, la lógica empresarial y el código de la interfaz de usuario entre sí.

La arquitectura de la mayoría de los Frameworks MVC de PHP se vé algo [así](http://alanstorm.com/2009/img/magento-book/php-mvc.png)

1. Una URL es interceptada por un solo archivo PHP (generalmente llamado un `FrontController`).
2. Este archivo PHP examinará la dirección URL para derivarla a un **Controlador** y una **Acción** (un proceso que se denomina enrutamiento).
3. El **Controlador** derivado es instanciado.
4. El nombre del método que coincide con el nombre de la **Acción** derivada se llama dentro del controlador.
5. Este método **Acción** instanciará y llamará a métodos de modelos, dependiendo de las variables de solicitud.
6. El método de **Acción** también preparará una estructura de datos de información. Esta estructura de datos se transmite a la vista.
7. La vista renderiza el HTML, utilizando la información en la estructura de datos que ha recibido del **Controlador**.

Si bien este patrón fue un gran paso adelante desde el patrón "cada archivo php es una página" establecido desde el principio, para algunos ingenieros de software, sigue siendo un hack feo. Las quejas más comunes son:

* El archivo PHP de Front Controller sigue funcionando en el espacio de nombres global.
* La convención sobre la configuración conduce a menos modularidad.
    * El enrutamiento de URL es a menudo inflexible.
    * Los controladores suelen estar vinculados a puntos de vista específicos.
    * Incluso cuando un sistema ofrece una forma de anular estos valores predeterminados, la convención lleva a aplicaciones en las que es difícil / imposible dejarlo en un nuevo modelo, vista o implementación de controlador sin re-factoring masivo.

Como probablemente has adivinado, el equipo de Magento comparte esta visión del mundo y ha creado un patrón MVC más abstracto que parece algo [así](http://alanstorm.com/2009/img/magento-book/magento-mvc.png):
1. Una URL es interceptada por un solo archivo PHP.
2. Este archivo PHP instancia una aplicación Magento.
3. La aplicación Magento instancia un objeto Front Controller.
4. Front Controller instancia cualquier número de objetos de enrutador (especificado en configuración global).
5. Los enrutador comprueban la URL de la solicitud hasta obtener un "match".
6. Si se encuentra una coincidencia, se derivan un controlador y una acción.
7. El controlador de acciones se instancia y se llama al nombre del método que coincide con el nombre de la acción.
8. Este método de acción instanciará y llamará a métodos en modelos, dependiendo de la solicitud.
9. Este controlador de acción instanciará entonces un objeto de diseño.
10. Este objeto de diseño, con base en algunas variables de solicitud y propiedades del sistema (también conocido como "identificadores"), crear una lista de objetos de bloque que son válidos para esta solicitud.
11. Layout también llamará a un método de salida en determinados objetos Block, que inician una renderización anidada (los bloques incluirán otros bloques).
12. Cada bloque tiene un archivo de plantilla correspondiente. Los bloques contienen la lógica de PHP, las plantillas contienen HTML y código de salida de PHP.
13. Los bloques se refieren directamente a los modelos para sus datos. En otras palabras, el controlador de acciones no les pasa una estructura de datos.

Mas adelante tocaremos cada parte de esta solicitud, pero por ahora estamos interesados en la sección **Front Controller -> Routers -> Action Controller**.


-------------------------------
## Hola mundo

Basta de teoría, es hora de Hello World. Iban a

1. Crear un módulo Hello World en el sistema Magento
2. Configure este módulo con rutas
3. Crear controlador (es) de acción para nuestras rutas

### Crear Módulo Hola mundo

En primer lugar, crearemos una estructura de directorios para este módulo. Nuestra estructura de directorios debe verse de la siguiente manera:

<code><pre>
app/code/local/Magentotutorial/Helloworld/Block
app/code/local/Magentotutorial/Helloworld/controllers
app/code/local/Magentotutorial/Helloworld/etc   
app/code/local/Magentotutorial/Helloworld/Helper
app/code/local/Magentotutorial/Helloworld/Model
app/code/local/Magentotutorial/Helloworld/sql
</pre></code>

A continuación, cree un archivo de configuración para el módulo (en path `app/code/local/Magentotutorial/Helloworld/etc/config.xml`):

```xml
<config>    
    <modules>
        <Magentotutorial_Helloworld>
            <version>0.1.0</version>
        </Magentotutorial_Helloworld>
    </modules>
</config>
```

A continuación, cree un archivo para activar el módulo (en la aplicación de ruta de acceso `app/etc/modules/Magentotutorial_Helloworld.xml`):

```xml
<config>
    <modules>
        <Magentotutorial_Helloworld>
            <active>true</active>
            <codePool>local</codePool>
        </Magentotutorial_Helloworld>
    </modules>
</config>
```

Finalmente, aseguramos que el módulo está activo:

1. Borre la caché de Magento.
2. En Magento Admin, vaya a System-> Configuration-> Advanced.
3. Expanda "Desactivar salida de módulos" (si no lo está).
4. Asegúrese de que Magentotutorial_Helloworld aparece.

### Configurando Rutas

A continuación, vamos a configurar una ruta. Una ruta convertirá una URL en un controlador de acciones y un método. A diferencia de otros sistemas PHP MVC basados en convenciones, con Magento necesitas definir explícitamente una ruta en la configuración global de Magento.

En el archivo config.xml (en la ruta de acceso app / code / local / Magentotutorial / Helloworld / etc / config.xml), agregue la siguiente sección:


```xml
<config>    
    ...
    <frontend>
        <routers>
            <helloworld>
                <use>standard</use>
                <args>
                    <module>Magentotutorial_Helloworld</module>
                    <frontName>helloworld</frontName>
                </args>
            </helloworld>
        </routers>  
    </frontend>
    ...
</config>
```

Tenemos una gran cantidad de terminología nueva aquí, vamos a desglosarla.

### ¿Que es <frontend>?

La etiqueta <frontend> se refiere a un área Magento. Por ahora, piensa en Areas como aplicaciones individuales de Magento. El "frontend" Área es el público frente a la aplicación Magento carrito de la compra. El área "admin" es la aplicación de la consola administrativa privada. La "instalación" Área es la aplicación que se utiliza para ejecutar a pesar de instalar Magento la primera vez.

### ¿Por qué las etiquetas <routers> si estamos configurando rutas individuales?

Hay una cita famosa sobre la informática, a menudo atribuida a Phil Karlton:

    "Sólo hay dos cosas difíciles en la informática: la invalidación de caché y el nombre de las cosas"

Magento, al igual que todos los grandes sistemas, sufre el problema de nombrar en espadas. Encontrarás que hay muchos lugares en la configuración global, y el sistema en general, donde las convenciones de nomenclatura parecen poco intuitivas o incluso ambiguas. Este es uno de esos lugares. A veces, la etiqueta <routers> incluirá información de configuración sobre enrutadores, otras veces incluirá información de configuración sobre los objetos del enrutador real que hacen el enrutamiento. Esto va a parecer contra intuitivo al principio, pero a medida que empiece a trabajar con Magento cada vez más, comenzará a entender su visión del mundo un poco mejor. (O, en las palabras de Han Solo, "Hey, confía en mí!").

### ¿Que es <frontName>?

Cuando un enrutador analiza una URL, se separa como sigue

<code><pre>http://example.com/frontName/actionControllerName/actionMethod/</pre></code>

Por lo tanto, al definir un valor de "hello world" en las etiquetas de <front Name>, le decimos a Magento que queremos que el sistema responda a las URLs en forma de

<code><pre>`http://example.com/helloworld/*`</pre></code>

Muchos desarrolladores nuevos en Magento confunden este nombre con el objeto Front Controller. No són la misma cosa. El frontName pertenece únicamente al encaminamiento.

### ¿Para qué sirve la etiqueta <hello world>?

Esta etiqueta debe ser la versión en minúscula de su nombre de módulo. Nuestro nombre del módulo es Helloworld, esta etiqueta es helloworld. Técnicamente esta etiqueta define el *nombre de nuestra ruta*

También notará que nuestro frontName coincide con nuestro nombre de módulo. Es una convención suelta que los nombres frontales coincidan con los nombres de los módulos, pero no es un requisito. En sus propios módulos, probablemente sea mejor usar un nombre de ruta que sea una combinación del nombre del módulo y el nombre del paquete para evitar posibles colisiones de espacio de nombres.

### ¿Para qué es <module> Magentotutorial_Helloworld </module>?

Esta etiqueta de módulo debe ser el nombre completo de su módulo, incluyendo su nombre de paquete / espacio de nombres. El sistema lo usará para localizar los archivos del controlador.

### Crear controlador (es) de acción para nuestras rutas

Un último paso por delante, y tendremos nuestro controlador de acción. Cree un archivo en

<code><pre>app/code/local/Magentotutorial/Helloworld/controllers/IndexController.php</pre></code>

Que contiene lo siguiente

```php
<?php
class Magentotutorial_Helloworld_IndexController extends Mage_Core_Controller_Front_Action {        
    public function indexAction() {
        echo 'Hello World';
    }
}
```

Borre la caché de configuración y cargue la siguiente URL

<code><pre>http://example.com/helloworld/index/index</pre></code>

También deberías poder cargar

<code><pre>
http://example.com/helloworld/index/
http://example.com/helloworld/
</pre></code>

Debería ver una página en blanco con el texto "Hello World". ¡Enhorabuena, has configurado tu primer controlador de Magento!

### ¿A dónde van los controladores de acción?

Los controladores de acción deben colocarse en la carpeta de los `controladores` del módulo (c minúscula). Aquí es donde el sistema los buscará.
