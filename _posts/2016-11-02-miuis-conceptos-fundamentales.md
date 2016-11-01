---
layout: post
title: miuis - conceptos fundamentales
date: 2016-11-02 12:32:18
summary: Las primeras pinceladas de lo que será nuestra primera aplicación: llevar de forma eficiente el control de las notas de las asignaturas vistas en la universidad. Como estudiantes, sabemos lo importantes que son las notas, lo importantísimo que es mantener el promedio y sobre todo, lo importante que es llevar un orden. ¿Hay una aplicación para eso?
categories: categorías de la publicación

---

Hay una necesidad que no ha sido cubierta (bueno sí, pero de forma parcial). Al finalizar el semestre, queremos saber qué tan bien o qué tan mal nos ha ido, por lo que procedemos a realizar el cálculo con los porcentajes de cada nota de cada asignatura. Es fácil hacerlo a lápiz y papel, si estamos en los primeros semestres de la carrera, pero cuando ya hemos avanzado un tiempo, se hace más laborioso el cálculo y toca recurrir a herramientas como el *Excel* y sus fórmulas. Así que de ahí surgió la idea de crear una aplicación que haga incluso más fácil este cálculo.

## Motivación
Atrás en el 2013, unos chicos de la universidad [lanzaron](https://www.facebook.com/uisenlinea/posts/10152928297414558) una aplicación llamada **Uisers**, que permite precisamente eso, calcular la nota final de la asignatura en base a los porcentajes y las notas de los exámenes. Fue lanzada para [Android](https://play.google.com/store/apps/details?id=co.tuister.uisers&hl=es_419) y para [iOS](https://itunes.apple.com/us/app/id923410614), pero al parecer no se actualiza desde hace más de año y medio. Tampoco existe una versión para Windows, por lo que consideramos conveniente ponernos manos a la obra.

![Uisers para Andriod](https://lh5.ggpht.com/8mQctaSsIMFv2xdwbI09OdxMKa-5rDghM1ps6lHTbEQwZIf1Sf382Ni89omBeOqczA4=h900-rw)

El objetivo principal de este proyecto será crear una aplicación universal para Windows que permita a los estudiantes llevar el control de los registros de sus calificaciones, calcular el promedio semestral, calcular el promedio ponderado, recibir información importante de la universidad y tenerlo todo sincronizado entre dispositivos.

## Let's dev this
El proceso de desarrollo será totalmente transparente, por lo que la aplicación será de código abierto, para que cualquiera que lo desee pueda revisar en las entrañas del código, por simple curiosidad o por altruismo y ayudar con el desarrollo encontrando bugs y errores o aportando sugerencias para mejorar la calidad de la aplicación.

La aplicación se llamará **miuis** y el código fuente está disponible [aquí](https://github.com/theshallowbay/miuis).

![miuis](https://i.imgur.com/PUNXSNT.png)

## Requerimientos
- Queremos ofrecer una *Experiencia de Usuario (UX)* sencilla, amigable, completa y fácil de entender. 
- Todos los datos generados en la aplicación serán totalmente privados y pertenecerán únicamente al usuario. La base de datos estará cifrada con una llave que sólo el usuario tiene.
- Es necesario que el usuario se registre en el servicio antes de poder utilizar la aplicación, para esto se facilitará un registro fácil y rápido con la cuenta Microsoft que el usuario ya tiene en el sistema.
- La aplicación deberá ser capaz de *sincronizar los datos (en concreto, la base datos)* entre distintos dispositivos. Para esto, se hará uso del almacenamiento en OneDrive de la cuenta Microsoft del usuario.
- Debe funcionar en cualquier escenario: tanto *online* como *offline*.

## Funcionalidades
Con esta aplicación podrás:

 - Anotar tus materias y horarios de clases
 - Navegar en el mapa del campus
 - Calcular la nota definitiva de las asignaturas en base a las notas parciales
 - Calcular el promedio del semestre
 - Conocer el promedio ponderado en base a las notas de los semestres anteriores
 - Inferir la calificación necesaria para aprobar la asignatura

## Bocetos iniciales
La interfaz, lo que el usuario puede *ver* es solo la punta del iceberg, pero es casi lo más importante. La aplicación tiene que entrar por los ojos y gustar. Pronto estaremos publicando los primeros bocetos de la interfaz.


Con estas ideas en mente, iniciamos el desarrollo de miuis.

Happy coding!

