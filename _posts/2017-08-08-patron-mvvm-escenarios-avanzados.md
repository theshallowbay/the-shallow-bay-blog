---
layout: post
date: 2017-08-08 11:00:00
summary: Continuemos nuestro viaje para aprender cómo usar el patrón MVVM y cómo aplicarlo para desarrollar aplicaciones universales UWP. En esta publicación exploraremos algunos escenarios avanzados que son frecuentes de encontrarse cuando estés desarrollando un proyecto real cómo manejar eventos secundarios, cómo intercambiar un mensaje y cómo usar un dispatcher.
title: El patrón MVVM - Escenarios avanzados
categories: tutoriales
published: true
---

Continuemos nuestro viaje para aprender cómo usar el patrón MVVM y cómo aplicarlo para desarrollar aplicaciones universales UWP. En esta publicación exploraremos algunos escenarios avanzados que son frecuentes de encontrarse cuando estés desarrollando un proyecto real: cómo manejar eventos secundarios, cómo intercambiar un mensaje y cómo usar un *dispatcher*.

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://blog.qmatteoq.com/introduction-to-mvvm-advanced-scenarios/). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

## Manejando eventos adicionales con comandos
En la [publicación anterior](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/) aprendimos como todos los controles XAML que permiten interacción con el usuario (como un botón) ofrecen una propiedad llamada **Command**, que podemos usar para conectar un evento a un método sin tener que usar un manipulador de eventos. Sin embargo, la propiedad Command puede usarse para manipular solamente la interacción principal del evento expuesto por el control. Por ejemplo, si estás trabajando con un control Button, puedes usar un comando para manejar el evento *Click*. Pero, hay varios escenarios en donde puede que necesites mandejar eventos secundarios. Por ejemplo, el control ListView expone un evento llamado *SelectionChanged*, que es disparado cuando el usuario selecciona un elemento de la lista. O la clase *Page* expone un evento llamado *Loaded* que se dispara cuando la página es cargada.

Para manejar esas situaciones podemos utilizar el ***Behaviors SDK***, que es una librería (o biblioteca, para los más puristas del español) de Microsoft (de hecho recientemente la hicieron [open source](https://github.com/Microsoft/XamlBehaviors)) que contiene un conjunto de *comportamientos* - de ahora en adelante behaviors- listos para ser usados en aplicaciones. Los behaviors son una de las características más interesantes de XAML, dado que permiten encapsular lógica, que típicamente debería manejarse en el code-behind, en componentes que pueden reusarse directamente en XAML. Los behaviors son ampliamente usados en las aplicaciones que implementan MVVM, porque ayudan a reducir el código que se necesita escribir en las clases del code-behind.

Un behavior está basado en:
1. Un *disparador*, o **Trigger**, que es la acción que causará que se ejecute el behavior
2. Una *acción*, o **Action**, que es la acción que se llevará a cabo cuando el behavior se ejecute

El Behavior SDK incluye un conjunto de triggers y de actions que son específicos para manejar nuestro escenario: conectar eventos secundarios a comandos definidos en el ViewModel.

Veamos un ejemplo real. El primer paso es agregar el Behaviors SDK a nuestro proyecto. Si estás trabajando en un proyecto Windows Phone/WPF/Silverligth, el SDK ya viene incluído en Visual Studio y está disponible desde el menú *Extensiones -> Agregar referencia*.  Por otro lado, si estpas trabajando en una aplicación UWP, hay una nueva versión del SDK disponible como paquete de NuGet, como cualquier otra librería, que puede actualizarse independientemente de la versión de Visual Studio o del SDK de Windows 10. Para instalarlo, basta con buscar 'behaviors sdk' en el administrador de paquetes de NuGet, [tal como ya explicamos aquí](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/#el-proyecto). El paquete identificado por el id. *Microsoft.Xaml.Behaviors.Uwp.Managed* es para aplicaciones escritas en C# / VB.NET y el otro paquete con el id. *Microsoft.Xaml.Behaviors.Uwp.Native* es para aplicaciones escritas en C++.

El siguiente paso es declarar, en la página XAML, los nombres de espacios (*namespaces*) del SDK, que se requieren para usar los behaviors: `Microsoft.Xaml.Interactivity` y `Microsoft.Xaml.Interactions.Code`, como en el siguiente ejemplo:

{% highlight html %}
    <Page
        x:Class="GeoLocate.MainPage"
        xmlns:core="using:Microsoft.Xaml.Interactions.Core"
        xmlns:interactivity="using:Microsoft.Xaml.Interactivity"
        mc:Ignorable="d">
    
    </Page>
{% endhighlight %}

Gracias a esos namespaces, podrás usar las siguientes clases:

1. **EventTriggerBehavior**, que es el behavior que podemos usar para conectar un trigger a cualquier evento expuesto por un control
2. **InvokeCommandAction**, que es la action que podemos usar para conectar un comando definido en el ViewModel con el manejador de eventos del trigger.

Así es como se aplican para manejar la selección de elementos de una lista:

{% highlight html %}
```
    <ListView ItemsSource="{Binding Path=Noticias}"
                      SelectedItem="{Binding Path=ElementoSeleccionado, Mode=TwoWay}">
                <interactivity:Interaction.Behaviors>
                    <core:EventTriggerBehavior EventName="SelectionChanged">
                        <core:InvokeCommandAction Command="{Binding Path=ComandoElementoSeleccionado}" />
                    </core:EventTriggerBehavior>
                </interactivity:Interaction.Behaviors>
    </ListView>
```
{% endhighlight %}

El behavior es declarado como su fuese una propiedad compleja de un control y, como tal, es incluído entre el inicio y el final de la etiqueta del control mismo (en este caso, entre  `<ListView>` y `</ListView>`. La declaración del behavior es incluída dentro de una colección llamada *Interaction.Behaviors*, que hace parte del namespace *Microsoft.Xaml.Interactivity*. Es una colección dado que puedes aplicar más de un behavior al mismo control. En este caso, estamos agregando el `EventTriggerBehavior` mencionado anteriormente que requiere, mediante el uso de la propiedad `EventName` el nombre del evento del control que queremos manejar. En este caso, queremos manejar la selección de un elemento de la lista, entonces enlazamos esta propiedad al evento llamado `SelectionChanged`.

Ahora que el behavior está enlazado al evento, podemos declarar qué action queremos ejecutar cuando el evento sea disparado. Podemos hacerlo con la `InvokeCommandActionClass`, que expone una propiedad *Command* que podemos enlazar, usando binding, a una propiedad *ICommand* en el ViewModel.

La tarea está ahora terminada: cuando el usuario seleccione un elemento de la lista, el comando llamado `ItemSelectedCommand` será invocado. Desde un punto de vista del ViewModel, no hay ninguna diferencia entre un comando estándar y un comando conectado a un behavior, como se puede ver en el siguiente ejemplo:

{% highlight c# %}
    private RelayCommand _comandoElementoSeleccionado;
    public RelayCommand ComandoElementoSeleccionado
    {
        get
        {
            if (_comandoElementoSeleccionado == null)
            {
                _comandoElementoSeleccionado = new RelayCommand(() =>
                {
                    debug.WriteLine(ElementoSeleccionado.Titulo);
                });
            }
    
            return _comandoElementoSeleccionado;
    
        }
    }
{% endhighlight %}

Este comando se encarga de mostrar, en una ventana de salida de Visual Studio (usando el método *Debug.WriteLine()*) el título del elemento seleccionado. **ElementoSeleccionado** es otra propiedad del ViewModel que es conectada, usando binding, a la propiedad **SelectedItem** del control *ListView*. De esta forma, la propiedad siempre almacenará una referencia al elemento seleccionado por el usuario en la lista.

## Mensajes
Otro requisito común cuando se está desarrollando una aplicación compleja es encontrar una forma de manejar la comunicación entre dos clases que no tienen nada en común, como dos ViewModels o un ViewModel con una clase code-behind. Digamos que, después de pase algo en un ViewModel, quieres disparar una animación en la View: en este caso, el código que hará esto será almacenado en una clase code-behind, dado que es código que está relacionado con la interfaz de usuario.

En esos escenarios, la fortaleza del patrón MVVM (que es una clara separación entre las capas) también puede convertirse en una debilidad: ¿cómo podemos manejar la comunicación entre la View y el ViewModel dado que no tienen nada en común, excepto por el hecho de la segunda es el DataContext de la primera? Esas situaciones pueden solucionarse usando mensajes, que son paquetes que una clase centralizada puede despachar a las varias clases de la aplicación. La fortaleza más importante de esos paquetes es que están completamente desconectados: no hay una relación entre el emisor y el receptor. Al usar este enfoque:

1. El emisor (un ViewModel o una View) envía un mensaje, especificando cuál es su tipo
2. El receptor (otro ViewModel o View) se suscribe al mensaje recibido que pertenece a un tipo específico.

Al final, un emisor puede enviar un mensaje sin conocer de antemano quién va a recibirlo. Y viceversa, el receptor puede recibir mensajes sin conocer quién es el emisor. Cada toolkit y framework MVVM típicamente ofrece una manera de manejar mensajes. MVVM Light no es la excepción, vamos a usar la clase `Messenger` para implementar el ejemplo que hemos descrito previamente: empezar una animación definida en el code-behind de un ViewModel.

### El mensaje
El primer paso es crear el mensaje que queremos que el emisor envíe de una clase a otra. Un mensaje es solo una clase simple: creemos una clase llamada *Mensajes* (es algo opcional, pero se hace para mantener una estrutura del proyecto limpia) y luego creemos una nueva clase dentro de esa carpeta. Así es como nuestro mensaje luce:

{% highlight c# %}
    public class EmpezarAnimacionMensaje
    {

    }
{% endhighlight %}

Como puedes ver, es solo una clase. En nuestro caso está vacía, dado que solamente necesitamos disparar una action. También puede tener una o más propiedades en caso de que, además de disparar una action, necesites también enviar datos de una clase a otra.

### El emisor
Ahora veamos cómo nuestro ViewModel puede enviar el mensaje que acabamos de definir. Podemos hacerlo usando la clase `Messenger`, incluída en el namespace *GalaSoft.MvvmLight.Messaging*. En nuestro ejemplo, asumimos que la animación se disparará cuando el usuario presione un botón. Consecuentemente, usamos la clase *Messenger* dentro de un comando, como en el siguiente ejemplo:

{% highlight c# %}
        private RelayCommand _empezarAnimacionComando;
        public RelayCommand EmpezarAnimacionComando
        {
            get
            {
                if (_empezarAnimacionComando == null)
                {
                    _empezarAnimacionComando = new RelayCommand(() =>
                    {
                        Messenger.Default.Send<EmpezarAnimacionMensaje>(new EmpezarAnimacionMensaje());
                    });
                }

                return _empezarAnimacionComando;
            }
        }
{% endhighlight %}

Enviar un mensaje es fácil. Usamos la propiedad *Default* de la clase *Messenger* para obtener acceso a la instancia estática del emisor. ¿Por qué estática? Porque, para funcionar apropiadamente, necesita ser la misma instancia para toda la aplicación, de otra forma no podrá despachar y recibir mensajes que vengan de diferentes clases. Para enviar un mensaje usamos el método *Send<T>* donde `T` es el tipo de mensaje que queremos enviar. Como parámetro, necesitamos pasarle una nueva instancia de la clase que hemos previamente creado para definir un mensaje: en nuestro ejemplo, es la que se llama `EmpezarAnimacionMensaje`.

Ahora el mensaje ha sido enviado y está listo para ser recibido por otra clase.

### El receptor
El primer paso, antes de hablar acerca de cómo recibir un mensaje, es definir en la página XAML la animación que queremos disparar cuando el botón sea presionado, al usar la clase `Storyboard`: 

{% highlight c# %}
    <Page
        x:Class="GeoLocate.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:GeoLocate"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:core="using:Microsoft.Xaml.Interactions.Core"
        xmlns:interactivity="using:Microsoft.Xaml.Interactivity"
        mc:Ignorable="d">
    
        <Page.Resources>
            <Storyboard x:Name="RectanguloAnimacion">
                <DoubleAnimation Storyboard.TargetName="RectangleTranslate"
                                 Storyboard.TargetProperty="X"
                                 From="0"
                                 To="200"
                                 Duration="00:00:03"/>
            </Storyboard>
        </Page.Resources>
    
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <Rectangle Width="100" Height="100" Fill="BlueViolet">
                <Rectangle.RenderTransform>
                    <TranslateTransform x:Name="RectangleTranslate" />
                </Rectangle.RenderTransform>
            </Rectangle>
        </Grid>
    
        <Page.BottomAppBar>
            <CommandBar>
                <CommandBar.PrimaryCommands>
                    <AppBarButton Label="Reproducir" Icon="Play" Command="{Binding Path=EmpezarAnimacionComando}" />
                </CommandBar.PrimaryCommands>
            </CommandBar>
        </Page.BottomAppBar>
        
    </Page>
{% endhighlight %}

Hemos incluído un control `Rectangle` y se le ha aplicado un `TranslateTransform`. Es una de las transformaciones incluídas en XAML, que podemos usar para mover un control en la página simplemente cambiando sus coordenadas en los ejes X, Y. La animación que vamos a crear se aplicará a esta transformación y cambiará las coordenadas del control.

La animación se define usando un `Storyboard` como recurso de la página: su tipo es `DoubleAnimation` porque va a cambiar una propiedad (la coordenada X del *TranslateTransform*) que está representada por un número. La animación va a mover el rectángulo desde la coordenada 0 (la propiedad `From`) hasta la coordenada 200 (la propiedad `To`) en X segundos (la propiedad `Duration`). Para empezar esta animación, necesitamos escribir algo de código en el code-behind: de hecho, necesitamos llamar al método `Begin()`del control `Storyboard` y, como es un recurso de página, no podemos acceder directamente desde nuestro ViewModel.

Y aquí viene nuestro problema: la animación solo puede iniciarse en el code-behind, pero el evento que la dispara está contenido en un comando en el ViewModel. Gracias a nuestro mensaje, podemos fácilmente resolverlo: es suficiente con registrar la clase del code-behind como receptor del objeto `EmpezarAnimacionMensaje` que hemos enviado desde el ViewModel. Para hacer esto, usamos de nuevo la clase *Messenger* y su instancia *Default*:

En el constructor de la página, usamos, esta vez, el método `Register<T>()` para convertir la clase del code-behind en un receptor. También en este caso, como `T` especificamos el tipo de mensaje que queremos recibir; todos los otros tipos de mensajes serán ignorados. Además, el método requiere dos parámetros:

1. El primero es una referencia a la clase que va a manejar el mensaje. En la mayoría de los casos, es la misma clase en donde estamos escribiendo el código, por lo que simplemente usamos la palabra clave `this`
2. Una *Action*, que define el código a ejecutar cuando el mensaje sea recibido. Como puedes ver en el ejemplo (definida como un método anónimo) también obtenemos una referencia al objeto mensaje, así podemos acceder fácilmente a sus propiedades si hubiéramos definido una o más para almacenar datos. Sin embargo, este no es nuestro caso: solo invocamos al método `Begin()` de nuestro `Storyboard`

Nuestro trabajo está listo: ahora si lanzamos la aplicación y presionamos el botón, el ViewModel emitirá un paquete `EmpezarAnimacionMensaje`. 

![](https://i.imgur.com/F8R90vQ.gif)

Sin embargo, como sólo la clase del code-behind de nuestra página principal está suscrita al paquete, será la única que va a recibirlo y manejarlo.

**¡Ten cuidado!**

Cuando trabajas con mensajes, es importante recordar que que tal vez hayas configurado tu aplicación para guardar algunas páginas en caché. Esto significa que si hemos configurado un ViewModel o una clase code-behind para recibir uno o más mensajes, esas clases pueden tal vez recibir los mensajes incluso si no están visibles en el momento. Esto puede llevar a algunos problemas de concurrencia: el mensaje que hemos enviado puede ser recibido por una diferente clase de la que estabamos esperando.

Por esta razón, la clase *Messenger* ofrece un método llamado `Unsubscribe<T>()` para detener la recepción de mensajes cuyo tipo sea `T`. Típicamente, cuando necesites interceptar mensajes en una clase code-behind, necesitarás recordar llamar al método mencionado en el evento `OnNavigatedFrom()`, así, cuando el usuario salga de la página dejará de recibir mensajes.

{% highlight c# %}
    protected override void OnNavigatedFrom(NavigationEventArgs e)
    {
        Messenger.Default.Unregister<EmpezarAnimacionMensaje>(this);
    }
{% endhighlight %}

Por la misma razón, también es mejor mover el método `Register<T>()` del constructor al método `OnNavigatedTo()` de la página, para asegurarse que cuando el usuario navegue de nuevo a la página, ésta podrá de nuevo recibir mensajes.

{% highlight c# %}
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        Messenger.Default.Register<EmpezarAnimacionMensaje>(this, message =>
        {
            RectanguloAnimacion.Begin();
        });
    }
{% endhighlight %}

## Trabajando con threads (hilos)
La interfaz de usuario de una aplicación UWP es manejada por un único hilo, o *thread*, llamado **UI Thread**.  Es algo crítico dejar este hilo tan libre como nos sea posible; si empezamos a ejecutar muchas operaciones sobre él, la interfaz de usuario puede volverse *unresponsive* (que no responda) y lenta de usar. Sin embargo, en algún punto, podemos necesitar acceder a ése hilo, por ejemplo, porque necesitamos mostrar el resultado de una operación en un control ubicado en una página. Por esta razón, la mayoría de las APIs de la Universal Windows Platform son implementadas usando el patrón ***async / await***, el cual se asegura que las operaciones demoradas no se ejecuten en en UI Thread, dejándolo libre para procesar la interfaz de usuario y las interacciones del usuario. Al mismo tiempo, el resultado se devuelve automáticamente en el UI Thread, de forma que esté inmediatamente listo para ser usado por cualquier control en la página.

Sin embargo, hay algunos escenario donde este despacho no se hace automáticamente. Tomemos que, por ejemplo, la API *Geolocator*, que es proveída por la UWP para detectar la ubiación del usuario. Si necesitamos continuamente rastrear la ubicación del usuario, la clase ofrece un evento llamado `PositionChanged`, a la cual se puede uno suscribir con un manejador de eventos para obtener toda la información acerca de la ubiación detectada (como las coordenadas). Construyamos una aplicación de ejemplo que rastree la ubicación del usuario y muestre las coordenadas en una página. La estructura del proyecto es la misma que ya hemos usado en los otros ejemplo: tendremos una *View*  (con un Button y un TextBlock) conectada a un *ViewModel*, con un comando (para empezar la detección) y una propiedad `string` (para mostrar las coordenadas).

**Por favor, nótese:** el propósito de este ejemplo solamente es mostrarte un escenario donde manejar el UI Thread de la manera apropiada es muy importante. En un proyecto real, usar una API específica de una plataforma (en este caso, la `Geolocator`) en un ViewModel no es la mejor forma de hacerlo. Aprenderemos de esto en el siguiente post.

Así es como luce nuestro ViewModel:

{% highlight c# %}
```
public class MainViewModel : ViewModelBase
{
    private readonly Geolocator _geolocator;

    public MainViewModel()
    {
        _geolocator = new Geolocator();
        _geolocator.DesiredAccuracy = PositionAccuracy.High;
        _geolocator.MovementThreshold = 50;
    }

    private string _coordenadas;
    public string Coordenadas
    {
        get { return _coordenadas; }
        set { Set(ref _coordenadas, value); }
    }

    private RelayCommand _iniciarGeolocalizacionComando;
    public RelayCommand IniciarGeolocalizacionComando
    {
        get
        {
            if (_iniciarGeolocalizacionComando == null)
            {
                _iniciarGeolocalizacionComando = new RelayCommand(() =>
                {
                    _geolocator.PositionChanged += _geolocator_PositionChanged;
                });
            }

            return _iniciarGeolocalizacionComando;
        }
    }

    private void _geolocator_PositionChanged(Geolocator sender, PositionChangedEventArgs args)
    {
        Coordenadas = $"{args.Position.Coordinate.Point.Position.Latitude}, {args.Position.Coordinate.Point.Position.Longitude}";
    }
}
```
{% endhighlight %}

Cuando el ViewModel es creado, inicializamos la clase *Geolocator* requerida para interactuar con los servicios de localización del teléfono. Entonces, en el *IniciarGeolocalizacionComando*, definimos una action que se suscribe al evento `PositionChanged` del *Geolocator*: desde ahora, el dispositivo empezará a detectar la ubicación del usuario y disparará el evento cada vez que la ubicación cambie. En el manejador del evento definimos la propiedad *Coordenadas* con un `string`, que es la combinación de las propiedades `Latitude` y `Longitude` devueltas por el manejador.

La View es muy sencilla: es un Button (conectado a la propiedad `IniciarGeolocalizacionComando`) y un TextBlock (conectado a la propiedad `Coordenadas`).

{% highlight html %}
    <Page
        x:Class="TrackMe.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:TrackMe"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        DataContext="{Binding Source={StaticResource Locator}, Path=Main}"
        mc:Ignorable="d">
    
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                <Button Content="Iniciar geolocalización" Command="{Binding IniciarGeolocalizacionComando}" />
                <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center"
                           Text="{Binding Path=Coordenadas}" />
            </StackPanel>
        </Grid>
    </Page>
{% endhighlight %}

Si ejecutamos la aplicación y presionamos el botón, notaremos que, después de unos pocos segundos, Visual Studio arrojará un error como este:

![](https://i.imgur.com/s0sn99V.png)

La razón de esto es que, para mantener el UI Thread libre, el manejador de eventos llamado *PositionChanged* se ejecuta en un hilo diferente del de la UI, que no tiene acceso directo al de la UI. Como la propiedad `Coordenadas` está conectada usando binding a un control en la página, tan pronto intentamos cambiar su valor obtendremos la excepción que se ve en la imagen.

Para esos escenarios la Plataforma Universal de Windows provee una clase llamada `Dispatcher`, que tiene la capacidad de despachar una operación en el UI Thread, sin importar cuál es el hilo donde está siendo ejecutada. El problema es que, típicamente, esta clase solo es accesible desde el code-behind, haciéndose díficil de usar desde un ViewModel. Consecuentemente, la mayoría de los toolkits MVVM proveen una forma de acceder al dispatcher también desde un ViewModel. En MVVM Light, se representa con la clase `DispatcherHelper`, que requiere ser inicializada cuando la aplicación  se inicia en el método `OnLaunched()` de la clase `App`:

{% highlight c# %}
    protected override void OnLaunched(LaunchActivatedEventArgs e)
            {
                Frame rootFrame = Window.Current.Content as Frame;
    
                // No repetir la inicialización de la aplicación si la ventana tiene contenido todavía,
                // solo asegurarse de que la ventana está activa.
                if (rootFrame == null)
                {
                    // Crear un marco para que actúe como contexto de navegación y navegar a la primera página.
                    rootFrame = new Frame();
    
                    rootFrame.NavigationFailed += OnNavigationFailed;
                    DispatcherHelper.Initialize();
    
                    if (e.PreviousExecutionState == ApplicationExecutionState.Terminated)
                    {
                        //TODO: Cargar el estado de la aplicación suspendida previamente
                    }
    
                    // Poner el marco en la ventana actual.
                    Window.Current.Content = rootFrame;
                }
    
                if (e.PrelaunchActivated == false)
                {
                    if (rootFrame.Content == null)
                    {
                        // Cuando no se restaura la pila de navegación, navegar a la primera página,
                        // configurando la nueva página pasándole la información requerida como
                        //parámetro de navegación
                        rootFrame.Navigate(typeof(MainPage), e.Arguments);
                    }
                    // Asegurarse de que la ventana actual está activa.
                    Window.Current.Activate();
                }
            }
{% endhighlight %}

`DispatcherHelper` es una clase estática, por lo que se puede llamar directamente al método `Initialize()` sin tener que crear una nueva instancia. Ahora que está inicializada, podemos empezar a utilizarla en nuestros ViewModels:

{% highlight c# %}
    private async void _geolocator_PositionChanged(Geolocator sender, PositionChangedEventArgs args)
    {
        await DispatcherHelper.RunAsync(() =>
        {
            Coordenadas = $"{args.Position.Coordinate.Point.Position.Latitude}, {args.Position.Coordinate.Point.Position.Longitude}";
        });
    } 
{% endhighlight %}

El código que necesita ser ejecutado en el UI Thread se rodea dentro de una *Action*, que es pasada como un parámetro del método asíncrono `RunAsync()`. Para mantener el UI Thread tan libre como sea posible, es importante rodear  dentro de esta acción el código que realmente se necesite ser ejecutado en el UI Thread y no otra lógica. Por ejemplo, si hubiéramos necesitado ejecutar algunas operaciones adicionales antes de fijar la propiedad `Coordenadas` (como convertir las coordenadas en una dirección de calle),  tendríamos que haberlo hecho por fuera del método `RunAsync()`.

## En el próximo post
En el próximo post veremos algunas librerías adicionales y helpers que podemos combinar con MVVM Light para hacernos la vida más fácil desarrollando aplicaciones UWP. 

Happy coding!

**El patrón MVVM: Series**
1. [Introducción](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/)
2. [La práctica](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/)
3. [Inyección de dependencias](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/)
4. [Escenarios avanzados](https://theshallowbay.github.io/tutoriales/2017/08/08/patron-mvvm-escenarios-avanzados/)
5. [Servicios, helpers y plantillas](https://theshallowbay.github.io/tutoriales/2017/08/09/patron-mvvm-servicios-helpers-plantillas/)
6. [Datos en tiempo de diseño](https://theshallowbay.github.io/tutoriales/2017/08/10/patron-mvvm-datos-tiempo-diseno/)