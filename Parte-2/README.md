# Magento para Desarrolladores
## Parte 2 - Configuración de Magento

#### Otros articulos en esta serie

* [Parte 1 - Introducción a Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-1)
* [Parte 2 - Configuración de Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-2)
* [Parte 3 - Controllers en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-3)
* [Parte 4 - Layouts, Blocks y Templates en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-4)
* [Parte 5 - Models y ORM básicos en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-5)
* [Parte 6 - Recursos de instalación en Magento](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-6)
* [Parte 7 - ORM avanzados: Entidad Atributo Valor](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-7)
* [Parte 8 - Varien Data Collections](https://github.com/devnido/Magento-for-Developers-Spanish/tree/master/Parte-8)

La configuración es el corazón palpitante del Sistema Magento. Describe, en su totalidad, casi cualquier módulo, modelo, clase, plantilla, etc. de los que necesitará acceder. Es un nivel de abstracción que la mayoría de los desarrolladores de PHP no están acostumbrados a trabajar, y mientras añade tiempo de desarrollo en forma de confusión y rascar la cabeza, también le permite una cantidad sin precedentes de flexibilidad en cuanto a la superación de los comportamientos predeterminados del sistema.

Para empezar, vamos a crear un Módulo Magento que nos permitirá ver la configuración del sistema en nuestro navegador web. Sigue adelante copiando y pegando el código a continuación, vale la pena ir por tu propia cuenta como una manera de empezar a sentirte cómodo con las cosas que vas a hacer mientras trabajas con Magento, así como el aprendizaje de la terminología clave.

#### En este articulo...

* [Creación de la Estructura de Directorios del Módulo](#creación-de-la-estructura-de-directorios-del-módulo)
* [Configuración del Módulo](#configuración-del-módulo)
* [¿Que estoy viendo?](#que-estoy-viendo)
* [¿Porqué me importa?](#porqué-me-importa)



## Creación de la Estructura de Directorios del Módulo

Vamos a crear un Módulo Magento. Un Módulo es un grupo de archivos php y xml destinado a extender el sistema añadiendo nuevas funcionalidades, o anular el comportamiento del sistema central. Esto puede significar agregar modelos de datos adicionales para rastrear la información de ventas, cambiar el comportamiento de las clases existentes o agregar funciones totalmente nuevas.

Vale la pena señalar que la mayor parte del sistema base Magento se construye utilizando el mismo sistema de módulos que utilizarás. Si miras en

<pre><code>app/code/core/Mage</code></pre>

Cada carpeta es un módulo separado creado por el equipo de Magento. En conjunto, estos módulos forman el sistema de carrito que estás utilizando. Sus módulos deben colocarse en la siguiente carpeta

<pre><code>app/code/local/Packagename</code></pre>

"Packagename" debe ser una cadena única para Namespace/Package en tu código. Es una convención no oficial que este debe ser el nombre de tu empresa. La idea es elegir una cadena que nadie más en el mundo podría estar usando.

<pre><code>app/code/local/Microsoft</code></pre>

Nosotros usaremos "Magentotutorial".

Por lo tanto, para agregar un módulo a tu sistema Magento, crea la siguiente estructura de directorios

```
app/code/local/Magentotutorial/Configviewer/Block
app/code/local/Magentotutorial/Configviewer/controllers
app/code/local/Magentotutorial/Configviewer/etc
app/code/local/Magentotutorial/Configviewer/Helper
app/code/local/Magentotutorial/Configviewer/Model
app/code/local/Magentotutorial/Configviewer/sql
```

No necesitarás todas estas carpetas para cada módulo, pero crearlas todas ahora es una buena idea.

A continuación, hay dos archivos que necesitarás crear. El primero, `config.xml`, va en la carpeta `etc` que acabas de crear.

El segundo archivo debe crearse en la siguiente ubicación

<pre><code>app/etc/modules/Magentotutorial_Configviewer.xml</code></pre>

La convención de nombres para estos archivos es `Packagename_Modulename.xml`.

El archivo `config.xml` debe contener el siguiente código XML. Por ahora no te preocupes demasiado por todo lo que hace el código, ya llegaremos al final

```xml
<config>
    <modules>
        <Magentotutorial_Configviewer>
            <version>0.1.0</version>
        </Magentotutorial_Configviewer>
    </modules>
</config>
```

Por último, `Magentotutorial_Configviewer.xml` debe contener el siguiente código.

```xml
<config>
    <modules>
        <Magentotutorial_Configviewer>
            <active>true</active>
            <codePool>local</codePool>
        </Magentotutorial_Configviewer>
    </modules>
</config>
```

Eso es. Ahora tienes un módulo que no hará nada, pero que Magento reconocerá como tal. Para asegurarte de que has hecho las cosas bien, haz lo siguiente:

1. Borra la caché de Magento
2. En el Panel de Control de Magento, dirigete a **System->Configuration->Advanced**
3. En el panel "Disable modules output" busca `Magentotutorial_Configviewer`

Si lo encuentras, ¡Felicidades, has creado tu primer módulo de Magento!

## Configuración del Módulo

Por supuesto, este módulo todavía no hace nada. Cuando hayamos terminado, nuestro módulo hará

1. Comprobar la existencia de una variable Query String llamada "showConfig"
2. Si "showConfig" existe, mostrará nuestra configuración de Magento y detendrá la ejecución normal
3. Comprobar la existencia de una variable Query String adicional, "showConfigFormat" que nos permitirá especificar el formato de salida, ya sea texto plano o XML.

Primero, vamos a agregar la siguiente sección `<global>` a nuestro archivo config.xml.

```xml
<config>
    <modules>...</modules>
    <global>
        <events>
            <controller_front_init_routers>
                <observers>
                    <Magentotutorial_configviewer_model_observer>
                        <type>singleton</type>
                        <class>Magentotutorial_Configviewer_Model_Observer</class>
                        <method>checkForConfigRequest</method>
                    </Magentotutorial_configviewer_model_observer>
                </observers>
            </controller_front_init_routers>
        </events>
    </global>
</config>
```

A continuación, crea un archivo en

<pre><code>Magentotutorial/Configviewer/Model/Observer.php</code></pre>

Y pega el siguiente código dentro

```php
<?php
    class Magentotutorial_Configviewer_Model_Observer {
        const FLAG_SHOW_CONFIG = 'showConfig';
        const FLAG_SHOW_CONFIG_FORMAT = 'showConfigFormat';

        private $request;

        public function checkForConfigRequest($observer) {
            $this->request = $observer->getEvent()->getData('front')->getRequest();
            if($this->request->{self::FLAG_SHOW_CONFIG} === 'true'){
                $this->setHeader();
                $this->outputConfig();
            }
        }

        private function setHeader() {
            $format = isset($this->request->{self::FLAG_SHOW_CONFIG_FORMAT}) ?
            $this->request->{self::FLAG_SHOW_CONFIG_FORMAT} : 'xml';
            switch($format){
                case 'text':
                    header("Content-Type: text/plain");
                    break;
                default:
                    header("Content-Type: text/xml");
            }
        }

        private function outputConfig() {
            die(Mage::app()->getConfig()->getNode()->asXML());
        }
    }
```

Eso es. Borra la caché de Magento de nuevo y luego carga cualquier URL de Magento con la Query String `showConfig=true`

<code><pre>http://magento.example.com/?showConfig=true</pre></code>




## ¿Qué estoy viendo?

Lo que ves es archivo XML gigante que describe el estado de tu sistema Magento. Enumera todos los módulos, modelos, clases, escuchas de los eventos o casi cualquier cosa que podrías pensar.

Por ejemplo, considera el archivo config.xml que creaste anteriormente. Si buscas el texto `Configviewer_Model_Observer` en el archivo XML en tu navegador, encontrarás su clase en la lista. El archivo config.xml de cada módulo es analizado por Magento e incluido en la configuración global.



##¿Porqué me importa?

Ahora mismo esto puede parecer esotérico, pero esta configuración es clave para entender a Magento. Cada módulo que vas a crear se agregará a esta configuración, y en cualquier momento que necesites acceder a una parte de la funcionalidad principal del sistema, Magento consultará la confugiración para buscar cualquier cosa.

Un ejemplo rápido: Como desarrollador de MVC, probablemente has trabajado con algún tipo de clase de ayuda (Helper), instanciando algo como

´´´php
$helper_sales = new HelperSales();
´´´

Una de las cosas que Magento ha hecho es abstraer la declaración de clase PHP. En Magento, el código anterior parece algo así como

```php
$helper_sales = Mage::helper('sales');
```

En palabras simple, el metodo estático del Helper hará:

1. Buscar en la configuración la sección <helpers/>
2. Dentro de <helpers/>, buscará la sección <sales/>
3. En la sección <sales/> buscará la sección <class/>
4. Añadirá la parte después de la barra inclinada el valor que se encuentra en #3 (Faltan los `datos` en este caso)
5. Instanciar la clase que se encuentra en #4 (Mage_Sales_Helper_Data)


Si bien esto parece un montón de trabajo (y lo es), la ventaja clave es siempre mirar al archivo de configuración de los nombres de las clases, podemos anular la funcionalidad principal de Magento **sin** cambiar o añadir código al núcleo. Este nivel de meta-programación, que normalmente no se encuentra en PHP, le permite extender de forma limpia sólo las partes del sistema que necesita.
