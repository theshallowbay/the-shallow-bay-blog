---
layout: post
date: 2017-08-09 17:09:00
summary: En este post vamos a introducir algunos conceptos y librerías que pueden hacer nuestra vida más fácil cuando estemos desarrollando aplicaciones universales UWP usando el patrón MVVM
title: El patrón MVVM - Servicios, helpers y plantillas
categories: tutoriales
published: true
---

En este post vamos a introducir algunos conceptos y librerías que pueden hacer nuestra vida más fácil cuando estemos desarrollando aplicaciones universales UWP usando el patrón MVVM.

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://blog.qmatteoq.com/the-mvvm-pattern-services-helpers-and-templates/). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

## Servicios, servicios, servicios
En [uno](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/#creemos-un-lector-de-noticias-rss) de los posts anteriores creamos una aplicación de ejemplo que mostraba una lista de noticias obtenidas de un feed RSS. Mientras estuvimos desarrollando esa aplicación introdujimos el concepto de *servicio*: una clase que se encarga de llevar a cabo operaciones y pasar los resultados al ViewModel. Los servicios pueden ser útiles también para alcanzar otra meta importante en el uso del patrón MVVM: evitar escribir código específico de plataforma directamente en los ViewModel, para hacer más fácil después compartirlo con otras aplicaciones o plataformas. Como es usual, vamos a explicar los conceptos con ejemplos reles, así que empecemos con uno nuevo,

Digamos que estás desarrollando una increíble aplicación que necesita mostrar un cuadro de diálogo al usuario. Aplicando el conocimiento que aprendimos en los posts anteriores, terminarías con un comando como este:

{% highlight c# %}
    private RelayCommand _mostrarDialogoComando;
    public RelayCommand MostrarDialogoComando
    {
        get
        {
            if (_mostrarDialogoComando == null)
            {
                _mostrarDialogoComando = new RelayCommand(async () =>
                {
                    MessageDialog dialogo = new MessageDialog("Hola Mundo");
                    await dialogo.ShowAsync();
                });
            }
            return _mostrarDialogoComando;
        }
    }
{% endhighlight %}

El comando va a mostrar un diálogo usando una API específica de la UWP, que es la clase `MessageDialog`. Ahora digamos que, tu cliente te pide portar esta increíble aplicación a otra plataforma, como Android o iOS, usando Xamarin, la tecnología multiplataforma que permite crear aplicaciones para los principales sistemas operativos móviles usando C# y el framework .NET. En este escenario, tu ViewModel tiene un problema: no lo puedes reusar tal como está para usarlo en Android o iOS, porque esos sistemas usan una API diferente para mostrar un cuadro de diálogo. Mover código específico de plataforma a un servicio es la mejor forma de solucionar este problema: la meta es cambiar nuestro ViewModel de tal forma que solo describa la operación a hacer (mostrar un diálogo) sin implementar nada realmente.

Empecemos creando una interfaz, que describe las operaciones a llevar a cabo:

{% highlight c# %}
    public interface IDialogoServicio
    {
        Task MostrarDialogoAsync(string mensaje);
    }
{% endhighlight %}

Esta interfaz será implementada por una clase real, que será diferente en cada plataforma. Por ejemplo, la implementación para la Plataforma Universal de Windows lucirá como sigue:

{% highlight c# %}
    public class DialogoServicio : IDialogoServicio
    {
        public async Task MostrarDialogoAsync(string mensaje)
        {
            MessageDialog dialogo = new MessageDialog("Hola mundo");
            await dialogo.ShowAsync();
        }
    }
{% endhighlight %}

En Xamarin.Android, en cambio, necesitaremos usar la clase `AlertDialog`, que es específica para Android:

{% highlight c# %}
```
public async Task MostrarDialogoAsync(string mensaje)
{
    AlertDialog.Builder alerta = new AlertDialog.Builder(Application.Context);
    alerta.SetTitle(mensaje);

    alerta.SetPositiveButton("OK", (senderAlert, args) =>
    {

    });

    alerta.SetNegativeButton("Cancelar", (senderAlert, args) =>
    {

    });

    alerta.Show();
    await Task.Yield();
}
```
{% endhighlight %}

Ahora que hemos movido las APIs específicas de plataforma en un servicio, podemos usar la inyección de dependencias (que ya hemos descrito antes), para usar, en el ViewModel, la interfaz en lugar de la clase real. De esta manera, nuestro comando solamente describirá la operación que debe llevar a cabo, delegando a la clase `DialogService` a que efectivamente ejecute el código. Con este nuevo enfoque, el ViewModel añadirá una dependencia a la clase `IDialogoServicio` en el constructor, como en el siguiente ejemplo:

{% highlight c# %}
    public class MainViewModel : ViewModelBase
    {
        private readonly IDialogoServicio _dialogoServicio;
        public MainViewModel(IDialogoServicio dialogoServicio)
        {
            _dialogoServicio = dialogoServicio;
        }
    }
{% endhighlight %}

Entonces cambiamos nuestro comando de la siguiente manera:

{% highlight c# %}
    private RelayCommand _mostrarDialogoComando;
    public RelayCommand MostrarDialogoComando
    {
        get
        {
            if (_mostrarDialogoComando == null)
            {
                _mostrarDialogoComando = new RelayCommand(async () =>
                {
                    await _dialogoServicio.MostrarDialogoAsync("Hola Mundo");
                });
            }
            return _mostrarDialogoComando;
        }
    }
{% endhighlight %}

Al usar la inyección de dependencias, la aplicación Android va a registrar en el container la implementación del `DialogoServicio` que use las APIs de Android; y viceversa, la aplicación UWP va a registrar en cambio, la implementación que use las APIs de UWP. Ahora nuestro ViewModel puede compartirse así como está entre las dos versiones de la aplicación, sin hacerle ningún cambio. Podemos mover el ViewModel, por ejemplo, en una PCL (Portable Class Library), que puede compartirse entre las versiones de la aplicación de Windows, Xamarin Android, Xamarin iOS, WPF, etc.

Para ayudar a los desarrolladores a mover código de plataforma específico a los servicios, hay varias librerías que ofrecen un conjunto de servicios para ser usados en las aplicaciones. Uno de los que mejor se lleva con MVVM Light es Cimbalino Toolkit, que es específico para el mundo Windows. Además de varias clases de *converters*, *behaviors*, y *helpers*, incluye un amplio conjunto de servicios para manejar almacenamiento, configuraciones, acceso a la red, etc. Todos los servicios son proveídos con sus propias interfaces, así será fácil usarlos con el enfoque de la inyección de dependencias.

 Si quieres aprender más acerca de cómo reusar tus ViewModels en diferentes plataformas, te recomendamos leer [este](https://msdn.microsoft.com/en-us/magazine/mt147239.aspx) artículo escrito por Laurent Bugnion, el creador de MVVM Light. El tutorial de ayudará a aprender cómo puedes reusar tus conocimientos sobre bindings también en plataformas como Android e iOS, que no soportan bindings como tal.

## Implementar la interfaz INotifyPropertyChanged de una forma fácil
En el segundo post de esta serie aprendimos que, para aplicar apropiadamente la interfaz *INotifyPropertyChanged*, necesitabamos cambiar la forma como definirmos las propiedades en el ViewModel. No podemos usar la síntaxis estándar de *get / set*, sino que en el *setter* necesitábamos llamar a un método que despache la notificación al canal del binding de que el valor ha cambiado. Este efoque hace que el código sea más "ruidoso", dado que la simple definición de una propiedad requiere de varias líneas de código.

Por favor démosle la bienvenida a **Fody**, una librería que puede cambiar el código que ya hemos escrito en tiempo de construcción. Fody ofrece varios añadidos y uno de ellos es *Fody.PropertyChanged*. Su propósito es convertir automáticamente cada propiedad estándar en una propiedad que, bajo el capó, implementa la interfaz INotifyPropertyChanged. Todo lo que tienes que hacer es decorar la clase (como un ViewModel) con el atributo `[ImplementPropertyChanged]`.

Por ejemplo, el siguiente código:

{% highlight c# %}
    [ImplementPropertyChanged]
    public class MainViewModel : ViewModelBase
    {
	    public string Mensaje { get; set; }
	}
{% endhighlight %}

Queda convertido en esto:

{% highlight c# %}
    public class MainViewModel : ViewModelBase, INotifyPropertyChanged
    {
	    private string _mensaje;
	    public string Mensaje
	    {
		    get { return _mensaje; }
		    set
		    {
			    _mensaje = value;
			    OnPropertyChanged(Mensaje)
			}
		}
	}
{% endhighlight %}

De esta forma, podemos simplificar el código que necesitamos escribir en nuestro ViewModel y hacerlo menos verboso. Para usar este atributo especial, necesitamos:

1. Instalar el paquete llamado *Fody.PropertyChanged* desde NuGet
2. Para funcionar correctamente, Fody requiere un archivo XML especial en la raíz del proyecto, que describa cuál es el añadido a aplicar en tiempo de compilación. El nombre de este archivo es *FodyWeavers.xml* y el contenido debe ser algo como esto:

{% highlight html %}
    <?xml version="1.0" encoding="utf-8" ?>
    <Weavers>
        <PropertyChanged />
    </Weavers>
{% endhighlight %}

Y eso es todo.

## MVVM y Template10
Template10 es una nueva plantilla, específica para el desarrollo de aplicaciones UWP, que ha sido creada para hacer la vida del desarrollador más fácil, al proveer una forma limpia y sencilla de manejar la inicialización de una aplicación, nuevos controles, MVVM helpers, etc. Template10 es un gran punto de partida también para aplicaciones MVVM, dado que ofrece un conjunto de clases que te ayudarán a resolver algunos de los desafíos específicos de plataforma que puedan surgir durante el desarrollo, como manejar la navegación or el ciclo de vida de la página. No vamos a indagar mucho en este post, pues próximante haremos otro post hablando de Template10.

Happy coding!

**El patrón MVVM: Series**
1. [Introducción](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/)
2. [La práctica](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/)
3. [Inyección de dependencias](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/)
4. [Escenarios avanzados](https://theshallowbay.github.io/tutoriales/2017/08/08/mvvm-escenarios-avanzados/)
5. [Servicios, helpers y plantillas](https://theshallowbay.github.io/2017/08/09/mvvm-servicios-helpers-plantillas/)
6. [Datos en tiempo de diseño](https://theshallowbay.github.io/tutoriales/2017/08/09/datos-tiempo-diseno/)



