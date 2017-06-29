---
published: true
layout: post
date: 2017-06-20 20:00:00
summary: Si vas a crear tu propia base de datos es una buena idea adherirse a las reglas del diseño de bases de datos, porque de esta forma te asegurarás a la larga de contar con integridad y mantenibilidad de tus datos. Este tutorial te enseñará qué son las bases de datos y cómo diseñarlas de tal manera que se conformen con las reglas del diseño de bases de datos relacionales.
categories: tutoriales
title: Diseño de base de datos
---
Las bases de datos son programas que permiten almacenar y recuperar grandes cantidades de datos relacionados entre sí. Las bases de datos consisten en **tablas** que contienen **datos**. Cuando estés creando una base de datos debes pensar en qué **tablas** vas a crear y qué **relaciones** existen entre los datos de esas tablas. En otras palabras, tienes que pensar acerca del ***diseño*** de tu base de datos. Una base de datos **bien diseñada** asegurará la integridad y la mantenibilidad de tus datos.

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}
- [Lenguaje de consulta estrucurada SQL](#lenguaje-de-consulta-estrucurada-sql)
    - [El modelo relacional](#el-modelo-relacional)
    - [Acerca de los ejemplos](#acerca-de-los-ejemplos)
        - [Base de datos](#base-de-datos)
        - [Sistema de gestión de bases de datos](#sistema-de-gestión-de-bases-de-datos)
        - [Herramientas de modelado de bases de datos](#herramientas-de-modelado-de-bases-de-datos)
    - [El diseño es independiente](#el-diseño-es-independiente)
- [Un poco de historia](#un-poco-de-historia)
    - [Tablas en las bases de datos](#tablas-en-las-bases-de-datos)
    - [Historia del modelo relacional](#historia-del-modelo-relacional)
- [Características de las bases de datos relacionales](#características-de-las-bases-de-datos-relacionales)
    - [El uso de claves](#el-uso-de-claves)
    - [Evitar la redundancia de datos](#evitar-la-redundancia-de-datos)
    - [Limitando la entrada](#limitando-la-entrada)
    - [Manteniendo la integridad de los datos](#manteniendo-la-integridad-de-los-datos)
        - [Permisos](#permisos)
        - [Lenguaje de consulta estructurada SQL](#lenguaje-de-consulta-estructurada-sql)
        - [Portabilidad](#portabilidad)
- [Tablas y la clave primaria](#tablas-y-la-clave-primaria)
    - [Claves primarias en la vida diaria](#claves-primarias-en-la-vida-diaria)
    - [Características de la clave primaria](#características-de-la-clave-primaria)
        - [La clave primaria identifica a los datos](#la-clave-primaria-identifica-a-los-datos)
        - [La clave primaria es única](#la-clave-primaria-es-única)
        - [Tipos de datos de la clave primaria](#tipos-de-datos-de-la-clave-primaria)
        - [Claves compuestas](#claves-compuestas)
        - [Autonumeración](#autonumeración)
- [Enlazar tablas con claves foráneas](#enlazar-tablas-con-claves-foráneas)
    - [Uno-a-muchos](#uno-a-muchos)
    - [Decidir qué datos guardar](#decidir-qué-datos-guardar)
        - [Diseñar la tabla de clientes](#diseñar-la-tabla-de-clientes)
        - [Diseñar la tabla de pedidos](#diseñar-la-tabla-de-pedidos)
    - [Crear la relación de clave foránea](#crear-la-relación-de-clave-foránea)
- [Crear el diagrama de Entidad-Relación (ERD)](#crear-el-diagrama-de-entidad-relación-erd)
    - [Entidades](#entidades)
        - [No nos pongamos *tan* académicos](#no-nos-pongamos-tan-académicos)
    - [Relaciones](#relaciones)
        - [Relación de uno-a-muchos](#relación-de-uno-a-muchos)
            - [¿Cómo identificar una relación uno-a-muchos?](#¿cómo-identificar-una-relación-uno-a-muchos)
            - [Ejemplos](#ejemplos)
        - [Relación muchos-a-muchos](#relación-muchos-a-muchos)
            - [Modelar una relación muchos-a-muchos](#modelar-una-relación-muchos-a-muchos)
            - [Otro ejemplo: reservaciones en un hotel](#otro-ejemplo-reservaciones-en-un-hotel)
        - [Relación uno-a-uno](#relación-uno-a-uno)
            - [En la misma tabla](#en-la-misma-tabla)
            - [En tablas separadas](#en-tablas-separadas)
            - [Ejemplos de relación uno-a-uno](#ejemplos-de-relación-uno-a-uno)
- [Normalización de una base de datos](#normalización-de-una-base-de-datos)
    - [Primera forma normal (1NF)](#primera-forma-normal-1nf)
        - [Clave primaria](#clave-primaria)
        - [Atomicidad](#atomicidad)
        - [El orden de las filas en realidad no importa](#el-orden-de-las-filas-en-realidad-no-importa)
    - [Segunda forma normal (2NF)](#segunda-forma-normal-2nf)
        - [Redundancia de datos:](#redundancia-de-datos)
    - [Tercera forma normal (3NF)](#tercera-forma-normal-3nf)
        - [Dependencias transitivas](#dependencias-transitivas)
    - [Otro ejemplo: una tienda web](#otro-ejemplo-una-tienda-web)
        - [Resumen del sistema de la tienda](#resumen-del-sistema-de-la-tienda)
        - [Identificar entidades y relaciones](#identificar-entidades-y-relaciones)
            - [Tabla de pedidos](#tabla-de-pedidos)
                - [Cantidad de pedidos](#cantidad-de-pedidos)
                - [Tipo de pago](#tipo-de-pago)
                - [Monto total por pedido](#monto-total-por-pedido)
                - [Almacenar un historial de precios para productos](#almacenar-un-historial-de-precios-para-productos)
            - [Tabla de productos](#tabla-de-productos)
    - [Conclusiones](#conclusiones)
        - [¿Dónde ir ahora?](#¿dónde-ir-ahora)


> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://en.tekstenuitleg.net/articles/software/database-design-tutorial/intro.html). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

## Lenguaje de consulta estrucurada SQL
(Del inglés *Structured Query Language*)

Una base de datos se crea para almacenar y recuperar datos. Esto significa que queremos ser capaces de poder **INSERT** (*INSERTAR*, en español) datos en la base de datos y también **SELECT** (*SELECCIONAR*) datos de esa base de datos.

Para esas tareas se inventó entonces un lenguaje de consultas llamado el **Lenguaje de Consulta Estructurada**, o SQL. Tanto SELECT como INSERT son operaciones que forman parte de ese lenguaje. Por ejemplo, podemos ver en la siguiente imagen una consulta en SQL con SELECT y su resultado:

![enter image description here](https://i.imgur.com/6BcsLZq.png)

SQL ya en sí mismo en su tema bastante grande, por lo que está por fuera del alcance de éste artículo. Aquí nos enfocaremos estrictamente en el **diseño de las bases de datos**.  Discutiremos los fundamentos de SQL en un tutorial separado en el futuro.

## El modelo relacional
Aquí te mostraremos cómo crear un modelo de bases de datos **relacional**, el cual es un modelo que describe cómo organizar datos en tablas y cómo definir relaciones entre esas tablas, esto resulta en un diagrama de base de datos o Diagrama de Entidad-Relación como el que se puede ver en la siguiente imagen:

![Ejemplo tomado de MySQL Workbench](https://i.imgur.com/hcWJf1z.png)

## Acerca de los ejemplos
Para los ejemplos en este tutorial se usaron varias aplicaciones:

### Base de datos
Primero, el software usado para crear las tablas de los ejemplos es [MySQL](www.mysql.com) el cual es el sistema de gestión de bases de datos más popular y que además es *libre* (asegúrate de leer el siguiente parrafo acerca de por qué este no es un tutorial de MySQL).

### Sistema de gestión de bases de datos
Cuando instalas MySQL sólo tendrás una interfaz con una línea de comandos para operar. Personalmente preferimos una herramienta gráfica para administrar las bases de datos. A menudo usamos [SQLyog](https://github.com/webyog/sqlyog-community/wiki/Downloads) para administrar MySQL. SQLyog es una herramienta disponible de forma gratuita para administrar MySQL. Cuando veas una imagen de una **tabla** en este tutorial, viene de SQLyog. 

### Herramientas de modelado de bases de datos
MySQL también tiene una gran herramienta de diseño llamada [MySQL Workbench](http://www.mysql.com/products/workbench/), el cual también está disponible de forma gratuita. MySQL Workbench te permite diseñar una base de datos de forma gráfica y te da la posibilidad de generar la base de datos directamente desde el diseño *(Forward engineering)* . Cuando veas imágenes de diagramas de bases de datos en este tutorial (incluyendo el de arriba), vienen de MySQL Workbench.

## El diseño es independiente
Es importante saber que aunque los ejemplos en este tutorial vengan de MySQL, el **diseño de las bases de datos es independiente de la propia base de datos**. Esto significa que la información en este artículo aplica a las bases de datos (relacionales) en general, no sólo a MySQL. Puedes implementar un diseño en cualquier base de datos relacional que quieras, como MySQL, [PostgreSQL](http://www.postgresql.org/), [Microsoft Access](office.microsof.com/access), [Microsoft SQL](http://www.microsoft.com/sqlserver/) u [Oracle](http://www.oracle.com/technetwork/developer-tools/sql-developer/overview/index.html). Éste es un tutorial de diseño de bases de datos relacionales, que **no** un tutorial de MySQL.

Ahora veamos un breve repaso de la evolución de las bases de datos, así que conocerás de dónde vienen tanto las bases de datos como el modelo relacional.

# Un poco de historia
En los 70's y 80's, cuando los científicos computacionales todavía vestían chaquetas marrones de esmoquín y gafas con marcos grandes y cuadrados, los datos se solían almacenar en **archivos planos**, que son documentos de texto en los cuales los datos son separados (normalmente) por comas o tabulaciones.

![Como los IT pro lucían en los setenta](https://i.imgur.com/z0PRObB.jpg) 

> Sí, ese en la esquina inferior izquierda es Bill Gates

Los archivos planos todavía se usan para representación de listas simples de datos. El formato de Valores Separados por Comas (CSV, *Comma Separated Values*) es muy popular y ampliamente soportado por diferentes programas y sistemas operativos, por ejemplo el Microsoft Excel. Los datos contenidos en un archivo plano pueden ser leídos por un progama. Un ejemplo de cómo luce un archivo plano puede ser el siguiente:

![enter image description here](https://i.imgur.com/ClhjPZU.png)

Un programa que esté leyendo este archivo plano tendría que decírsele que los datos están separados por comas. Si quisiera seleccionar la **categoría** en la que está **Tutorial de diseño de bases de datos**, tendría que leer el archivo línea por línea hasta encontrar las palabras "Tutorial de diseño de bases de datos" y luego tendría que leer la palabra después de la coma para encontrar la categoría **Software**.

## Tablas en las bases de datos

Leer un archivo plano línea por línea no es muy eficiente. En una **base de datos relacional** los datos se almacenan en **tablas**. La siguiente tabla contiene los mismos datos que el archivo plano. Cada **fila** o **"registro"** contiene un tutorial. Cada **columna** representa una propiedad del tutorial, en este caso el título y la categoría.

![enter image description here](https://i.imgur.com/JSDer4S.png)

Un programa puede buscar en la columna **tutorial_id** de esta tabla para un id específico y encontrar rápidamente el correspondiente título y categoría. Esto es mucho más rápido que ir por todos los datos línea por línea.

Las bases de datos relacionales modernas son diseñadas para permitir selecciones avanzadas de filas, columnas y desde múltiples tablas al mismo tiempo a velocidades muy altas.

## Historia del modelo relacional
El modelo de bases de datos relacional fue inventado en los sententas por [Tedd Codd](https://es.wikipedia.org/wiki/Edgar_Frank_Codd), un científico computacional británico. Él quería que su modelo superara las deficiencias del modelo de bases de datos en redes y el [modelo de bases de datos jerárquico](https://es.wikipedia.org/wiki/Base_de_datos_jer%C3%A1rquica). Y lo hizo muy bien. El modelo relacional está ahora ampliamente adoptado y considerado como muy poderoso para la organización eficiente de datos.

A día de hoy, un amplio rango de productos se encuentra disponible, yendo desde aplicaciones ligeras hasta sistemas de servidos completos con métodos altamente optimizados. Algunos de los sistemas de gestión de bases de datos (RDBMS, *Relational Dababase Management Systems*) más conocidos son:

 - **Oracle**: Usado principalmente para apliaciones grandes y profesionales
 - **Microsoft SQL Server**: RDBMS de Microsoft, disponible tanto para Linux como para Windows
 - **MySQL**: Es el sistema de gestión de código abierto más popular. Es ampliamente usado en la comunidad Open Source tanto por principiates como por profesionales
 - **IBM**: Tiene una amplia gama de sistemas de bases de datos, del cual DB2 es el más conocido
 - **Microsoft Access**: En realidad es más que un RDBMS, es una herramienta que permite construir una completa aplicación de bases de datos con interfaz de usuario.

Ahora veamos algunas de las características de las bases de datos relacionales:

# Características de las bases de datos relacionales
Las bases de datos se diseñan para almacenar y recuperar rápidamente grandes cantidades de datos.

## El uso de claves
Cada fila de datos en una tabla está identificada por una única "clave" llamada la *clave primaria* (PK, Primary Key). La clave primaria es a menudo un número que se incrementa automáticamente como 1, 2, 3, 4, ...,. Los datos en tablas diferentes pueden enlazarse usando claves. Los valores de la clave primaria de una tabla pueden agregarse a las filas de una tabla diferente, y por lo tanto enlazar a esas filas.

Usando el Lenguaje de consulta estructurada SQL, los datos de diferentes tablas que estén enlazados por claves pueden seleccionarse de una sola vez. Por ejemplo, haces una consulta que selecciona todas las órdenes desde una **tabla de órdenes** que pertenezcan al usuario con id 3 (mike) desde la **tabla de usuarios**.  Hablaremos más en detalle de las claves más adelante en éste artículo.

![enter image description here](https://i.imgur.com/1LiNPo9.jpg)
> La columna id de esta tabla es la "clave primaria". Cada registro tiene una única llave primaria, usualmente un número. La columna grupo es una columna de "claves foráneas". A juzgar por su nombre, probablemente referencia una tabla que contiene grupos de usuarios.

## Evitar la redundancia de datos
En un diseño de bases de datos que se adhiera a las reglas del modelo relacional, cada elemento de datos, un nombre de usuario por ejemplo, se almacena una sóla vez, esto es, en un único lugar. Así se evita tener que mantener los mismos datos en varios lugares. La duplicación de datos se llama **redundancia de datos** y debe ser evitada en un buen diseño.

## Limitando la entrada
Usando una base de datos relacional puedes especificar qué tipo de datos está permitido en una columna de la base de datos. Puedes crear campos que contengan números, números decimales, textos cortos, textos largos, fechas, entre otros.

![enter image description here](https://i.imgur.com/EX1p6nF.png)
> Cuando se define la tabla en una base de datos se debe especificar un tipo para cada columna. 'Varchar' es el tipo de datos en MySQL para un texto corto de máximo 255 caracteres e 'int' es un número entero.

Aparte de los tipos de datos, los sistemas de bases de datos permiten aplicar otras restricciones como la **longitud** o como hacer cumplir la **unicidad** de un cierto campo. La restricción de unicidad a menuda se usa para campos que contienen nombres de usuarios y direcciones de correo electrónico.

Esas restricciones te dan el control sobre la integridad de tus datos. Previenen situaciones como:

- Ingresar una dirección (texto) en un campo donde se esperaría un número
- Ingresar un código postal de 100 digitos
- Tener dos usuarios con el mismo nombre de usuario
- Tener dos usuarios con la misma dirección de correo electrónico
- Ingresar un peso (número) en un campo de cumpleaños (fecha)

## Manteniendo la integridad de los datos
 Al configurar las propiedades de los campos, enlazar tablas y configurar restricciones puedes incrementar la confiabilidad de tus datos.

### Permisos
La mayoría de los sistemas de bases de datos ofrecen una estructura de permisos que pueden ser asignados a varios usuarios. Algunas de las operaciones que se le pueden habilitar o deshabilitar a un usuario son: SELECT, INSERT, DELETE, ALTER, CREATE, etc. Esos permisos corresponden a las operaciones que pueden llevarse a cabo usando SQL.

### Lenguaje de consulta estructurada SQL
Para poder ejecutar operaciones en una base de datos, como guardar nuevos datos, seleccionar y alternar datos existentes, se usan las consultas SQL. El lenguaje de consulta estructurada es relativamente fácil de entender y permite hacer operaciones avanzadas, como seleccionar datos enlazados en múltiples tablas usando JOIN. Como se discutió anteriormente, SQL se encuentra fuera del alcance de éste artículo. Discutiremos SQL en un artículo separado. Nos enfocaremos estrictamente en el **diseño**.

La manera en la que diseñes tu base de datos tiene un efecto directo en las consultas que necesites escribir para recuperar datos. Esta es otra razón importante por la cual es buena idea pensar en cómo diseñas. Con una base de datos bien diseñada puedes escribir consultas SQL limpias y fáciles.

![enter image description here](https://i.imgur.com/6BcsLZq.png)
> Una consulta SQL que selecciona (SELECT) todo el contenido de la tabla USER

### Portabilidad
El modelo relacional es un estándar. Al adherirnos a las reglas del modelo relacional nos aseguramos que los datos puedan transferirse entre sistemas de bases de datos relacionales con relativa facilidad.

Como se dijo anteriormente, el **diseño de la base de datos** se trata de identificar las relaciones existentes entres los datos y ponerlas en el papel (o en un modelo computacional). El diseño es independiente del sistema de bases de datos que vayas a usar para implementar la base de datos.

# Tablas y la clave primaria
Como aprendimos anteriormente, los datos en una base de datos se guardan en **tablas** que contienen **filas** o **registros**.  Retomemos el ejemplo anterior de la tabla que cotiene información acerca de tutoriales:

![enter image description here](https://i.imgur.com/JSDer4S.png)
> Una tabla con tutoriales

Esta tabla consiste en 6 tutoriales. Todos son diferenstes, pero cada tutorial tiene los mismos **campos**, llamados *tutorial _id*, *titulo* y *categoria*, donde *tutorial_id* es la **clave primaria** de la tabla. La **clave primaria** es un valor único para cada registro de la tabla.

En la siguiente tabla de clientes, el *doctor_id* es es la clave primaria. También es un número único para cada registro.

![enter image description here](https://i.imgur.com/YqnFaAh.png)

## Claves primarias en la vida diaria
Las claves primarias se usan para identificar registros en una base de datos. Están en nuestro alrededor. Cada vez que encuentras un número único pude que sea una clave primaria en una base de datos (pero no necesariamente tiene que serlo):

- El número de pedido que recibes cuando haces una compra en internet puede ser una clave primaria de alguna tabla de pedidos en la base de datos de la tienda, porque su valor es único.
- Tu número de identificación puede ser una clave primaria en alguna base de datos gubernamental.
- Un número de factura puede ser una clave primaria en una tabla en una base datos que guarde facturas que hayan sido enviadas a clientes.
- Entre otros

## Características de la clave primaria
Veamos las caracerísticas de la clave primaria:

### La clave primaria identifica a los datos
La clave primaria se usa para **identificar** registros en una tabla. Cuando llamas a un centro de atención al cliente y la persona que te atiende te pregunta por tu identificación en realidad está preguntando "¿por qué código único puedo buscar en nuestra base de datos para identificarte en la tabla de clientes?"

### La clave primaria es única
La clave primaria es siempre un valor único, porque si no fuera único, no se podría usar para identificar sin ambigüedades registros en una tabla. Esto significa que un valor sólo puede aparecer una vez en una columna de claves primarias. Los sistemas de bases de datos relacionales están generalmente programados para deshabilitar la inserción de un valor duplicado en una clave primaria. Tratar de hacer eso resultará en un error.

### Tipos de datos de la clave primaria
La clave primaria de una tabla normalmente es un número, pero puede ser cualquier tipo. No es extraño utilizar una cadena de caracteres como una clave primaria.

### Claves compuestas
Normalmente la clave primaria es un único campo, pero también puede ser una combinación de dos campos que identifiquen un registro. La combinación debe ser única en la tabla. Hablaremos de esto más adelante.

### Autonumeración
Las claves primarias normalmente, pero no siempre, son administradas por la base de datos. Puedes decirle a la base de datos que automáticamente asigne una clave primaria única numérica a cada registro en la tabla. A menudo la numeración empieza desde 0. Tal clave primaria es llamada una clave **autonumerada**. Las claves autonumeradas son una buena forma de asegurarse que los registros tengan una clave primaria única.  Otro nombre para tal campo de claves primarias es la *[clave sustituta](https://es.wikipedia.org/wiki/Clave_sustituta)*, como la clave no es parte real de los datos que se almacenan en la tabla (como un usuario), se llama "sustituta".

# Enlazar tablas con claves foráneas
Cuando empezamos a diseñar bases de datos tenemos la tendencia de intentar guardar datos que parecen "relacionados" en una tabla. Podemos, por ejemplo, intentrar guardar **pedidos** en un campo en la tabla de **clientes**. Porque los pedidos "pertenecen" a un cliente, ¿cierto? **No**. Los clientes y los pedidos representan **entidades** separadas en una base de datos. Ambos necesitan tener su propia tabla. Y los registros en esas dos tablas pueden enlazarse para establecer la relación entre clientes y pedidos. Diseñar una base de datos es cuestión de decidir qué entidades son las que quieres guardar y qué relaciones existen entre ellas.

## Uno-a-muchos
Clientes y pedidos están relacionados el uno al otro con una relación **uno-muchos**, porque, **un** cliente puede tener **varios** pedidos y cada pedido (**muchos**) pertenece exactamente a **un** cliente. No te preocupes si esto es muy vago por el momento. Hablaremos más acerca de las relaciones más adelante.

Lo que es imporante ahora es que una relación uno-a-muchos requiere el uso de **dos** tablas separadas. Una para los clientes y otra para los pedidos. Practiquemos un poco creando esas tablas.

## Decidir qué datos guardar
Primero, debemos decidir qué información guardar acerca de los **clientes** y de los **pedidos**. Para poder hacer esto, debemos preguntarnos entonces:

"***¿Qué únicas piezas de información pertenecen a un cliente?*** y ***¿Qué únicas piezas de información pertenecen a un pedido?***

### Diseñar la tabla de clientes
Los **pedidos** pertenecen a un cliente, pero no son una **única** pieza de información. Los siguientes campos son piezas únicas de información que pertenecen a un cliente:

- cliente_id (clave primara)
- nombres
- apellidos
- direccion
- codigo_postal
- pais
- fecha_nacimiento
- usuario
- contraseña

Creemos la tabla entonces en *SQLyog*.  La siguiente imagen es un ejemplo que cómo luce una tabla cuando es creada usando "new table" en la ventana de SQLyog. Todos los sistemas gráficos de administración de bases de datos tienen interfaces similares para crear tablas. También puedes crear la tabla usando SQL en la línea de comandos, sin interfaz gráfica:

![enter image description here](https://i.imgur.com/spqOiOE.png)
> Creando una tabla en SQLyog. Nótese que la caja de PK (Clave primaria, Primary Key) está marcada para el campo cliente_id, por lo tanto cliente_id es la clave primaria. También, la caja Auto Incr está marcada, por lo que la base de datos automáticamente proveerá un valor único (incremental) para este campo.

### Diseñar la tabla de pedidos
***¿Qué únicas piezas de información pertenecen a un pedido?*** Veamos:

- El pedido_id
- La fecha en que se realizó el pedido
- El cliente que realizó el pedido

La siguiente imagen es el ejemplo de cómo luciría la tabla creada en SQLyog:

![enter image description here](https://i.imgur.com/nQTfeJV.png)
> Diseño de la tabla de pedidos. El campo cliente es una referencia (una 'clave foránea') a cliente_id en la tabla de clientes.

Esas dos tablas están enlazadas, porque el campo **cliente** en la tabla de **pedidos** es una referencia a la clave primaria (**cliente_id**) de la tabla de clientes. Tal referencia es llamada una relación de **clave foránea**. Deberías ver la clave foránea como la copia de la llave primaria de otra tabla. En este caso, el **cliente_id** desde la tabla de **clientes** se copia en la tabla de pedidos  de tal forma que cada pedido queda enlazado a un cliente.

## Crear la relación de clave foránea
Te estarás preguntando cómo puedes **ver** que el campo *cliente* en la tabla de pedidos referencia al campo *cliente_id* en la tabla de clientes. No puedes, porque aún no te hemos mostrado como creamos la relación.

![enter image description here](https://i.imgur.com/KIA5l72.png)
> Estableciendo relaciones de claves foráneas entre las tablas de pedidos y clientes

En la imagen anterior se puede ver que la columna cliente de la tabla de pedidos queda enlazada a la llave primaria *cliente_id* de la tabla de clientes.

Ahora, cuando veamos los datos que pudieran estar presentes en las tablas de pedidos y clientes, verás cómo están enlazados los clientes y los pedidos:

![enter image description here](https://i.imgur.com/M4ECq92.png)
> Los pedidos están enlazados con los clientes por medio del campo cliente que referencia a la tabla de clientes.

Por los datos arriba presentados, podemos ver que el cliente llamado *Joe* hizo dos pedidos, el cliente *Satya* hizo uno y el cliente *Terry* también hizo uno.

Tal vez te estés preguntando **qué** pidieron exactamente. Esa es una buena pregunta. Tal vez estuvieras esperando los productos pedidos en la tabla de pedidos. Pero eso sería un mal diseño. ¿Cómo pondrías múltiples productos en un solo registro de pedido? Los **productos** son entidades separadas que deben almacenarse/guardarse en una tabla separada. La relación entre **pedidos** y **productos** es una relación de *muchos-a-muchos*. Lo veremos más adelante.

# Crear el diagrama de Entidad-Relación (ERD)

Acamos de aprender cómo los registros de diferentes tablas se relacionan los unos a los otros en una base de datos relacional. Antes de empezar a crear y relacionar tablas es importante pensar acerca de las **entidades** que existen en el sistema y sus relaciones. En el diseño de una base de datos, las entidades y sus relaciones son representadas en un **diagrama de entidad-relación** (ERD, *Entity-Relationship Diagram*) que  es el resultado del proceso de diseñar la base de datos.

## Entidades
Te estarás preguntando qué es una entidad. Bueno, es una "cosa". En un sistema. Allí. Aquí. Mi mamá siempre quiso que fuera profesor, porque explico *muy bien*.

En el contexto del **diseño** de una base de datos una entidad es cualquier cosa que **tal vez** merece su propia tabla en el modelo. Cuando diseñas una base de datos debes identificar las **entidades** en el sistema que estés creando. Esto ya es más algo de hablarcon tu cliente o contigo mismo y resolver con qué datos de tus sistema vas a trabajar.

Tomemos una tienda web por poner un ejemplo. Una tienda web vende **productos**. Un producto puede ser una muy obvia entidad en un sistema de la tienda. Los productos son **pedidos** por **clientes**. Dos entidades más que obvias que además ya habíamos visto: pedidos y clientes.

Un pedido es pagado por un clinete... qué interesante. ¿Tendremos una tabla de **pagos** en la base de datos de nuestra tienda? Posiblemente. ¿Pero acaso no es el pago una "única pieza de información" que pertenece a un pedido? Eso también es posible.

Si no está seguro sólo piensa acerca de qué información quieras almacenar del pago. Tal vez quieras guardar el **método de pago** y la **fecha del pago**. Esas son todavía piezas de información que pertenecen a un pedido. Puedes interpretrar esto como el método de pago de un *pedido* y la fecha de pago de un *pedido*, así que no vemos la necesidad de modelar el **pago** en una tabla separada, aunque, conceptualmente, puedas ver al pago como una "entidad", porque puedes tratarlo como un contenedor separado de informarción (fecha de pago, método de pago).

### No nos pongamos *tan* académicos
Como puedes ver hay una diferencia entre una entidad y una una tabla real en una base de datos. Los especialistas en IT pueden volverse MUY académicos acerca de esta diferencia. No somos como tales especialistas en IT. Esta diferencia depende de la forma en que veas los datos. Si ves al modelado de datos desde una perspectia de software puedes terminar con un montón de entidades que no se traducen directamente en tablas. En este tutorial estamos mirando a los datos estrictamente desde una perspectiva de la base de datos y en nuestro pequeño mundo una entidad se traduce en una tabla.

![enter image description here](https://i.imgur.com/pnFPdLd.jpg)
> Y casi lo logras, está muy cerca de tener tu diploma en diseño de base de datos.

Como puedes ver, decidir qué entidades tiene tu sistema es un poco de un proceso intelectual que requiere cierta experiencia y que es a menudo una materia que requiere cambiar y repensar y reconsiderar.

![enter image description here](https://i.imgur.com/s3nTpF4.png)
> Un diagrama entidad-relación puede volverse muy grande si estás construyendo una aplicación compleja. Algunos contienen cientos o incluso miles de tablas.

## Relaciones
El segundo paso de un diseño de bases de datos es decidir qué relaciones existen entre las entidades de tu sistema. Ahora esto puede ponerse un poco complicado. Con algo de experiencia y un poco de (re)consideración puedes terminar con un modelo de base de datos correcto o *al menos* correcto.

### Relación de uno-a-muchos
Ya vimos como los datos de diferentes tablas pueden enlazarse definiendo una relación de clave foránea. También vimos cómo los pedidos se enlazaban a los clientes al incluir el *cliente_id* como una clave foránea en la tabla de pedidos.

Otro ejemplo de una relación uno-a-muchos es la relación que existe entre madres e hijos. Una madre puede tener **varios** hijos pero cada hijo tiene solo **una** madre. (Técnicamente sería mejor hablar de una mujer y de sus hijos, porque en una relación de uno-a-muchos una madre puede tener 0, 1 o varios hijos y una madre con 0 hijos no es técnicamente una madre)

Cuando un registro en la tabla A se enlaza con 0, 1 o varios registros de la tabla B, estás trabajando con una **relación uno-a-muchos**. En el modelo relacional una relación uno-a-muchos es modelado usando dos tablas.	

![enter image description here](https://i.imgur.com/dfh0pHW.png)
> Esquema que representa una relación de uno-a-muchos. Un registro en la tabla A tiene 0, 1 o varios registros asociados en la tabla B.

#### ¿Cómo identificar una relación uno-a-muchos?
Cuando tengas dos entidades pregúntate estas preguntas:
1. ¿Cuántas entidades de B pueden pertenecer a la entidad A?
2. ¿Cuántas entidades de A pueden pertenecer a la entidad B?

Si la respuesta a la pregunta 1 es **muchas** y la respuesta a la pregunta 2 es **una** estás tratando con una relación uno-a-muchos.

#### Ejemplos
Algunos ejemplos de una relación uno-a-muchos:

- Un carro y sus partes. Cada parte pertenece a un carro y carro tiene múltiples partes
- Un cine y sus salas. Un cine normalmente tiene varias salas y cada sala pertenece a un único cine
- Un ERD y sus tablas. Un diagrama entidad-relación tienen una o más tablas y cada una de esas tablas pertenece a un único diagrama
- Las casas en una calle. Una calle tienen múltiples casas y una casa pertenece a una única calle

### Relación muchos-a-muchos
La relación muchos-a-muchos es donde múltiples files de la tabla A pueden corresponder a múltiples filas de la tabla B. Un ejemplo de esta relación es una escuela donde los profesores enseñan a estudiantes. En la mayoría de escuelas cada profesor tiene múltiples estudiantes y cada estudiante tiene múltiples profesores.

La relación entre distribuidores de cerveza y las cervezas que distribuyen es una relación muchos-a-muchos también. Nótese que en el diseño de bases de datos te debes preguntar no qué relación existe *en el momento* sino qué relación puede existir *en el futuro*. Si en el presente todos los distribuidores distribuyen múltiples tipos de cerveza, pero cada cervez es distribuída por sólo un distribuidor, estás al frente de una relación uno-a-muchos. (Un distribuidor tiene múltiples cervezas, pero cada cerveza tiene sólo un distribuidor). No te tientes a modelar esta situación en una relación uno-a-muchos. Hay una buena probabilidad de que en el futuro dos o más distribuidores también distribuyan el mismo tipo de cerveza y cuando eso pase tu base de datos no estará preparada .

#### Modelar una relación muchos-a-muchos
Una relación muchos-a-muchos se modela con tres tablas. Dos tablas "fuentes" y una tabla de **unión**. La clave primaria de una tabla de unión *A_B* es una clave **compuesta**, hecha de dos campos: las dos claves foráneas que referencian a la clave primaria de la tabla A y la tabla B.

![enter image description here](https://i.imgur.com/8Bl8zE6.png)

Todas las claves primarias deben ser únicas. Esto implica que la combinación del **campo A y el B** debe ser único en la tabla A_B. El siguiente ejemplo muestra cómo las tablas que pueden existir en una relación muchos-a-muchos entre marcas de cerveza de Bélgica y sus distribuidores en Holanda. Nótese cómo las combinaciones de *cerveza_id* y *distribuidor_id* son únicas en la tabla de unión.

![enter image description here](https://i.imgur.com/Vf9dQYp.png)

En las tablas de arriba se enlazan las cervezas y sus distribuidores con una relación muchos-a-muchos usando la tabla de unión *cerveza_distribuidor*. Nótese que "Gentse Tripel" (157) es distribuída por Horeca Import NL (157, AC001), Jansen Horeca (157, AB899) y Peterson Drankenhandel (157, AC009). Viceversa, Peterson Drankenhandel es el distribuidor de tres cervezas: Gentse Tripel (157, AC009), Uilenspiegel (158, AC009) y Jupiler (163, AC009).

Nótese que en las tablas de arriba los campos de la clave primaria están en azul y en negrita. En los modelos de bases de datos, las claves primarias usualmente se subrayan. Nótese también (otra vez) que la tabla de asociación *cerveza_distribuidor* tiene una clave primaria compuesta por dos claves foráneas. Las tablas de unión tienen siempre una clave primaria compuesta.

Hay otra cosa imporante por notar acerca de la relación muchos-a-muchos: consiste de **dos relaciones uno-a-muchos**. Tanto la tabla *cervezas* como la tabla de *distribuidores* tienen una relación uno-a-muchos con la tabla de unión.

#### Otro ejemplo: reservaciones en un hotel
Como ejemplo final dejános mostrarte cómo podría modelarse las reservas de habitaciones en un hotel por los huéspedes.

![enter image description here](https://i.imgur.com/V7NW8Jx.png)

En este ejemplo vemos que hay una relación muchos-a-muchos entre huéspuedes y habitaciones. Una habitación puede ser reservada por varios huéspuedes al pasar el tiempo y al pasar el tiempo un huésped puede reservar varias habitaciones en el hotel. La tabla de unión en este ejemplo no es una tabla de unión clásica que consiste de sólo dos claves foráneas, sino que es una entidad separada que tiene relación con otras dos entidades.

Encontrarás situaciones en las que la tabla de **unión** de dos entidades sea una nueva entidad.

### Relación uno-a-uno
En una relación uno-a-uno cada elemento de la entidad A puede asociarse con 0 o con 1 elemento de la entidad B. Un empleado, por ejemplo, usualmente está asociado a una oficina. O una marca de cerveza tiene un país de origen.

#### En la misma tabla
La relación uno-a-uno es fácilmente modelada: en una tabla. Las filas de la tabla contienen datos que están relacionados en una relación uno-a-uno a la clave primaria o a la fila. Los datos de cada cliente, por ejemplo, están enlazados en una relación uno-a-uno a su clave primaria, el *cliente_id*.

#### En tablas separadas
En casos raros, una relación uno-a-uno se modela usando dos tablas separadas. Tal configuración a veces se usa para sobrepasar limitaciones del sistema de la base de datos o para ganar rendimiento. O en casos raros tal vez decidas que quieras tener dos entidades en diferentes tablas, manteniendólas relacionadas en una relación uno-a-uno. Normalmente tener dos tabalas separadas en una relación uno-a-uno es considerada una mala práctica.

#### Ejemplos de relación uno-a-uno
- Las personas y sus pasaportes. Sin embargo, esto solo cuenta si miramos a sus pasaportes actuales. Cada persona tienen un único, actual pasaporte y cada pasaporte pertenece a una única persona.

![enter image description here](https://i.imgur.com/gD5kZyD.png)

Un diseño de base de datos relacional es una colección de tablas que están interrelacionadas por claves primarias y claves foráneas. El modelo relacional comprende un número de reglas que te ayudan a descubrir la relaciones correctas entre los datos. Esas reglas son llamadas las *formas normales*. Ahora veamos como normalizar una base de datos.

# Normalización de una base de datos
Las guías para un apropiado diseño de bases de datos relacionales residen en el **modelo relacional**. Están agrupadas en 5 grupos llamados las **formas normales**. La primera forma normal representa la forma más baja de normalización de la base de datos, la quinta representa la más alta forma de normalización de la base de datos.

Las forma normales son *guías* para un buen diseño de base de datos. No estás obligado a adherirte a todas las cinco formas normales cuando estés diseñando una base de datos. Sin embargo, se te sugiere normalizar la base de datos sobre la que vayas a trabajar, porque la normalización tiene ventajas significativas en términos de eficiencia y de mantenibilidad de tu base de datos.

- En una estructura de base de datos normalizada puedes hacer selecciones complejas de datos con relativa simplicidad con consultas SQL
- **Integridad de los datos**. Una base de datos normalizada permite confiabilidad en los datos almacenados
- La normalización evita redundancia. Los datos se almacenan siempre en una única ubicación que hace que sea fácil insertar, actualizar o eliminarlos. Hay una excepción a esta regla. Las claves por sí mismas se almacenan en múltiples lugares, porque son copiadas como claves foráneas en otras tablas. Si quieres decirlo correctamente dirías que los **datos lógicos** no se deben duplicar.
- La **escalabilidad** es la habilidad de un sistema para manejar con un crecimiento en el futuro. Para una base de datos esto significa que debe poder permitir realizar consultas rápidamente incluso cuando el número de datos crezca. La escalabilidad es una característica importante de cualquier modelo de bases de datos y de los sistemas de administración.

Estas son algunas de las tareas más generales que están asociadas a la normalización de una base de datos:

- Ordenar los datos en grupos lógicos o *conjuntos*
- Encontrar relaciones entre conjuntos de datos. Ya haz visto ejemplos de las relaciones uno-a-muchos y muchos-a-muchos en este tutorial
- Minimizar la redundancia de los datos. En otras palabras, asegurarse que los datos lógicos se almacenan en un único lugar.

Muy pocas bases de datos se adhieren a todas las cinco formas normales presentadas en el modelo relacional. Usualmente las bases de datos se normalizan hasta la segunda o tercerca forma normal. La cuarta y la quinta forma se usan raramente. Por lo tanto discutiremos solamente la primera, segunda y tercerca forma normal.

## Primera forma normal (1NF)
La forma normal establece que una tabla en una base de datos es la representación de una *entidad* en el sistema que estás construyendo. Ejemplos de entidades son pedido, cliente, reserva, hotel, producto, etc. Cada fila en la tabla representa una instancia de una entidad. Por ejemplo en una tabla de clientes cada fila representa a un cliente.

### Clave primaria
**Regla:** cada tabla tiene una clave primaria, consustente de la más pequeña cantidad de campos posibles.

Como ya sabes, una clave primaria puede cosnsitir de múltiples campos. Puedes por ejemplo escoger la combinación de fecha de cumpleaños, nombres y apellidos como la clave primaria (y esperar que esta combinación sea siempre única). Podría ser una mucho mejor opción escoger el número de identificación de alguien como la clave primaria, porque éste es el único campo que identifica ínequivocamente a una persona.

Mejor aún, cuando no hay un obvio candidato a la clave primaria, asigna una calve sustituta a tu tabla al crear un campo de id autoincremental.

### Atomicidad
**Regla:** los campos no se duplican en una fila y cada campo contiene un único valor.

Tomemos por ejemplo un sitio web de coleccionistas de carros en los que cada collecionista de carro que se registra en el sitio lo hace con sus carros. La tabla siguiente muestra los registros de carros de los usurarios que se registraron en el sitio web:

![enter image description here](https://i.imgur.com/vgwqLTl.png)
> Duplicación horizontal de los campos es un mal diseño

La duplicación horizontal de los campos es un mal diseño. Con este diseño sólo puedes almacenar hasta cinco carros y si tienes menos de cinco carros estás desperdiciando espacio en la base de datos con columnas vacías.

Otra mala solución es colocar múltiples valores en una celda:

![enter image description here](https://i.imgur.com/hcfuInc.png)

La solución correcta aquí es dividir los carros en una tabla separada e incluir una clave foránea que referencie a la tabla de usuarios.

### El orden de las filas en realidad no importa
**Regla:** el orden de las filas en una tabla no debe importar.

Tal vez estés inclinado a usar el orden de las filas en una tabla de clientes para determinar qué cliente se registró primero. Para este propósito deberías crear campos de fecha y hora que guarden la fecha y la hora de la subcripción del cliente. El orden de las filas inevitablemente cambiará cuando se eliminen, modifiquen e inserten clientes. Es por esto que no deberías confiar en el orden de las filas en una tabla. 

## Segunda forma normal (2NF)
Para que una base de datos esté normalizada de acuerdo a la segunda forma normal primero debe ser normalizada de acuerdo a las reglas de la primera forma normal. La segunda forma normal trata con la redundancia de datos.

### Redundancia de datos:
**Regla:** Los campos que no sean parte de la clave primaria deben depender de la clave primaria.

Eso puede sonar un poco académico. Lo que significa que es sólo debes almacenar datos en una tabla que estén directamente relacionados y que no pertenezcan a otra entidad. Adherirse a la segunda forma normal es cuestión de encontrar datos que frecuentemente estén duplicados a través de filas y que puedan pertenecer a una entidad diferente.

![enter image description here](https://i.imgur.com/xAJkWMT.png)
> Duplicación de datos a través de filas en el campo de tienda

La tabla de arriba podría pertenecer a una compañía que vende carros y tiene múltiples tiendas en Holanda. Si miras detalladamente la tabla, puedes encontrar múltiples ejemplos de duplicación a través de filas. El campo **marca** puede dividirse en una tabla separada. También, el campo **tipo** puede dividirse un otra tabla que tenga relaciones uno-a-muchos con la tabla *marca*, porque una marca tiene múltiples tipos.

La columna **tienda** contiene la tienda donde se encuentra actualmente el carro. *Tienda* es un caso obvio de redundancia de datos y una buena candidata a ser una entidad separada que puede ser referenciada desde la tabla de carros con una relación de clave foránea.

Abajo vemos un ejemplo de cómo se podría modelar mejor la situación de los carros evitando redundancia de datos en la tabla de carros:

![enter image description here](https://i.imgur.com/rGVhPr4.png)

En la configuración anterior, la tabla **carro** tiene una clave foránea referenciada a las tablas *tipo* y *tienda*. Ya no está la columna *marca*, porque está ímplicitamente referenciada a través de la referencia *tipo*. Cuando se referencia un tipo, también se referencia la marca, porque un tipo pertenece a una marca.

Se ha removido un montón de redundancia de datos del modelo. Si eres estricto puede que aún no estés satisfecho con esta solución. ¿Qué hay del campo **pais_de_origen** en la tabla *marca*? No hay duplicación **aún**, porque sólo hay cuatro marcas de diferentes países. Un diseñador de bases de datos estricto podría dividir los nombres de los países en una tabla separada de **países**.

E incluso puede que aún no estés satisfecho, porque podrías dividir el campo **color** en otra nueva tabla.

Qué tan estrictamente diseñes las tablas depende de tí mismo y de la situación. Si vas a tener grandes cantidades de carros en tu sistema y quieres poder ser capaz de buscar carros por *color*, podría ser de sabios separar los colores en un otra tabla, así no se tendrían duplicados.

Hay otra situación donde tal vez quieras separar los colores en otra tabla. Si quieres permitir que empleados de la compañía ingresen nuevos carros y quieres que puedan **escoger** un color de carro desde una lista predefinida. En ese caso deberías almacenar todos los colores posibles en tu base de datos, incluso si no hay carros con cierto color *aún*, quieres que esté presente en la base de datos, para que el empleado pueda seleccionarlo. 

## Tercera forma normal (3NF)
La tercera forma normal trata con las **dependencias transitivas**. Una dependencia transitiva entre campos de la base de datos existe cuando el valor de un campo que no sea una clave (primaria o foránea) es determinado por el valor de otro campo que no es tampoco una clave. Para que una base de datos esté en la tercera forma normal primero debe estar en la segunda forma normal.

### Dependencias transitivas
**Regla:** no deben haber dependencias transitivas entre campos en una tabla.

La tabla de clientes siguiente (mis clientes son jugadores de fútbol franceses y holandeses) contiene relaciones transitivas:

![enter image description here](https://i.imgur.com/Pz6jdPb.png)

En esta tabla no todos los campos dependen única y exclusivamente de la clave primaria. Existe una relación separada entre *codigo_postal*  y la *ciudad* y la *provincia*. En Holanda, la ciudad y la provincia están determinadas por el código postal, así que no hay necesidad de almacenar la ciudad y la provincia en la tabla de clientes. Si conoces el código postal, conoces la ciudad y la provincia.

Tales relaciones transitivas deben ser evitadas si quieres modelar tu base de datos en la tercera forma normal.

En este caso, eliminar las relaciones transitivas de la tabla puede lograrse por remover los campos de ciudad y de provincia de la tabla y guardarlos en una tabla separada, que contenga el código postal (clave primaria), el nombre de la provincia y la ciudad. Averigüar las combinaciones de código postal - ciudad - provincia para un país entero es un trabajo difícil. Por eso es que tales tablas se venden comercialmente.

Otro ejemplo de aplicación de la tercera forma normal es esta (muy) sencilla tabla de pedidos de una tienda online:

![enter image description here](https://i.imgur.com/lhTmJ6z.png)

El *Impuesto de Valor Agregado* es un porcentaje que se le agrega al precio de un producto (19% en la tabla anterior). Esto significa que la cantidad *total_sin_iva* puede calcularse de la cantidad *total_con_iva* y viceversa. Debes almacenar sólo uno de esos dos campos, pero no ambos. Debes dejar la tarea de calcular el *total_con_iva* desde el *total_sin_iva* o viceversa al programa que usa la base de datos.

La tercera forma normal básicamente dice que no debes almacenar datos en campos que puedan derivarse de otros campos (que no sean claves) en una tabla. Especialmente en el ejemplo de la tabla de clientes, aplicar la tercera forma normal requiere un montón de trabajo o adquirir una tabla comercial de códigos postales - provincias - ciudades.

La tercera forma normal no siempre se adhiere al diseño de bases de datos. Cuando se esté diseñando una base de datos debes siempre comparar las ventajas de una forma normal más alta al trabajo que tome aplicar y mantener esa forma normal. En el caso de la tabla de clientes, personalmente escogería no normalizarla a la tercera forma normal. En último caso, lo haría. Almacenar **datos derivados**, como el resultado de un cálculo que está basado en datos existentes es usualmente una mala idea.

## Otro ejemplo: una tienda web
Esperamos que ahora estés familiarizado con los conceptos de diseño de bases de datos y que seas capaz de poder diseñar una bse de datos relacional sencilla. En el siguiente ejemplo de diseño de bases de datos recapitularemos las tareas y consideraciones que normalmente encontrarás cuando estés diseñando una base de datos relacional.

### Resumen del sistema de la tienda
Con el fin de tener una buena vista de los datos que están involucrados en el concepto de una tienda web hagamos un resumen de algunas de las tareas que normalmente hace una tienda web:

- Muestra productos
- Categoriza productos
- Registra clientes
- Guarda productos en un carrito de compras
- Muestra el contenido del carrito
- Envía órdenes al dueño de la tienda web
- etc.

### Identificar entidades y relaciones
De la lista de tareas podemos derivar las entidades que juegan un rol en el sistema de la tienda web. **Productos**, **categorías**, **clientes**, y **pedidos** son entidades encontradas en casi todas las bases de datos de las tiendas web. En este ejemplo mostraremos un modelo que sólo contenga las entidades **cliente**, **pedido** y **producto**.

Una vez hayas identificado las entidades que quieras modelar puedes analizar las relaciones existentes entre entidades:

- Entre *pedido* y *producto* hay una relación muchos-a-muchos. Cada pedido contiene 1 o más productos y cada producto puede ser asociado con 0, 1 o más pedidos. Una relación muchos-a-muchos se modela con tres tablas. Dos tablas fuentes (Pedido y Producto) y una tabla de unión (PedidoProducto), como se muestra en el modelo más adelante. Nótese como los pedidos y los productos tienen una relación uno-a-muchos con la tabla de unión. Juntos forman la relación muchos-a-muchos entre pedidos y productos

- Clientes y pedidos están enlazados en una relación uno-a-muchos. Cada registro en la tabla *cliente*  puede asociarse con múltiples registros en la tabla de *pedidos* y lo mismo a la inversa, cada registro en *pedidos* es asociado con un registro en *cliente*.

![enter image description here](https://i.imgur.com/8QmN8UD.png)

Las tablas mostradas en el modelo anterior sirven como ejemplo sencillo. Una tabla de clientes en la vida real por supuesto debe tener más datos del cliente (dirección, ciudad, etc.)

Ahora veamos algunas consideraciones de este modelo:

#### Tabla de pedidos
Cada fila en la tabla de *pedido* está enlazada a una única fila en la tabla de *cliente* con el campo de **clave foránea** cliente_id. La tabla de pedidos contiene únicamente datos que no son claves que son dependientes del *pedido_id* (clave primaria), tal como la fecha y la hora.

##### Cantidad de pedidos
El modelo actual está algo limitado. Permite que un usuario sólo agregue *un* producto de cada tipo por cada pedido. ¿Por qué? Porque la tabla PedidoProducto tiene una clave primaria compuesta que consiste del pedido_id y del producto_id. Como una clave primaria siempre debe ser única, sólo un registro de cada cierta combinación pedido_id - producto_id puede existir en la tabla.

¿Entones cómo permitimos que un usuario almacenar una cantidad de más de 1 del mismo producto en el mismo pedido? Hay dos soluciones:

La primera es eliminar la restricción de unicidad de la tabla PedidoProducto eliminando la definición de la clave primaria de la tabla (y posiblemente agregando una clave primaria sustituta). Entonces, se permitirían múltiples registros con la misma combinación de pedido_id - producto_id. Esto debería servir, pero la segunda solución es un poco mejor: agregar un campo de *cantidad* a la tabla actual de PedidoProducto. La clave primaria se mantendría y la aplicación que use esta base de datos actualizaría el campo de cantidad cuando un usuario altere la cantidad de un producto en su carrito de compras.

##### Tipo de pago
Un campo que se podría agregar a la tabla de pedidos es *tipo_pago*. Éste es único para cada pedido y no puede derivarse de otros datos. (Nótese que tipo_pago se convertiría en un campo de clave foránea en la tabla de pedidos referenciando a una tabla separada que contenga los tipos de pagos).

##### Monto total por pedido
Otro campo que podrías (y por tanto deberías) agregar a la tabla de pedidos es el *monto_total* de un pedido, esto es, el precio total. Podemos escucharte gritar: *¡eso es un dato derivado! Puedes sumar el importe de los productos individuales ¿o no?* Sí. Y no. El precio de un producto está sujeto a cambiar. Así que cuando determinas el precio total de un pedido sumando los precios de todos los productos contenidos en un pedido y si tal vez, el encargado de la tienda dobla el precio de uno de los productos que están en el pedido hace que todos los pedidos que tengan ese producto cambien de precio. Es por esto que es una biena idea calcular el precio total en el momento en que se lleva a cabo el pedido y guardarlo junto con el registro del pedido.

#####  Almacenar un historial de precios para productos
Otra solución para el problema del precio total de un pedido podría ser almacenar un historial del precio de los productos. En ese caso, podrías mirar en la tabla de pedidos y consultar la tabla *historial_precios* para buscar el precio del producto en la fecha y hora en que se llevó a cabo el pedido. En este caso no tendrías que almacenar el precio total en la tabla de pedidos. La mayoría de sistemas almacenan el precio total del pedido y no tienen un historial de precios. Pero tú eres el diseñador, así que tu decides lo que es mejor para tu sistema :)

#### Tabla de productos
En la tabla de productos se almacena el precio sin incluir el IVA. El precio incluyendo el IVA puede ser calculado del precio sin incluir el IVA por un programa o por una consulta SQL. Es por esto que no se guarda el precio del producto incluyendo el IVA. Debes ser consciente que guardar el precio de un producto de esta forma puede tener implicaciones en el futuro. En este modelo el precio del producto es almacenado en un único campo. Una vez cambie el precio del producto, el viejo precio se ha ido. Si quisieras poder tener un reporte histórico de ventas de tu base de datos deberías almacenar un historial de precios para cada producto por separado. Si un producto cambia de precio dos veces al año, necesitarás un historial de precios si quieres poder calcular la cantidad total de dinero que hiciste de un producto en ese año.

## Conclusiones
Una base de datos relacional en una pieza de equipamiento maravillosa para almacenar grandes cantidades de datos eficientemente. En este tutorial nos enfocamos principalmente en construir el modelo de la base de datos. Esos modelos pueden implementarse en cualquier RDBMS y ser consultados usando el Lenguaje de Consulta Estructura.

### ¿Dónde ir ahora?
Si quieres diseñar tu propia base de datos, asegúrate de probar a MySQL Workbench. Es una gran herramienta para crear diagramas entidad-relación. La podremos usar mucho en nuestro trabajo de desarrolladores, incluso si no estamos usando MySQL.

Otro paso lógico para tomar desde aquí es aprender más acerca de SQL. Las herramientas de modelado como [MySQL Workbench](http://www.mysql.com/products/workbench/) y las herramientas de administración como SQLyog son grandiosas, pero si en realidad quieres aprender cómo usar una base de datos, SQL es una habilidad imprescindible. La W3Schools tienen un excelente [tutorial](http://www.w3schools.com/sql/default.asp) para principiantes y hay incluso muchos otros tutoriales por los cual empezar. Por ahora llegamos hasta aquí. En un futuro estaremos incluyendo nuestro propio tutorial curado en este blog.

Happy coding!

