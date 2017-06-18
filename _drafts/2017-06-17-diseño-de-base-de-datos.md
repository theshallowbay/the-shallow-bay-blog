---
published: false
layout: post
date: 2017-06-20 20:00:00
summary: Aquí va un resumen del post
categories: tutoriales
title: Diseño de base de datos
---
Las bases de datos son programas que permiten almacenar y recuperar grandes cantidades de datos relacionados entre sí. Las bases de datos consisten en **tablas** que contienen **datos**. Cuando estés creando una base de datos debes pensar en qué **tablas** vas a crer y qué **relaciones** existen entre los datos de esas tablas. En otras palabras, tienes que pensar acerca del ***diseño*** de tu base de datos. Una base de datos **bien diseñada** asegurará la integridad y la mantenibilidad de tus datos.

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://en.tekstenuitleg.net/articles/software/database-design-tutorial/intro.html). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de esribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

## Lenguaje de consulta estrucurada SQL
(Del inglés *Structured Query Language*)

Una base de datos se crea para almacenar y recuperar datos. Esto significa que queremos ser capaces de poder **INSERT** (*INSERTAR*, en español) datos en la base de datos y también **SELECT** (*SELECCIONAR*) datos de esa base de datos.


