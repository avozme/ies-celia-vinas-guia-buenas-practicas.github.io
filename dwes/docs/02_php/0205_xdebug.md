---
layout: page
title: 2.5 Depurando el código PHP con xdebug
permalink: /php/depuracion-php-xdebug.html
nav_order: 5
has_children: false
parent: 2 Introducción a PHP
grand_parent: Desarrollo Web en Entorno Servidor
---

## 2.5. Depurando el código PHP con xdebug
{: .no_toc }

- TOC
{:toc}

Depurar el código PHP es, en principio, una tarea ardua, porque el programa se ejecuta en un servidor y nosotros solo podemos ver el resultado de esa ejecución en un cliente web. Esto significa que, si ocurre un error, nos lo encontraremos mucho después de que haya ocurrido. Es algo así como cuando la policía encuentra los restos de un crimen (el cadáver, algunas huellas y unos cuantos cristales rotos en la habitación) mucho después de que el asesinato haya ocurrido. Hay que ejercer de auténticos detectives para, a partir de esas pistas, tratar de recomponer qué es lo que ha podido suceder en el servidor que ha provocado ese error.

PHP ofrece varias funciones estándar para ayudarnos en la depuración pero, como vamos a ver, son bastante primitivas y poco funcionales, aunque pueden servirte en casos sencillos. Cuando la cosa se complica y el origen del error es difícil de rastrear, tendremos que recurrir a métodos más elaborados. Ahí entra en juego **xdebug**, el depurador más extendido en PHP.

### 2.5.1. Herramientas estándar de PHP para depuración

Como te decía, el propio lenguaje te ofrece varias funciones estándar para depurar errores. Las dos más utilizadas son:

* ***echo()***: La función de salida estándar de PHP también se usa con propósitos de depuración. De hecho, esto se ha hecho siempre en todos los lenguajes.

   Simplemente, si un fragmento de tu código PHP está fallando (una función, un método, un módulo o lo que sea), coloca unos cuantos *echo()* estratégicos para mostrar el valor de tus variables clave, las que pueden estar tomando valores incorrectos. Verás el resultado de esos *echo()* en tu navegador web y te puede proporcionar información valiosa sobre lo que puede estar sucediendo.

* ***print_r()***: La función *echo()* solo muestra variables simples, pero no objetos o arrays. Si necesitas mostrar el contenido de algo más complejo que una variable simple, puedes usar *print_r()*, que enviará el contenido de cualquier variable a la salida estándar, por lo que podrás verla en tu navegador web.

* ***var_dump()***: Esta función es como *print_r()*, pero muestra aún más información sobre tu variable, como su tamaño.

Además de estas funciones, los **archivos de registro (*logs*)** de tu servidor pueden mostrarte información importante sobre el error que se haya podido producir. La ubicación de los archivos *log* es diferente según tu sistema operativo, por lo que tendrás que investigar un poco acerca de qué archivos consultar.

En general, hay un montón de archivos *log* que tu sistema operativo podría estar produciendo, así que lo primero sería averiguar cómo se llaman los archivos de *log* que produce tu servidor web en concreto. Por ejemplo, **Apache** en Ubuntu Linux tiene su registro de errores en */var/log/httpd/error_log*.

Como es imposible proporcionarte una lista de todos los archivos de registro de todos los servidores web en todos los sistemas operativos, tendrás que bichear un poco por Internet para localizar el *log* de tu servidor en concreto.

### 2.5.2. xdebug

Todo eso está muy bien, pero tanto las funciones estándar de PHP como los archivos *log* del servidor nos proporcionan una imagen *a posteriori* de lo que ha sucedido en el servidor. Es decir, vemos lo que ha ocurrido cuando el programa ya ha finalizado su ejecución, lo que a menudo dificultad localizar los errores.

Además, usar funciones como *print_r()* o *var_dump()* implica introducir líneas de código de depuración dentro de mi programa. Ese código de depuración, poco a poco, va dejando *basura* que a veces se nos olvida eliminar. Nada hace peor efecto ante los usuarios que una aplicación web que de pronto muestra la salida de un *var_dump()* en mitad de una de sus vistas. Bueno, sí: un error de ejecución de PHP hace todavía peor efecto.

Por todo ello, sería estupendo poder depurar el código PHP como si se estuviera ejecutando en nuestra máquina, igual que con cualquier otro lenguaje de programación. Eso es posible gracias a ***xdebug***.

#### ¿Qué es xdebug?

***xdebug*** es una extensión de PHP que permite al cliente y al servidor comunicarse mediante un protocolo especial para depurar el código que se ejecuta en el servidor.

Es decir: nuestro código seguirá ejecutándose en el servidor, pero nuestro cliente (normalmente, nuestro editor de texto o nuestro IDE) podrá pedirle al servidor que lo ejecute paso a paso o que le comunique el valor de cualquier variable en ese momento.

#### Cómo instalar xdebug

Como es una extensión, ***xdebug*** no viene de serie y necesita ser instalada en el servidor.

La configuración más habitual del servidor web, es decir, un **Apache bajo Linux** (supondremos que es Debian o Ubuntu), necesita estos pasos para lograr la instalación de xdebug:

1. Instalar ***php-dev***, un paquete de herramientas de desarrollo para PHP: ```$sudo apt install php-dev```
2. Instalar ***xdebug*** a través de *composer* o de *pecl* (el precursor de *composer*): ```$sudo pecl install xdebug```
3. Habilitar el módulo xdebug en nuestro servidor: ```$sudo phpenmod xdebug```
4. Reiniciar el servidor: ```$sudo service apache2 restart```

Obviamente, estos pasos son diferentes en otros servidores y otros sistemas operativos, incluso en otros sistemas Linux. Es imposible mostrar aquí todas las configuraciones posibles de todos los sistemas, por lo que, nuevamente, tendrás que buscar cuál es la manera de instalar *xdebug* en tu servidor dependiendo de cuál sea tu configuración exacta.

(Incluso es posible que tengas suerte y tu servidor ya tenga *xdebug* instalado)

Después de habilitar *xdebug* en tu servidor, puedes comprobar que está funcionando ejecutando *phpinfo()* en cualquier script o escribiendo ```$php -i``` en tu servidor. En ambos casos debería aparecer una sección dedicada a *xdebug* con información sobre la configuración de esta extensión.

#### Cómo activar xdebug en mi IDE

Una vez instalada la extensión *xdebug*, llega el momento de usarla.

*xdebug* puede integrarse prácticamente con cualquier IDE medianamente decente. Nosotros vamos a ver cómo integrarla con ***Visual Studio Code***, que es probablemente el IDE más utilizado en la actualidad. Si utilizas otro entorno de desarrollo, tendrás que buscar por ahí cómo habilitar *xdebug* en tu IDE, pero los pasos serán bastante similares a estos:

1. **Instalar una *extensión*** adecuada para la integración de *xdebug* con VS Code. La más utilizada es una llamada ***PHP Debug***.
2. **Editar el archivo *.vscode/launch.json***. Este archivo contiene la configuración de *debugging*, es decir, la conexión con el componente *xdebug* de nuestro servidor. El archivo se crea automáticamente al instalar la extensión *PHP Debug* en VS Code, pero puede que tengas que cambiar algunas cosas. En concreto, tendrás que revisar:
   * El puerto en el que está escuchando el servidor. La depuración de PHP se hace a través de un protocolo diferente de http/https, así que usa unos puertos diferentes. Un puerto habitual es el 9003, pero debes revisar la configuración de tu servidor por si está usando un puerto diferente, y en tal caso indicarlo en el archivo *launch.json* (sección *port*).
   * El directorio del servidor donde está tu aplicación web instalada. Los archivos del servidor se mapearán con archivos locales de VS Code. En la sección *pathMappings* del archivo *launch.json* debes indicar dónde están los archivos dentro del servidor.

#### Cómo usar xdebug

Si ya tenemos instalado *xdebug* en el servidor y lo hemos activado correctamente en nuestro IDE o en nuestro editor de texto preferido, el proceso de depuración es bastante parecido al de cualquier otro lenguaje de programación.

Nuevamente, nos referiremos a VS Code, pero el funcionamiento de *xdebug* será muy semejante en otros entornos de desarrollo.

* Al ejecutar el código, VS Code nos mostrará las opciones para avanzar paso a paso, entrar dentro de funciones, saltar la ejecución de una función, continuar la ejecución sin parar o detener el programa.
* Podemos crear **puntos de ruptura** o ***breakpoints*** haciendo click a la izquierda de la línea de código donde queremos detener la ejecución de la aplicación.
* Podemos visualizar el valor de cualquier variable en ese instante poniendo el ratón encima de la variable en el propio código fuente.
* En el panel izquierdo, podemos acceder a las variables del programa (tanto locales como globales y superglobales) y definir cualquier expresión que queramos que se vaya evaluando en tiempo real (*watches*).

Además de estas funciones comunes de depuración, *xdebug* ofrece muchas funciones avanzadas que nosotros no vamos a ver, pero que puede que te interese usar en el futuro. Por ejemplo, puedes crear diferentes configuraciones de la sesión de *debugging* en el archivo *launch.json* para depurar diferentes aplicaciones (no es lo mismo depurar una aplicación web que una aplicación de línea de comandos). Otra función muy interesante de *xdebug* es la posibilidad de hacer *profiling*, es decir, un análisis del rendimiento de la aplicación web para detectar posibles problemas de rendimiento o cuellos de botella.

Si quieres ampliar la información sobre *xdebug*, [aquí tienes la documentación oficial](https://xdebug.org/docs/).