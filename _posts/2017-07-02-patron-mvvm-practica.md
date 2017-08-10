---
layout: post
date: 2017-07-04 11:54:00
summary: Continuemos en nuestro viaje para aprender el patrón MVVM, aplicado al desarrollo de aplicaciones universales UWP. Después de haber aprendido los conceptos básicos, es tiempo de empezar a escribir algo de código. Usaremos MVVM Light como toolkit para ayudarnos a implementar el patrón, como es el más flexible y simple de usar, será más fácil de entender y aplicar los conceptos básicos que hemos aprendido.
title: El patrón MVVM - La práctica
categories: tutoriales
published: true
---

Continuemos en nuestro viaje para aprender el patrón MVVM, aplicado al desarrollo de aplicaciones universales UWP. Después de haber aprendido los [conceptos básicos](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/), es tiempo de empezar a escribir algo de código. Usaremos MVVM Light como toolkit para ayudarnos a implementar el patrón, como es el más flexible y simple de usar, será más fácil de entender y aplicar los conceptos básicos que hemos aprendido.

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://blog.qmatteoq.com/the-mvvm-pattern-the-practice/). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

## El proyecto
La meta del patrón MVVM es ayudar a los desarrolladores a organizar su código de una mejor manera. Como tal, el **primer paso** es definir la estructura del proyecto de una forma que, también desde un punto de vista lógico, se pueda seguir los principios del patrón. Consecuentemente, usualmente la primera cosa que se hace es crear un conjunto de carpetas donde ubicar nuestras clases y nuestros archivos, como:

- **Models**, donde se almacenan todas nuestras entidades básicas
- **ViewModels**, donde se almacenan todas las clases que conectarán al Model con la Vista
- **Views**, donde se almacenan todas las Vistas, que son las páginas XAML en el caso de aplicaciones universales UWP

![](https://i.imgur.com/RPwZ1jr.png)

En un proyecto típico MVVM, sin embargo, terminarás teniendo más carpetas: una para los *assets*, una para los servicios, una para las *helper classes*, etc.

El **segundo paso** es instalar en el proyecto la librería que hemos escogido para ayudarnos a implementar el patrón. En nuestro caso, escogimos MVVM Light, así que podemos usar NuGet para instalarlo. Encontraremos dos diferentes versiones:

- El [paquete completo](http://www.nuget.org/packages/MvvmLight/) que, además de las librerías, agrega documentación y una serie de clases por defecto (como un ViewModel, un ViewModelLocator, etc.)
- [Sólo las librerías](http://www.nuget.org/packages/MvvmLightLibs/), que agregarán solamente los DLL requeridos.

Por el momento, nuestra sugerencia es usar la segunda opción: de esta forma, haremos todo desde el principio, dándonos la oportunidad de aprender mejor los conceptos básicos. En el futuro, eres libre de usar la primera opción si quieres ahorrar tiempo.

Para hacer esto, en Visual Studio vamos al menú **Herramientas -> Administrador de paquetes NuGet -> Administrar paquetes NuGet para la solución...** luego en la pestaña *Examinar* escribimos el nombre de la librería, en este caso, *MVVM Light*:

![](https://i.imgur.com/ypGVXQP.png)

![](https://i.imgur.com/t3WbPOz.png)

Una vez encontrado el paquete que queremos instalar, en el pequeño cuadro de diálogo que se abre al lado derecho seleccionamos nuestro proyecto y hacemos click en el botón instalar (claramente, necesitamos tener conexión a internet para descargar el paquete)

![](https://i.imgur.com/YvsYjJN.png)

Una vez descargado, revisamos la ventana de cambios y si queremos, podemos marcar la opción *No volver a mostrar* para que los cambios sean aceptados automáticamente, solo queda hacer click sobre el botón *Aceptar*.

![](https://i.imgur.com/CdleK5q.png)

Como proyecto de ejemplo, vamos a crear una aplicación universal UWP que es probable que seas capaz de desarrollar sin usar code-behind: una apliación de *Hola Mundo* con un TextBox, donde insertar tu nombre, y un botón que, cuando se presione, muestre un mensaje de hola seguido de tu nombre.

## Enlazando la Vista y el ViewModel
El primer paso es identificar los tres componentes de nuestra aplicación: Modelo, Vista y ViewModel.

- El **Modelo** no es necesario en esta aplicación, porque no necesitaremos manipular ninguna entidad
- La **Vista** estará hecha de una única página, que mostrará al usuario el TextBox donde insertar su nombre y el Button para mostrar el mensaje.
- El **ViewModel** será una clase, que manejará la interacción en la Vista: recuperará el nombre escrito por el usuario, compondrá el mensaje de hola y mostrará el mensaje en la página

Empecemos agregando los componentes: creemos una carpeta de **Views** y agreguemos, dentro de ella, la única página de nuestra aplicación. Como comportamiento por defecto, cuando se crea una nueva aplicación universal UWP, la plantilla creará una página por defecto, llamada **MainPage.xaml**. Esa página puede moverse a la carpeta de *Views* o eliminarse y luego crear una nueva en el *Explorador de Soluciones* haciendo click derecho en la carpeta y seleccionando **Agregar -> Nuevo elemento -> Página en blanco**.

**NOTA IMPORTANTE:** Si eliminamos la Vista que crea Visual Studio por defecto, y luego ponemos otra en la carpeta de *Views* le debemos específicar cuál va a ser la página que será el punto de partida cuando se incie la aplicación. Para hacer esto, ubicamos en el *Explorador de soluciones* la clase *App.xaml.cs* (sí, hace parte del code-behind de la página App.xaml, pero estrictamente hablando  App.xaml no es una vista en sí misma), luego al inicio de la clase debemos escribir dónde tenemos nuestra Vista a usar:

{% highlight c# %}
    using HolaMundo.Views
{% endhighlight %}

Después, aproximadamente sobre la línea 70, escribimos el nombre de nuestra página como argumento del método `Navigate` como ejemplo:

{% highlight c# %}
    rootFrame.Navigate(typeof(MainPage), e.Arguments);
{% endhighlight %}

Ahora creemos el **ViewModel** que se conectará a esta página. Como explicamos anteriormente, el ViewModel es sólo una clase simple: crea la carpeta *ViewModels*, click derecho sobre ella y escoge **Agregar -> Nuevo elemento -> Clase.**

![](https://i.imgur.com/5sbqP2K.png)

![](https://i.imgur.com/ZohTKYG.png)

Ahora necesitamos conectar la Vista con el ViewModel, usando la propiedad del DataContext. Como aprendimos anteriormente, la clase ViewModel se configura como el DataContext de la página, de esta forma, todos los controles en la página XAML podrán acceder a todas las propiedades y comandos declarados en el ViewModel. Hay varias maneras de lograr esto, veamos.

### Declarar el ViewModel como recurso

Digamos que hemos creado un ViewModel llamado MainViewModel. Podemos declararlo como un recurso global en el archivo App.xaml de esta forma:

{% highlight html %}
    <Application x:Class="HolaMundo.App"
				 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
				 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
				 xmlns:local="using:HolaMundo"
				 xmlns:viewModel="using:HolaMundo.ViewModels">
				 
		<Appication.Resources>
			<viewModel:MainViewModel x:Key="MainViewModel" />
		</Aplication.Resources>
		
	</Application>
{% endhighlight %}

El primer paso es declarar, como un atributo de la clase Aplicación, el espacio de nombres (*namespace*) que contiene al ViewModel (en este ejemplo es *HolaMudo.ViewModels*) en la línea `xmlns:viewModel="using:HolaMundo.ViewModels"`. Luego en la colección Resources de la clase Application, se declara un nuevo recurso cuyo tipo es *ManViewModel* y se asocia a una llave (*key*) con el mismo nombre. 

En caso de que Visual Studio marque un error subrayado en azul, diciendo que *ManViewModel no existe en el espacio de nombres* se debe implementar el proyecto haciendo click derecho sobre el nombre del proyecto en el Explorador de soluciones y luego escogiendo la opción *Implementar*

![](https://i.imgur.com/dMTJ1NC.png)

Ahora se puede usar esta key y la palabra clave **StaticResource** para conectar el *DataContext* de la página al recurso, como en el siguiente ejemplo:

{% highlight html %}
    <Page x:Class="HolaMundo.Views.MainPage"
		  DataContext="{Binding Source={StaticResource MainViewModel}}"
		  mc:Ignorable="d">
		  
		  <!-- El contenido de la página va aquí -->
	
	</Page>
{% endhighlight %}

### El enfoque del ViewModelLocator

Otro enfoque usado frecuentemente es utilizar una clase llamada **ViewModelLocator**, que tiene la responsabilidad de despachar los ViewModels a las distintas página. En lugar de registrar todos los ViewModels como recursos globales de la aplicación, como en el enfoque anterior, se registra solamente el *ViewModeLocator*. Todos los ViewModels quedarán expuestos, como propiedades, por el *locator* que será administrado por la propiedad DataContext de la página.

Esta es una definición de ejemplo de una clase *ViewModelLocator*:

{% highlight c# %}
    public class ViewModelLocator
    {
	    public ViewModelLocator()
	    {
	    }
	    
	    public MainViewModel Main
	    {
		    get
		    {
			    return new MainViewModel();
			}
		}
	}
{% endhighlight %}

O, como alternativa, se puede simplificar el código usando una de las características nuevas en C# 6.0:

{% highlight c# %}
    public class ViewModelLocator
    {
	    public ViewModelLocator()
	    {
	    }
	    
	    public MainViewModel Main => new MainViewModel();
	}
{% endhighlight %}

Después de haber agregado la clase *ViewModelLocator* como un recurso global, se podrá usar para conectar la propiedad **Main** a la propiedad *DataContext* de la página, como en el siguiente ejemplo:

{% highlight html %}
    <Page x:Class="HolaMundo.Views.MainPage"
	      DataContext="{Binding Source={StaticResource Locator}, Path=Main}"
	      mc:Ignorable="d">
	      
	      <!-- El contenido de la página va aquí -->
	      
	</Page>
{% endhighlight %}

La sintaxis es muy similar a la que ya vimos en el primer enfoque, la principal diferencia es que, como la clase *ViewModelLocator* puede exponer múltiples propiedades, y se necesita especificar con el atributo **Path** cuál se quiere usar.

El enfoque con ViewModelLocator agrega una nueva clase para mantener, pero da la flexibilidad de manejar la creación del ViewModel en caso de que se necesite pasar parámetros a la clase constructor. Más adelante introduciremos el concepto de *inyección de dependencias*, y será más fácil entender las ventajas de usar el enfoque con ViewModelLocator.

## A configurar el ViewModel
No importa cuál enfoque hayas decidido utilizar en el paso anterior, ahora tenemos una Vista (la página que mostrará el formulario al usuario) conectada a un ViewModel (la clase que maneja las interacciones del usuario).

Empecemos a poblar el ViewModel y a definir las propiedades que necesitamos para alcanzar nuestra meta. Como ya aprendimos anteriormente, los ViewModels necesitan de la interfaz INotifyPropertyChanged, de otro modo, cada vez que cambiemos el valor de una propiedad en el ViewModel, la Vista no podrá detectar el cambio y el usuario no verá ningún cambio en la interfaz.

Para hacer más fácil la implementación de esta interfaz, MVVM Light ofrece una clase base de la cual podemos heredar a nuestros ViewModels, como en el siguiente ejemplo:

{% highlight c# %}
    using GalaSoft.MvvmLight;
    
    public class MainViewModel : ViewModelBase
    {
    
    }
{% endhighlight %}

Esta clase nos da acceso a un método llamado **Set()**, que podemos usar cuando definimos propiedades para despachar las notificaciones a la interfaz de usuario cuando el valor cambie. Veamos un ejemplo relacionado a nuestro escenario. Nuestro ViewModel tiene que ser capaz de usar el nombre que el usuario a escrito en un control *TextBox* en la Vista. Consecuentemente, necesitamos una propiedad en nuestro ViewModel para almacenar este valor. Así es como luce gracias al apoyo en MVVM Light:

{% highlight c# %}
    private string _nombre;
    public string Nombre
    {
	    get
	    {
		    return _nombre;
		}

		set
		{
			Set(ref _nombre, value);
		}
	}
{% endhighlight %}

Este código es muy similar al que ya vimos anteriormente cuando [introdujimos el concepto](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/#la-interfaz-inotifypropertychanged) de la interfaz *INotifyPropertyChanged*. La única diferencia es que, gracias al método *Set()*, podemos matar dos pájaros de un solo tiro: almacenamos el valor en la propiedad **Nombre** y despachamos una notificación al canal binding de que el valor ha cambiado.

Ahora que sabemos cómo crear propiedades, creamos otra para guardar el mensaje que se mostrará al usuario después de que haya presionado el botón.

{% highlight c# %}
    private string _mensaje;
    public string Mensaje
    {
	    get
	    {
		    return _mensaje;
		}
		set
		{
			Set(ref _mensaje, value);
		}
	}
{% endhighlight %}

Al final, necesitamos manejar la interacción con el usuario. Cuando él presione el *Botón* en la *Vista*, se debe mostrar el mensaje de hola. Desde un punto de vista lógico, esto significa:

1. Recuperar el valor de la propiedad **Nombre**
2. Dejar a las APIs crear y preparar el mensaje (algo como "¡Hola James!")
3. Asignar el resultado a la propiedad **Mensaje**

Anteriormente aprendimos que, en el patrón MVVM, se usan los *comandos* para manejar las interacciones del usuario con un ViewModel. Por tal razón, vamos a usar otra de las clases ofrecidas por MVVM Light, que es **RelayCommad**. Gracias a esta clase, en lugar de crear una nueva clase que implemente la interfaz *ICommand* por cada comado, podemos declarar una nueva con el siguiente código:

{% highlight c# %}
    using GalaSoft.MvvmLight.Command;
    
    private RelayCommand _diHola;
    public RelayCommand DiHola
    {
	    get
	    {
		    if (_diHola == null)
		    {
			    _diHola = new RelayCommand(() =>
			    {
				    Mensaje = $"¡Hola {Nombre}!";
				});
			}
			
			return _diHola;
		}
	}
{% endhighlight %}

Cuando se crea un nuevo objeto *RelayCommand* se le tiene que pasar, como parámetro, una ***Action***, que define el código que se quiere ejecutar cuando el comando es invocado. El ejemplo anterior declara la *Action* usando un método anónimo. Eso significa que, en lugar de definir un nuevo método en la clase con un nombre específico, se define en el mismo lugar de la definición de la propiedad, sin asignarle un nombre.

Cuando el comando es invocado, se usa la nueva característica de C# 6.0 para llevar a cabo la concatenación de cadenas y obtener el valor de la propiedad **Nombre** y agregarle el prefijo "Hola". El resultado es guardado en la propiedad **Mensaje**.

## Crear la Vista
Ahora que el ViewModel está listo, podemos ir a crear la Vista. El paso anterior debería haberte ayudado a entender uno de los grandes beneficios del patrón MVVM: hemos sido capaces de definir el ViewModel para manejar la lógica y la interacción del usuario sin escribir una sola línea de código en XAML. Con el enfoque de code-behind, eso habría sido imposible; por ejemplo, si hubierámos querido recuperar el nombre escrito en el *TextBox*, primero debíamos haber agregado el control a la página y asignado un nombre usando la propiedad *x:Name*, para haber podido acceder desde el code-behind. O si hubierámos querido definir el método a ejecutar cuando se presiona el botón, hubiera sido necesario agregar un control *Button* en la página y suscribirse al evento *Click*.

Desde un punto de vista de interfaz de usuario, el XAML que necesitamos escribir para nuestra aplicación es más o menos el mismo que hubierámos creado para una aplicación code-behind. La interfaz, de hecho, no tiene ninguna conexión con la lógica, así que la manera en que la diseñemos no va a cambiar cuando usemos el patrón MVVM. La principal diferencia es que, en una app MVVM, usaremos binding mucho más, porque es la forma en que conectamos los controles con las propiedades en la ViewModel.
Así es como luce nuestra Vista:

{% highlight html %}
    <Page x:Class="HolaMundo.Views.MainPage"
		  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
		  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
		  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
		  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
		  DataContext="{Binding Source={StaticResource Locator}, Path=Main}"
		  mc:Ignorable="d">
		  <Grid>
			  <StackPanel Margin="12, 30, 0, 0>
				  <StackPanel Orientation="Horizontal" Margin="0, 0, 0, 30">
					  <TextBox Text="{Binding Path=Nombre, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" Width="300" Margin="0, 0, 30, 0" />
					  <Button Command="{Binding Path=DiHola}" Content="Haz click aquí" />
				  </StackPanel>
				  <TextBlock Text="{Binding Path=Mensaje}" Style="{StaticResource HeaderTextBlockStyle}" />
			</StackPanel>
		</Grid>
	</Page>
{% endhighlight %}

Hemos agregado tres controles:
1. Un *TextBox*, donde el usuario puede escribir su nombre. Lo hemos conectado a la propiedad **Nombre** del ViewModel. Hay dos características que resaltar:
	- Hemos fijado el atributo **Mode** a *TwoWay*. Se requiere porque, para este escenario, no sólo necesitamos que cuando la propiedad en el ViewModel cambie el control muestre el valor actualizado, sino también lo contrario. Cuando el usuario escribe texto en el control, necesitamos guardar su valor en el ViewModel.
	- Hemos configurado el atributo **UpdateSourceTrigger** a *PropertyChanged*. De esta forma, nos aseguramos que, cada vez que el texto cambie (esto signfica cada vez que el usuario agregue o elimine un carácter en el TextBox), la propiedad *Nombre* automaticamente se actualice para almacenar el nuevo valor. Sin este atributo, el valor de la propiedad se atualizaría solamente cuando el control *TextBox* pierde el foco.
2. Un *Button*, que el usuario presionará para ver el mensaje. Lo hemos conectado al comando **DiHola** en el ViewModel.
3. Un *TextBlock*, donde el usuario verá el mensaje. Lo hemos conectado a la propiedad **Mensaje** del ViewModel.

Si hemos hecho todo correctamente y lanzamos la aplicación, deberiamos ver la app comportándose como hemos descrito al principio: después de escribir el nombre y presionar el botón, se ve el mensaje de hola.

![](https://i.imgur.com/9H3Kowg.gif)

## Mejoremos un poco la interacción del usuario
Nuestra aplicación tiene un punto débil: si el usuario presiona el botón sin haber escrito nada en el *TextBox*, solamente verá un "¡Hola!". Queremos evitar este comportamiento deshabilitando el botón si el control *TextBox* está vacío. Para lograr esto, podemos hacer uso de una de las características ofrecidas por la interfaz **ICommand**: podemos definir cuándo un comando debe ser habilitado o no. Para este escenario la clase **RelayCommand** permite un segundo parámetro durante la inicialización, que es una función que devuelve un valor booleano. Veamos un ejemplo:

{% highlight c# %}
    private RelayCommand _diHola;
    public RelayCommand DiHola;
    {
	    get
	    {
		    if (_diHola == null)
		    {
			    _diHola = new RelayCommand(() =>
			    {
				    Mensaje = $"¡Hola {Nombre}!";
				},
				() => !string.IsNullOrEmpty(Nombre));
			}
			return _diHola;
		}
	}
{% endhighlight %}

Hemos cambiado la inicialización del comando **DiHola** para agregar, como segundo parámetro, un valor booleano: específicamente, comprobamos si la propiedad **Nombre** es nula o está vacía (en este caso, devuelve *false*, si no *true*). Gracias a este cambio, ahora el *Button* conectado a este comando se deshabilitará automaticamente (también desde un punto de vista visual) si la propiedad *Nombre* está vacía. Sin embargo, hay otro problema: si intentamos ejecutar la aplicación tal como está, notaremos que el *Button* estará deshabilitado por defecto cuando la aplicación se inicie. Es el comportamiento esperado, porque cuando la aplicación se lanza el control *TextBox* está vacío por defecto. Sin embargo, si empezamos a escribir texto en el *TextBox*, el *Button* continuará estando deshabilitado.

![](https://i.imgur.com/bP5xzSw.png)

La razón de esto es que el ViewModel no puede automaticamente determinar cuando la ejecución del comando **DiHola** necesita ser evaluada de nuevo. Necesitamos hacerlo manualmente cada vez que hagamos algo que pueda hacer cambiar el valor de la función booleana; en nuestro caso, eso pasa cuando cambiamos el valor de la propiedad *Nombre*, así que necesitamos cambiar la definición de la propiedad como en el siguiente ejemplo:

{% highlight c# %}
    private sting _nombre;
    public sring Nombre
    {
	    get
	    {
		    return _nombre;
		}
		
		set
		{
			Set(ref _nombre, value);
			DiHola.RaiseCanExecuteChanged();
		}
	}
{% endhighlight %}

Además de solamente usar el método *Set()* para almacenar la propiedad y enviar la notifiación a la interfaz de usuario, invocamos el método **RaiseCanExecuteChanged()** del comando *DiHola*. De esta manera, la condición booleana será evaluada de nuevo: si la propiedad *Nombre* contiene algún texto, entonces el comando se habilitará, de otra forma, si queda vacía de nuevo, el comando se deshabilitará.

![](https://i.imgur.com/5wdyhEl.gif)

Hemos finalmente aplicado en un proyecto real los conceptos aprendidos en la primera parte de post escribiendo código en nuestra aplicación basada en el patrón MVVM. 

Happy coding!

**El patrón MVVM: Series**
1. [Introducción](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/)
2. [La práctica](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/)
3. [Inyección de dependencias](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/)
4. [Escenarios avanzados](https://theshallowbay.github.io/tutoriales/2017/08/08/patron-mvvm-escenarios-avanzados/)
5. [Servicios, helpers y plantillas](https://theshallowbay.github.io/2017/08/09/patron-mvvm-servicios-helpers-plantillas/)
6. [Datos en tiempo de diseño](https://theshallowbay.github.io/tutoriales/2017/08/09/patron-mvvm-datos-tiempo-diseno/)
