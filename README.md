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
* MVC Básado en Configuración (Configuration-Based)
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

Por ejemplo, podrás encontrar Controllers, Models, Helpers, Blocks, etc. relacionados con la funcionalidad "checkout" de Magento en

<code>
    app/code/core/Mage/Checkout
</code>
