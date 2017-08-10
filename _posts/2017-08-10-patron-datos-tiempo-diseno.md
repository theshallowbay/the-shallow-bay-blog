---
layout: post
date: 2017-08-10 16:36:00
summary: Hablemos acerca de una de las características más interesantes de trabajar con el patrón MVVM datos en tiempo de diseño.
title: El patrón MVVM - Datos en tiempo de diseño
categories: tutoriales
published: true
---

Hablemos acerca de una de las características más interesantes de trabajar con el patrón MVVM: datos en tiempo de diseño.

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://blog.qmatteoq.com/the-mvvm-pattern-design-time-data/). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.


En [un post](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/) anterior hablamos acerca de la inyección de dependencias, la cual es una forma sencilla de intercambiarla implementación de un servicio usado por el ViewModel. Hay varias razones para hacerlo: para hacer refactorización de código más fácil, para reemplazar la fuente de datos de manera rápida, etc. Hay otro escenario en que la inyección de dependencias puede ser muy útil: ayudar a los diseñadores a definir la interfaz de usuario de la aplicación. Si bien este requisito es sencillo de satisfacer cuando se trata de datos estáticos (como el encabezado de una página), que es renderizado en tiempo real por el diseñador de Visual Studio, las cosas se ponen un poco más complicadas cuando se trata de datos dinámicos.

Es muy probable que la mayoría de los datos mostrados en una aplicación no sean estáticos, sino que son cargados cuando la aplicación se esté ejecutando: desde un servicio *REST*, desde una base de datos local, etc. Por defecto, todos esos escenarios no funcionan cuando la página XAML se muestra en el diseñador, dado que la aplicación efectivamente *no* se está ejecutando. Para solucionar este problema, XAML introdujo el concepto de **datos en tiempo de diseño**: es una forma de definir una fuente de datos para que se muestre solo cuando la página XAML está siendo mostrada en Visual Studio o en Blend en modo diseño, incluso si la aplicación no se está ejecutando.

El patrón MVVM hace fácil soportar este escenario, gracias a la separación de las capas proveídas por el patrón y por el enfoque de la inyección de dependencias: es suficiente con intecambiar en el container de dependencias el servicio que provee los datos reales de la aplicación con uno falso, que crea un conjunto de datos de ejemplo estáticos.

Sin embargo, comparado al ejemplo que vimos en el post anterior de inyección de dependencias, hay varios cambios que hay que hacer. Específicamente, necesitamos detectar cuando la página XAML está siendo mostrada en el diseñador y cuando está en tiempo de ejecución para que cargue los datos correctos desde la fuente. Para hacer esto, usamos de nuevo una de las características ofrecidas por el toolkit MVVM Light, que es una propiedad ofrecida por la clase `ViewModelBase` que nos dice si hay una clase que esté siendo usada por el diseñador o por la aplicación en ejecución.

Veamos en detalle los cambios que tenemos que hacer. Vamos a usar el mismo ejemplo que vimos en el post de inyección de dependencias. La aplicación es sencilla: muestra una lista de noticias, obtenida de un feed RSS. En ese post anterior implementamos una interfaz, llamada `IServicioRss`, que ofrece un método con la siguiente firma:

{% highlight c# %}
    public interface IRssService
    {
        Task<List<FeedItem>> ObtenerNoticias(string url);
    }
{% endhighlight%}

Entonces, la interfaz es implementada por dos clases: una llamada *ServicioRSS*, que provee los datos reales desde un feed RSS real, y otra llamada *ServicioRSSFalso*,  que provee datos estáticos falsos.

{% highlight c# %}
    public class ServicioRSS : IServicioRSS
    {
        public async Task<List<ItemFeed>> ObtenerNoticias(string url)
        {
            HttpClient cliente = new HttpClient();
            string resultado = await cliente.GetStringAsync(url);
            var xdoc = XDocument.Parse(resultado);
    
            return (from item in xdoc.Descendants("item")
                    select new ItemFeed
                   {
                       Titulo = (string)item.Element("title"),
                       Descripcion = (string)item.Element("description"),
                       Enlace = (string)item.Element("link"),
                       FechaPublicacion = DateTime.Parse((string)item.Element("pubDate"))
                   }).ToList();
                      
        }
    }
    
    public class ServicioRSSFalso : IServicioRSS
    {
        public Task<List<ItemFeed>> ObtenerNoticias(string url)
        {
            List<ItemFeed> items = new List<ItemFeed>
            {
                new ItemFeed
                {
                    FechaPublicacion = new DateTime(2016, 6, 26),
                    Titulo = "Noticia 1"
                },
                new ItemFeed
                {
                    FechaPublicacion = new DateTime(2016, 11, 1),
                    Titulo = "Noticia 2"
                },
                new ItemFeed
                {
                    FechaPublicacion = new DateTime(2017, 6, 20),
                    Titulo = "Noticia 3"
                },
                new ItemFeed
                {
                    FechaPublicacion = new DateTime(2017, 7, 2),
                    Titulo = "Noticia 4"
                },
                new ItemFeed
                {
                    FechaPublicacion = new DateTime(2017, 7, 4),
                    Titulo = "Noticia 5"
                }
            };
    
            return Task.FromResult(items);
        }
    }
{% endhighlight%}

Antes de empezar a explorar los cambios que necesitamos hacer en la aplicación para que soporte los datos en tiempo de diseño, habría que resaltar una posible solución para manejar operaciones asíncronas. Uno de los desafíos de crear datos falsos viene del hecho que, típicamente, los servicios reales usan métodos asíncronos (dado que obtienen los datos de una fuente que puede tomarse su tiempo en procesarlos). El *ServicioRSS* es un buen ejemplo: como el método `ObtenerNoticias()` es asíncrono, tiene que devolver un objeto `Task<T>`, de forma que pueda ser apropiadamente llamado por nuestro ViewModel usando la palabra clave *await*. Sin embargo, es muy improbable que el servicio falso necesite usar métodos asíncronos: solo devueve datos estáticos. El problema es que, como los dos servicios implementan la misma interfaz, no podemos tener un servicio que devuelva operaciones `Task` mientras el otro devuelva objetos planos. Como solución, como puedes ver dle código de ejemplo, se puede usar el método `FromResult()`  de la clase `Task`. Su propósito es encapsular dentro de un objeto  `Task` una respuesta simple. En este caso, como el método `ObtenerNoticias()` devuelve una respuesta `Task<List<ItemFeed>>`, creamos una colección falsa de `List<ItemFeed>` y se la pasamos al método `Task.FromResult()`. De esta forma, incluso si el método no es asíncrono, se comportará como si lo fuera, y así podemos mantener la misma firma definida por la interfaz.

### El ViewModeLocator
El primer cambio que vamos a hacer es en el *ViewModelLocator*. En nuestra aplicación de ejemplo tenemos el siguiente código, que registra en el container de dependencias la interfaz `IServicioRss` con la implementación de `ServicioRSS`:

{% highlight c# %}
    public class ViewModelLocator
    {
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(() => SimpleIoc.Default);

            SimpleIoc.Default.Register<IServicioRSS, ServicioRSS>();
            SimpleIoc.Default.Register<MainViewModel>();
        }

        public MainViewModel Main => ServiceLocator.Current.GetInstance<MainViewModel>();
    }
{% endhighlight%}

Necesitamos cambiar el código de tal forma que, basado en la forma en que la aplicación está siendo renderizada, se use el servicio apropiado. Podemos usar la propiedad `IsInDesignModeStatic` ofrecida por la clase `ViewModelBase` para detectar si la aplicación está siendo ejecutada o renderizada por el diseñador:

{% highlight c# %}
    public class ViewModelLocator
    {
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(() => SimpleIoc.Default);
            
            if (ViewModelBase.IsInDesignModeStatic)
            {
                SimpleIoc.Default.Register<IServicioRSS, ServicioRSSFalso>();
            }
            else
            {
                SimpleIoc.Default.Register<IServicioRSS, ServicioRSS>();
            }

            SimpleIoc.Default.Register<MainViewModel>();
        }

        public MainViewModel Main => ServiceLocator.Current.GetInstance<MainViewModel>();
    }
{% endhighlight%}

En el caso de que la aplicación está siendo renderizada en el diseñador, conectamos la interfaz `IServicioRSS` con la clase `ServicioRSSFalso`, que devuelve datos falsos. Si la aplicación se está ejecutando, en cambio, conectamos la interfaz `IServicioRSS` con la clase `ServicioRSS`.

### El ViewModel
Para poder hacer uso de los datos en tiempo de diseño, también necesitamos cambiar un poco el ViewModel. La razón es que, cuando la aplicación está siendo renderizada en el diseñador, en realidad no se está ejecutando; el diseñador se encarga de inicializar todas las clases requeridas (como el *ViewModelLocator* o los diferentes ViewModels), pero no ejecuta todos los eventos de la página. Como tal, dado que típicamente la aplicación carga los datos basándose en los eventos como `OnNavigatedTo()` o `Loaded`, nunca los veremos en el diseñador. Nuestra aplicación es un buen ejemplo de este escenario: en nuestro ViewModel tenemos un `RelayCommand` llamado *ComandoCargar*, que se encarga de obtener los datos del *ServicioRSS*:

{% highlight c# %}
        private RelayCommand _comandoCargar;
        public RelayCommand ComandoCargar
        {
            get
            {
                if (_comandoCargar == null)
                {
                    _comandoCargar = new RelayCommand(async () =>
                    {
                        List<ItemFeed> items = await _servicioRSS.ObtenerNoticias("https://theshallowbay.github.io/feed.xml");
                        Noticias = new ObservableCollection<ItemFeed>(items);
                    });
                }

                return _comandoCargar;
            }
        }        
{% endhighlight%}

Al usar el Behaviors SDK, tal como se describió en este [otro](https://theshallowbay.github.io/tutoriales/2017/08/08/mvvm-escenarios-avanzados/#manejando-eventos-adicionales-con-comandos) post, conectamos este comando con el evento `Loaded` de la página:

Sin embargo, cuando la página está siendo renderizada en el diseñador, el comando `ComandoCargar` nunca es invocado, dado que el evento `Loaded` de la página nunca es lanzado. Por eso, tenemos que obtener los datos de nuestro servicio también en el constructor del ViewModel que, en cambio, es ejecutado cuando el diseñador crea una instancia de nuestro ViewModel. Pero, necesitamos hacerlo **solo** cuando el ViewModel está siendo renderizado por el diseñador: cuando la aplicación se esté ejecutando normalmente, es correcto dejar la operación de cargado de datos al comando. Para lograr esto, usamos la propiedad `IsInDesign`, que es parte de la clase `ViewModelBase` que ya hemos usado como clase base para nuestro ViewModel:

Solo si la aplicación está en modo de diseño, obtenemos los datos desde el servicio y poblamos la propiedad `Noticias`, que es la colección conectada al control *ListView* en la página. Como el métodon `ObtenerNoticias()` es asíncrono y no se pueden llamar métodos asíncronos suando el operador *async / await* en el constructor, primero necesitamos llamar al método `Wait()` en la `Task` y luego acceder a la propiedad `Result` para obtener la lista de objetos `ItemFeed`. En una aplicación real este enfoque llevaría a una llamada síncrona, que bloquearía el UI Thread. Sin embargo, como nuestro `ServicioRSSFalso` no es realmente asíncrono, no va a tener ningún efecto secundario.

Este ejemplo muestra también la razón por la cual, en caso de que quisiéramos hacer las cosas sencillas, no podemos llamar al método `ObtenerNoticias()` en el constructor cuando la aplicación se esté ejecutando: dado que no podemos usar los operadores *async / await*, terminaríamos con comportamientos impredecibles. Por tal razón, es correcto seguir llamado a los métodos que cargan los datos en los eventos de la página que son disparados cuando la página está siendo carga, o navegada: como son métodos o manejadores de eventos simples, pueden ser usados con los operadores async /await.

## ¡Estamos listos!
Ahora el trabajo está terminado. Si lanzamos la aplicación, vamos a continuar viendo los daros que vienen del feed RSS. 

![](https://i.imgur.com/nlUSRES.png)

Pero, si abrimos la página *PaginaPrincipal.xaml* en el diseñador de Visual Studio o en Blend, vamos a ver algo como esto:

![](https://i.imgur.com/WzdK9SP.png)

El diseñador ha creado una instancia de nuestro ViewModel, que obteniene desde el container de dependencias una instancia de `ServicioRSS`. Como el ViewModel está corriendo en modo de diseño, va a ejecutar el código que está escrito en el constructor, que obtendrá los datos falsos. Genial, ¿no? gracias a esta implementación, podemos ver fácilmente como nuestra colección de noticias se verá al final, y, si no nos satisface, podemos cambiar el `DataTemplate` que hemos definido en la propiedad `ItemTemplate`.

Este ha sido el fin de nuestro viaje. Si has llegado hasta acá te felicitamos, sigue adelante y recuerda: *keep hustling!*


**El patrón MVVM: Series**
1. [Introducción](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/)
2. [La práctica](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/)
3. [Inyección de dependencias](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/)
4. [Escenarios avanzados](https://theshallowbay.github.io/tutoriales/2017/08/08/patron-mvvm-escenarios-avanzados/)
5. [Servicios, helpers y plantillas](https://theshallowbay.github.io/2017/08/09/patron-mvvm-servicios-helpers-plantillas/)
6. [Datos en tiempo de diseño](https://theshallowbay.github.io/tutoriales/2017/08/09/patron-datos-tiempo-diseno/)
