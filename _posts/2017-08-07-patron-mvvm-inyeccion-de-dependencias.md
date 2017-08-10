---
layout: post
date: 2017-08-07 11:54:00
summary: Hasta aquí, hemos aprendido los conceptos básicos del patrón MVVM y cómo aplicarlos en una aplicación real. Ahora veremos algunos conceptos más avanzados, que serán de mucha ayuda para implementar el patrón MVVM en proyectos reales y complejos
title: El patrón MVVM - Inyección de dependencias
categories: tutoriales
published: true
---

Hasta aquí, hemos aprendido los conceptos básicos del patrón MVVM y cómo aplicarlos en una aplicación real. Ahora veremos algunos conceptos más avanzados, que serán de mucha ayuda para implementar el patrón MVVM en proyectos reales y complejos

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://blog.qmatteoq.com/the-mvvm-pattern-dependency-injection/). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

## Creemos un lector de noticias RSS
Para entender qué es la inyección de dependencias, vamos a crear una aplicación que muestre al usuario una lista de noticias obtenidas de un [feed RSS](https://support.google.com/feedburner/answer/79408). Como ya mencionamos en la sección anterior, el primer paso es dividir la aplicación en tres diferentes componentes principales:

- El **modelo**, que es la entidad base que identifica una noticia del feed.
- La **Vista**, que es la página XAML que mostrará las noticias usando un control *ListView*
- El **ViewModel**, que se encargará de obtener las noticias del feed RSS y de pasarselos a la Vista

Como la meta del patrón MVVM es separar, tanto como sea posible, las tres diferentes capas de la aplicación, es una buena práctica evitar incluir la lógica requerida para [*parsear*](https://groups.google.com/forum/#!topic/phplatinoamerica/nBe6PQm-VVY) (analizar sintácticamente) el feed RSS directamente en el ViewModel. Una mejor solución es dejar manejar esta tarea a una clase específica (la llamaremos `ServicioRSS`), que se encargará de descargar el XML del feed RSS, convertirlo en una colección de objetos y devolvérselos al ViewModel.

Empecemos con el modelo y la defición de una clase, llamada **ItemFeed**, que identifica a un único elemento del feed:

{% highlight c# %}
    public class ItemFeed
    {
        public string Titulo { get; set; }
        public string Descripcion { get; set; }
        public string Enlace { get; set; }
        public string Contenido { get; set; }
        public DateTime FechaPublicacion { get; set; }
    }
{% endhighlight %}

Ahora podemos crear el servicio que, aprovechando LINQ a XML (un poderoso lenguaje para manipular archivos XML incluído en el framework .NET), puede parsear el feed RSS y convertirlo en una colección de objetos `ItemFeed`:

{% highlight c# %}
    public class ServicioRSS
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
{% endhighlight %}

La clase `ServicioRSS` tiene un método asíncrono que usa la clase `HttpClient` y su método *GetStringAsync()* para descargar el contenido del feed RSS. Luego, usando LINQ a XML, convertimos el XML en una colección de objetos *ItemFeed*: cada nodo del archivo XML (como *título*, *descripción*, *enlace*, etc.) es guardado en una propiedad del objeto *ItemFeed*.

Ahora tenemos una clase que acepta la URL de un feed RSS de entrada y devuelve una lista de objetos *ItemFeed*, que podemos manipular dentro del ViewModel. Así es como luce el ViewModel de la página que muestra las noticias:

{% highlight c# %}
    public class MainViewModel : ViewModelBase
    {
	    private List<ItemFeed> _noticias;
	    public List<ItemFeed> Noticias
	    {
		    get { return _noticias; }
		    set { Set(ref _noticias, value); }
		}
		
		private RelayCommand _comandoCargar;
		public RelayCommand ComandoCargar
		{
			get
			{
				if (_comandoCargar == null)
				{
					_comandoCargar = new RelayCommand(async () =>
					{
						ServicioRSS servicioRss = new ServicioRSS();
						List<ItemFeed> items = await servicioRss.ObtenerNoticias("https://theshallowbay.github.io/feed.xml");
						Noticias = items;
					});
				}
				
				return _comandoCargar;
			}
		}
	}
{% endhighlight %}	    

Hemos reutilizado el conocimientos adquirido previamente para:

1. Definir una propiedad llamada *Noticias*, cuyo tipo es `List<ItemFeed>`, donde vamos a almacenar la lista de noticias, que se mostrarán en la página.
2. Hemos definido un comando, que podemos conectar a un evento (como el click de un botón), que crea una nueva instancia de la clase ServicioRSS y llama al método  *ObtenerNoticias()*.

Al final, podemos crear la página XAML:

{% highlight html %}
```
    <Page
    x:Class="UniverseLector.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:UniverseLector"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    DataContext="{Binding Source={StaticResource Locator}, Path=Main}"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <ListView ItemsSource="{Binding Path=Noticias}">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <StackPanel>
                        <TextBlock Text="{Binding Path=FechaPublicacion}" Style="{StaticResource SubtitleTextBlockStyle}" />
                        <TextBlock Text="{Binding Path=Titulo}" Style="{StaticResource TitleTextBlockStyle}" />
                    </StackPanel>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </Grid>

    <Page.BottomAppBar>
        <CommandBar>
            <CommandBar.PrimaryCommands>
                <AppBarButton Icon="Refresh" Label="Cargar" Command="{Binding Path=ComandoCargar}" />
            </CommandBar.PrimaryCommands>
        </CommandBar>
    </Page.BottomAppBar>
 </Page>
```
{% endhighlight %}

En la página hemos agregado:

1. Un control *ListView*, cuya propiedad *ItemsSource* está conectada, usando binding, a la propiedad **Noticias** del ViewModel. Cuando el ViewModel usa el método *ServicioRSS* para cargar dentro de la propiedad *Noticias* la lista de noticias, el control automáticamente actualizará su diseño visual para mostrarlas. Específicamente, hemos definido un *ItemTemplate* que muestra la fecha de publicación de la noticia y su título.
2. Un *AppBarButton* en la barra de comandos de la aplicación, que está conectado a la propiedad **ComandoCargar**. Cuando se presiona el botón, el ViewModel descargará las noticias desde el feed RSS usando la clase *ServicioRSS*.

![](http://i.imgur.com/z0Lhld5.gif)

Ahora que tenemos nuestra aplicación de ejemplo lista y corriendo, podemos introducir algunos nuevos conceptos que nos ayudarán a hacerla mejor.

## Inyección de dependencias
Digamos que, en algún punto, necesitamos reemplazar la clase *ServicioRSS* con otra que, en lugar de obtener datos de un feed RSS real, genera datos falsos para mostrar dentro de la aplicación. Hay varias razones para hacer esto: por ejemplo, un diseñador puede tener el requisito de trabajar en la interfaz y necesita simular escenarios "límites", como una noticia con un título muy largo. O tal vez nuestra aplicación maneja muchos datos complejos y, en lugar de un feed RSS, tenemos una base de datos o un servicio en la nube que pueden ser dificiles de configurar solo para propósitos de prueba.

En nuestra aplicación de ejemplo, no es tan difícil lograr esto. Solamente necesitamos crear una nueva clase (por ejemplo, *ServicioRSSFalso*) que devuelve un conjunto de objetos **ItemFeed** falsos, como el siguiente:

{% highlight c# %}
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using UniverseLector.Models;

namespace UniverseLector.Services
{
    public class ServicioRSSFalso
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
}
```
{% endhighlight %}

 Ahora necesitamos cambiar nuestro código a ejecutar cuando se ejecute el *CargarComando* para usar esta nueva clase como reemplazo de la anterior:
 
 {% highlight c# %}
 ```
 public class MainViewModel : ViewModelBase
    {
        private ObservableCollection<ItemFeed> _noticias;
        public ObservableCollection<ItemFeed> Noticias
        {
            get { return _noticias; }
            set { Set(ref _noticias, value); }
        }
        private RelayCommand _comandoCargar;
        public RelayCommand ComandoCargar
        {
            get
            {
                if (_comandoCargar == null)
                {
                    _comandoCargar = new RelayCommand(async () =>
                    {
                        ServicioRSSFalso servicioRss = new ServicioRSSFalso();
                        List<ItemFeed> items = await servicioRss.ObtenerNoticias("http://theshallowbay.github.io/feed.xml");
                        Noticias = new ObservableCollection<ItemFeed>(items);
                    });
                }

                return _comandoCargar;
            }
        }
    }
    
```
{% endhighlight %}

![](https://i.imgur.com/qqHIL5d.gif)

Hasta aquí, las cosas han sido un poco sencillas. Sin embargo, digamos que estamos trabajando en una aplicación mucho más compleja que, en lugar de tener una única página que usa la clase *ServicioRSS*, tiene 30 páginas, lo que significa que tenemos 30 ViewModels que están usando la clase *ServicioRSS*. Consecuentemente, cuando necesitemos reemplazarlos con la clase falsa que recién hemos creado, necesitaríamos editar cada uno de los ViewModel y cambiar la clase usada. Puedes imaginar fácilmente cómo esto quitaría mucho tiempo y, en mayor medida, es más propenso a introducir errores. Solo basta con olvidar reemplazar de nuevo a la clase original en uno de los ViewModels cuando hayamos terminado la fase de pruebas para crear un grandísimo problema.

Entonces señoras y señores, presentamos el concepto de **inyección de dependencias**, que es una forma de manejar este escenario de una mejor manera. El problema que acabamos de describir radica del hecho que, en el enfoque anterior, la clase *ServicioRSS* (que podemos considerar como una dependencia del ViewModel, porque él no puede trabajar apropiadamente sin ella) fue creada en el tiempo de construcción. Con la inyección de dependencias, podemos cambiar este enfoque y empezar a resolver esas dependencias en tiempo de ejecución, que es cuando la aplicación se está ejecutando.

Esto es posible gracias a una clase especial, llamada **container**, que podemos considerar como una caja: dentro de ella, vamos a registrar todos los ViewModels y todas las dependencias que requiere nuestra aplicación. Cuando la alicación se está ejecutando y necesitemos una nueva instancia de un ViewModel (simplemente porque el usuario está navegando a una nueva página), ya no la vamos crear manualmente, sino que le diremos al *container* que lo haga por nosotros. El container revisará si puede resolver todas las dependencias (lo que significa que todos los servicios usados por los ViewModels estén disponibles) y, si la respuesta es sí, devolverá una nueva instancia del ViewModel con todas sus dependencias ya resueltas y listas para usar. Ténicamente hablando, decimos que las dependencias son "inyectadas" en el ViewModel: es por esto que esta ténica se llama *inyección de dependencias*.

¿Por qué este enfoque resuelve el problema anterior, que es evitar cambiar todos los ViewModels si necesitamos cambiar la implementación de un servicio? Porque la conexión entre el ViewModel y el servicio es manejada por el container, que es un única instancia que atraviesa toda la aplicación. Cada ViewModel no tendrá que crear manualmente una nueva instancia de la clase *ServicioRSS*, sino que irá automáticamente incluída en el constructor del ViewModel. Es trabajo del container crear una nueva instancia de la clase *ServicioRSS* y pasársela al ViewModel, de forma que la pueda usar. Cuando necesitemos reemplazar la clase *ServicioRSS* con *ServicioRSSFalso*, solo necesitaremos cambiar la clase registrada en el container y, automáticamente, todos los ViewModels empezarán a usar la nueva.

Para usar apropiadamente usar este enfoque, hay un par de cambios que debemos hacer a nuestra aplicación. Veamos.

### Definir una interfaz
El primer problema en nuestro código es que *ServicioRSS* es una clase y, como tal, no tenemos una manera fácil de intercambiarla con otra en el container. Para hacer eso, necesitamos algo que describa de una forma abstracta la clase *ServicioRSS* y la operación que puede llevar a cabo, de forma que podamos usar ambas clases *ServicioRSS* y *ServicioRSSFalso*.

Para esto es que se utilizan las interfaces: su propósito es describir un conjunto de propiedades y métodos que, al final, serán implementados en una clase real. El primer paso, consecuentemente, es crear una interfaz que describa las operaciones que lleva a cabo nuestro servicio RSS. La misma interfaz será implementada tanto por el servicio real (la clase *ServicioRSS*) y el falso (la clase *ServicioRSSFalso*). Después de que hagamos esta modificación, podremos:

1. Referenciar el servicio en el ViewModel usando la interfaz en lugar de la clase real
2. Registrar en el container la conexión entre la interfaz y la clase real que queremos usar. En tiempo de ejecución, el container inyectará dentro del ViewModel la implementación que hayamos escogido. Si queremos intercambiarlas, solo necesitamos registrar en el container otra clase con la misma interfaz.

Empecemos creando una interfaz llamada *IServicioRSS*:

{% highlight c# %}
```
public interface IServicioRSS
    {
        Task<List<ItemFeed>> ObtenerNoticias(string url);
    }
```
{% endhighlight %}

Nuestro servicio, por el momento, expone solamente una operación asincrónica: **ObtenerNoticias()**, que toma la URL de entrada del feed RSS y devuelve, como salida, una colección de objetos **ItemFeed**.

Ahora tenemos que cambiar nuestras clases *ServicioRSS* y *ServicioRSSFalso* para que implementen esta interfaz, solo debemos agregar, después del nombre de la clase, dos puntos y el nombre de la interfaz, como si se tratara de una herencia de otra clase:

{% highlight c# %}
    public class ServicioRSS : IServicioRSS
    {
    ...
    }
    
    public class ServicioRSSFalso : IServicioRSS
    {
    ...
    }
{% endhighlight %}

Como puedes ver, ambos servicios están usando la misma interfaz e implementan el mismo método: la única diferencia es que la clase *ServicioRSS* devuelve datos reales, mientras que *ServicioRSSFalso* está creando manualmente un conjunto de objetos **ItemFeed** falsos.

### El ViewModel
Hay un par de cambios que necesitamos hacerle a nuestro ViewModel:

1. Ya no necesitamos crear manualmente una nueva instancia de la clase *ServicioRSS* en el comando, pero necesitamos agregarla como parámetro en el constructor del ViewModel. Necesitamos referenciar la interfaz `IServicioRSS`, o de otra forma no podremos intercambiar fácilmente entre diferentes implementaciones
2. Necesitamos cambiar *ComandoCargar* para poder usar esta nueva instancia del servicio.

Así es como el ViewModel actualizado luce:

{% highlight c# %}
```
public class MainViewModel : ViewModelBase
    {

        private readonly IServicioRSS _servicioRSS;

        public MainViewModel(IServicioRSS servicioRSS)
        {
            _servicioRSS = servicioRSS;
        }

        private ObservableCollection<ItemFeed> _noticias;
        public ObservableCollection<ItemFeed> Noticias
        {
            get { return _noticias; }
            set { Set(ref _noticias, value); }
        }

        private RelayCommand _comandoCargar;
        public RelayCommand ComandoCargar
        {
            get
            {
                if (_comandoCargar == null)
                {
                    _comandoCargar = new RelayCommand(async () =>
                    {
                        List<ItemFeed> items = await _servicioRSS.ObtenerNoticias("http://theshallowbay.github.io/feed.xml");
                        Noticias = new ObservableCollection<ItemFeed>(items);
                    });
                }

                return _comandoCargar;
            }
        }
    }
```
{% endhighlight %}

### El ViewModelLocator
El último paso y el más importante, el cual es crear el container y registrar todos los ViewModels con sus dependencias. Típicamente esta configuración se hace cuando la aplicación se lanza, así que, cuando usamos MVVM Light y el enfoque con ViewModelLocator, el mejor lugar para hacerlo es en el ViewModelLocator mismo, porque es la clase que se ocupa de despachar todos los ViewModels.

Así es como nuestra nueva implementación del ViewModelLocator luce:

{% highlight c# %}
```
public class ViewModelLocator
    {
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(() => SimpleIoc.Default);

            SimpleIoc.Default.Register<IServicioRSS, ServicioRSS>();
            SimpleIoc.Default.Register<MainViewModel>();
        }

        public MainViewModel Main
        {
            get
            {
                return ServiceLocator.Current.GetInstance<MainViewModel>();
            }
        }
    }
```
{% endhighlight %}

Lo primero es, que necesitamos resaltar que este código es solamente un ejemplo: hay muchas librerías disponibles para implementar el enfoque de inyección de dependencias, como Ninject o LightInject. Sin embargo, la mayoría de los toolkits y frameworks MVVM proveen también una infraestructura para manejar estos escenarios y MVVM Light no es la excepción, al proveer un container simple identificado por la clase **SimpleIoc**, que es configurada como container por defecto por la aplicación al usar el método *SetLocationProvider()* de la clase **ServiceLocator**.

El siguiente paso es registrar todos los ViewModels y sus dependencias en el container. La clase *SimpleIoc* ofrece el método *Register()* para lograr esto, que puede hacerse de dos maneras:

1. La versión `Register<T>()`, que es usada cuando sólo necesitamos una nueva instancia de la clase que no está descrita por una interfaz. Es el caso de nuestros ViewModels porque, en nuestro contexto, no necesitamos una manera de intercambiarlas completamente.
2. La versión `Register<T, Y>()`, que es usada, en cambio, cuando necesitemos registrar una clase que esté descrita por una interfaz. En este caso, especificamos como `T` la interfaz y a `Y` su implementación que queremos usar.

Al final, necesitamos cambiar la forma como la clase *ViewModelLocator* define la propiedad **Main**: en lugar de crear manualmente un nuevo objeto *MainViewModel*, le preguntamos al container que nos de una nueva instancia de la clase, al usar el método `GetInstance<T>` (donde `T` es el tipo que necesitamos).

Después de haber terminado esos cambios, esto es lo que pasará cuando lancemos nuestra aplicación de nuevo:

1. La aplicación creará una nueva instancia de la clase *ViewModelLocator*, que iniciará la configuración del container. La clase *SimpleIoc* registrará las clases *MainViewModel* y *ServicioRSS* (la segunda, descrita por la interfaz `IServicioRSS`).
2. La aplicación disparará la navegación a la página principal de la aplicación. Consecuentemente, como la propiedad `DataContext` de la *Page* está conectada a la propiedad `Main` del *ViewModelLocator*, el locator intentará crear una nueva instancia del ViewModel.
3. El container definido en la clase *ViewModelLocator* comprobará si hay una clase registrada cuyo tipo sea *MainViewModel*. La respuesta es sí, sin embargo la clase depende de una clase que implementa la interfaz *IServicioRSS*. Consecuentemente, el container empezará otra búsqueda y comprobará si hay una clase conectada a la interfaz registrada *IServicioRSS*. También en este caso la respuesta es sí: el container devolverá una nueva instancia de la clase *MainViewModel* con la implementación *ServicioRSS* lista para ser usada.

Si alguna de las condiciones previas no se satisfacen (por ejemplo, el container no encuentra una clase registrada para la interfaz *IServicioRSS*), obtendremos una excepción, porque el container no pudo resolver todas las dependencias. Ahora que hemos alcanzado el fin de nuestro viaje, debes ser capaz de entender porqué la inyección de dependencias es extemadamente útil para nuestro escenario. Tan pronto como hagamos algunas pruebas y queramos usar la clase *ServicioRSSFalso* en reemplazo de *ServicioRSS*, solo necesitaríamos cambiar una línea de código en la clase *ViewModelLocator*. En lugar de:

{% highlight c# %}
    SimpleIoc.Default.Register<IServicioRSS, ServicioRSS>();
{% endhighlight %}

tendríamos que escribir:

{% highlight c# %}
    SimpleIoc.Default.Register<IServicioRSS, ServicioRSSFlaso>();
{% endhighlight %}

Gracias a este nuevo enfoque, no importa si nuestra aplicación tiene un solo ViewModel o si tiene 50 que dependen de la interfaz `IServicioRSS`: automáticamente, el container se encargará de intectar la clase real apropiada a cualquiera de ellos.

Este enfoque es extremadamente útil no solo cuando estamos haciendo pruebas, sino también si necesitamos hacer refactorización de código. Digamos que, en algún punto, nuestra aplicación no tiene que recuperar noticias de ningún feed RSS, sino de un servicio REST publicado en la nube. Como nuestro servicio seguirá la misma estructura definida por la interfaz `IServicioRSS` (en nuestro caso, un método *GetNews()* que devuelve una colección de objetos **ItemFeed**) no tendremos que cambiar nada en los ViewModels. Solo necesitamos crear una nueva clase que implemente la interfaz `IServicioRSS` y registrarla en el container, en reemplazo del *ServicioRSS*.

## Para terminar
Estamos alcanzando el final de nuestro viaje de aprendimiento. En el próximo post veremos algunos escenarios avanzados, como enviar mensajes y manejar eventos secundarios.

Happy coding!

**El patrón MVVM: Series**
1. [Introducción](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/)
2. [La práctica](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/)
3. [Inyección de dependencias](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/)
4. [Escenarios avanzados](https://theshallowbay.github.io/tutoriales/2017/08/08/patron-mvvm-escenarios-avanzados/)
5. [Servicios, helpers y plantillas](https://theshallowbay.github.io/tutoriales/2017/08/09/patron-mvvm-servicios-helpers-plantillas/)
6. [Datos en tiempo de diseño](https://theshallowbay.github.io/tutoriales/2017/08/09/patron-mvvm-datos-tiempo-diseno/)