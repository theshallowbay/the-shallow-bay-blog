---
layout: post
title: El patrón MVVM - Introducción
date: 2017-07-02 11:45:00
summary: El patrón de diseño Model-View-ViewModel (de ahora en adelante, MVVM) es un tema de amor y odio cuando se trata del desarrollo de aplicaciones universales UWP. Si nunca lo has usado y lo intentas por primera vez, probablemente te encontrarás un poco confundido, porque es un enfoque completamente distinto del estándar basado en code-behind. Por otro lado si ya llevas un tiempo usando MVVM, probablemente no seas capaz de empezar cualquier proyecto usando otro enfoque diferente a MVVM. Esta es la razón por lo que escribimos este post ¿qué es MVVM? ¿por qué está tan ampliamente usado en el desarrollo de aplicaciones universales UWP y, generalmente hablando, por cualquier tecnología basada en XAML? Esperamos que, al final de este viaje, encuentres las respuestas a todas esas preguntas y que puedas empezas a usar el patrón MVVM en tus aplicaciones sin miedo.
categories: tutoriales
published: true
---

Lo primero que tienes que entender es que MVVM no es un *framework* o una librería, sino un **patrón**: no es un conjunto de APIs o de métodos, sino una forma de definir la arquitectura de una aplicación. Probablemente ya hayas escuchado hablar de [*MVVMLight*](http://www.mvvmlight.net/) o de [*Caliburn Micro*](http://caliburnmicro.com/), pero no hay porqué confundirlos con MVVM: esas son herramientas que ayudan a los desarrolladores a adoptar el patrón MVVM, pero no represenan al patrón mismo.

> NOTA: Este tutorial es una traducción propia de otro que se puede encontrar [aquí](http://wp.qmatteoq.com/the-mvvm-pattern-introduction/). Si deseas, puedes ir allí y leerlo en inglés. Todo el crédito va para el autor original. La intención de escribirlo aquí en español se rige de acuerdo a nuestra conducta de compartir el conocimiento.

El propósito del patrón es ayudar a los desarrolladores a definir la arquitectura de una aplicación. ¿Por qué es tan importante hacerlo? ¿Por qué no podemos simplemente continuar desarrollando una aplicación como siempre solemos hacerlo, que es escribiendo el todo código en una clase en el [*code-behind*](https://en.wiktionary.org/wiki/code-behind)? El enfoque estándar es muy rápido y simple de entender pero tiene muchas limitaciones cuando se trata de proyectos más complejos, que necesitan mantenimiento en algún momento en el tiempo. La razón de esto es que la clase en el *code-behind* tiene una muy fuerte dependencia de la página XAML. Consecuentemente, la mayoría del código no puede aislarse y terminaríamos mezclando la lógica de negocio y la capa de presentación.

**TABLA DE CONTENIDOS**
* Tabla de contenidos
{:toc}

A la larga, el enfoque con code-behind introduce muchos problemas:

1. **Es más complicado de mantener el código y hacer evolucionar el proyecto**. Cada vez que necesitemos agregar una nueva característica o solucionar un bug, se vuelve difícil entender dónde precisamente necesitaremos hacerlo, porque no hay una clara diferenciación entre los varios componentes de la aplicación. Esto se vuelve todavía más cierto si necesitamos reanudar el trabajo en un proyecto que ha estado "en remojo" por largo tiempo.
2. **Es complejo hacer pruebas unitarias**. Cuando se trata de proyectos complejos, muchos desarrolladores y compañías adoptan el enfoque de pruebas unitarias (*unit-testing*), que es una forma de llevar a cabo pruebas automáticas que validen pequeños fragmentos de código. Con estas pruebas se vuelve más fácil hacer evolucionar el proyecto: cada vez que agreguemos una nueva característica o que hagamos un cambio al código ya existente, podemos verificar fácilmente si el trabajo que hemos hecho ha roto o no alguna de las características existentes de la aplicación. Sin embargo, tener una fuerte dependencia entre la lógica y la interfaz de usuario hace casi imposible escribir pruebas unitarias, debido a que el código no está aislado.
3. **Es más complejo diseñar la interfaz de usuario**. Como hay una fuerte dependencia entre la interfaz de usuario y la lógica de negocio, no es posible para un diseñador enfocarse en la interfaz de usuario sin conocer todos los detalles de la implementación que hay detrás. Preguntas como "¿de dónde vienen los datos? ¿de una [base de datos](https://theshallowbay.github.io/tutoriales/2017/06/20/dise%C3%B1o-de-bases-de-datos/)? ¿un servicio en la nube?" no deberían ser preguntadas por un diseñador.

![](https://i.imgur.com/PZqu0lr.png)

***Code-behind*** de forma rápida: cada vez que agregamos una página XAML a nuestro proyecto, se crea una clase con el mismo nombre *debajo* de la página. Algunos desarrolladores llaman a esto "el cuarto de máquinas" pero este nombre no es tan *cool* como decir *code-behind*. Y no, en esas clases no debemos escribimos código a menos que sea **muy** estrictamente necesario. Ya veremos por qué.

El [fin último](http://www.mailxmail.com/curso-etica-moral-filosofia/fin-ultimo) del patrón MVVM es *romper* la fuerte relación entre el code behind y la interfaz de usuario, haciendo más fácil para un desarrollador entender cuáles son los diferentes componentes de la aplicación. Más precisamente, es fundamental distinguir los componentes que hacen parte de la lógica de negocios de los que manejan la presentación de los datos.

El nombre de este patrón viene del hecho de que un proyecto se puede divir en tres componentes diferentes, que vamos a explicar con más detalles:

> El fin último del patrón MVVM es *romper* la fuerte relación entre el code behind y la interfaz de usuario.

### El modelo
El modelo es un componente de la aplicación que define y maneja todas las entidades básicas de la aplicación. El objetivo de esta capa es remover cualquier dependencia de la forma en que los datos son representados. Idealmente, deberías ser capaz de tomar las clases que pertenecen a éste componente y usarlas en otra aplicación sin hacerles ningún cambio. Por ejemplo, si estás trabajando en una [aplicación que maneja los pedidos y los clientes](https://theshallowbay.github.io/tutoriales/2017/06/20/dise%C3%B1o-de-bases-de-datos/#otro-ejemplo-una-tienda-web) de una compañía, el modelo puede definirse por todas las clases que definen las entidades base, como un cliente, un pedido, un producto, etc.

![](http://img12.deviantart.net/2a19/i/2013/016/f/7/layered_database_source_documents_by_barrymieny-d5rnycs.jpg)

### La vista
La vista es el lado opuesto del modelo y se representa por la interfaz de usuario. En el mundo de las aplicaciones universales, las vistas son páginas XAML, que contienen todos los controles y las animaciones que definen la parte visual de la aplicación. Reciclando la ya mencionada aplicación de ejemplo que maneja pedidos y clientes, podemos tener múltiples vistas para mostrar la lista de clientes, de los productos disponibles, los pedidos hechos por un cliente, etc.

![](http://utekontakten-trondheim.no/wp-content/uploads/sites/29/2017/02/grafisk-design-app.jpg)

### El ViewModel
El ViewModel es el punto de conexión entre la vista y el modelo: se encarga de obtener los datos en crudo del modelo y manipularlos de tal forma que se muestren de forma correcta por la vista. La gran diferencia con una clase en code-behind es que el ViewModel es solo una clase plana y simple, sin ninguna dependencia de la vista. En una aplicación basada en el patrón MVVM, típicamente se crea un ViewModel por cada vista.

![](http://i.imgur.com/uD5pADO.png)

![](https://i.imgur.com/M5Ue5T6.png)

## ¿Por qué el patrón MVVM?
Luego de desta breve introducción, debería ser más fácil entender por qué el patrón MVVM es tan importante y cómo, al adoptarlo, podemos resolver todos los problemas mencionados al inicio de este post.

1. Al dividir el código en tres diferentes capas se vuelve más fácil (especialmente si estás trabajando en un equipo) mantener y evolucionar la aplicación. Si necesitas agregar una característica o resolver un bug, es más fácil identificar qué capa se tiene que manipular. Es más, como no hay dependencias entre cada capa, el trabajo puede hacerse en paralelo (por ejemplo, un diseñador puede empezar a trabajar en la interfaz de usuario mientras otro desarrollador puede crear los servicios que usará la página para obtener datos)
2. Para realizar pruebas unitarias de forma apropiada, el código a probar tiene que estar aislado y ser tan simple como sea posible. Cuando se trabaja con un enfoque code-behind, esto simplemente no es posible: a menudo la lógica está conectada a un manipulador de eventos (*event handler*, por ejemplo, porque el código tiene que ejecutarse cuando se presiona un botón) y necesitarías encontrar una forma de simular el evento con el fin de accionar/disparar (*trigger*) el código a probar. Al adoptar el patrón MVVM se rompe esta fuerte dependencia: el código incluído en un ViewModel puede fácilmente aislarse y probarse.
3. Como se ha roto la fuerte dependencia entre la interfaz de usuario y la lógica de negocio, para un diseñador es más fácil definir la interfaz sin tener que conocer todos los detaller de la implementación de la aplicación. Por ejemplo, si el diseñador tiene que trabajar en una nueva página que muestra una lista de pedidos, podemos fácilmente reemplazar el ViewModel real (que obtiene los datos de una fuente de datos real, como un servicio en la nube o una base de datos) con uno falso, que pueda generar datos falsos que permitan al diseñador entender fácilmente qué tipo de información debería mostrar la página.

***¿Por qué en el mundo de las aplicaciones universales UWP la mayoría de desarrolladores tienden a usar el patrón MVVM y no otros patrones populares como MVC o MVP?*** Principalmente, porque el patrón MVVM está basado en varias de las características que hacen parte del núcleo del runtime de XAML, como el *binding*, las *dependency properties*, etc. En este post hablaremos un poco más acerca de esas características. Puedes notar como he acabado de mencionar el runtime de XAML y no la Plataforma Universal de Windows (UWP): la razón de esto es que la mayoría de las cosas que vamos a ver en este post no son específicas al mundo de las aplicaciones UWP, sino que pueden aplicarse a cualquier tecnología basada en XAML, como WPF (*Windows Presentation Fundation*), Silverlight (ahora en desuso, como Flash), Windows Phone (versiones anteriores a la 10), Xamarin, etc.

Veamos ahora los detalles, que son las características básicas de XAML tratadas en el patrón MVVM.

## El binding
El binding (se podría traducir como *unión*, *atadura*, *encadenamiento*, llámalo como quieras, que en inglés *bind* es una cadena) es una de las características más importantes de XAML que permiten **crear un canal de comunicación entre dos diferentes propiedades**. Pueden ser propiedades que pertenezcan a diferentes controles XAML, o una propiedad declarada en el código con una propiedad de control. La característica principal tratada por el patrón MVVM es la segunda: las *Vistas* y los *ViewModel* se conectan gracias al  binding. El ViewModel se encarga de exponer los datos a mostrar en las Vistas como si fuesen *propiedades*, que estarán conectadas a los controles que los mostrarán usando binding. 

Digamos, por ejemplo, que tenemos una página en una aplicación que muestra una lista de productos. El ViewModel se encargará de obtener esta información (por ejemplo, de una base de datos local) y guardarla en una propiedad específica (como una colección del tipo `List<Order>`):

{% highlight c# %}
    public List<Order> Orders { get; set; }
{% endhighlight %}

Para mostrar la colección en una aplicación tradicional con code-behind, en el mismo punto, se asignaría esta propiedad manualmente a la propiedad `ItemsSource` de un control como un `ListView` o un `GridView`, como en el siguiente ejemplo:

{% highlight c# %}
    MyList.ItemsSource = Orders;
{% endhighlight %}

Sin embargo, este código crea una fuerte dependencia entre la lógica y la interfaz: como estamos accediendo a la propiedad ItemsSource usando el nombre del control, sólo podemos llevar a cabo esta operación en la clase del code behind.

Con el patrón MVVM, en cambio, se conectan propiedades en el ViewModel con los controles en la interfaz usando binding, como en el siguiente ejemplo:

{% highlight xml %}
    <ListView ItemsSource="{Binding Path=Orders}" />
{% endhighlight %}

De esta forma, se ha roto la dependencia entre la interfaz de usuario y la lógica, porque la propiedad `Orders` puede definirse también en una clase simple y plana como un ViewModel.

Como ya se mencionó, el binding puede también ser bidireccional: este enfoque se usa cuando no solamente el ViewModel necesita mostrar datos en la Vista, sino que también la Vista debe ser capaz de poder cambiar el valor de una de las propiedades del ViewModel. Digamos que tu aplicación tiene una página donde el usuario pueda crear un nuevo pedido y, consecuentemente, incluye un control `TextBox` donde especificar el nombre del producto. Esta información necesita ser manejada por el ViewModel, porque él se encargará de interactuar con el modelo y de agregar el pedido a la base de datos. En este caso, se aplica el atributo `Mode` del binding y se especifica como `TwoWay`, así cada vez que el usuario agregue texto al control TextBox, la propiedad conectada en el ViewModel obtendrá el valor escrito.

Si, por ejemplo, en XAML tenemos el siguiente código:

{% highlight xml %}
    <TextBox Text="{Binding Path=ProductName, Mode=TwoWay}" />
{% endhighlight %}

significa que en el ViewModel tendremos una propiedad llamada `ProductName`, que guardará el texto escrito por el usuario en el TextBox.

## El DataContext
En la sección inmediatamente anterior vimos cómo, gracias al binding, podemos conectar las propiedades del ViewModel a los controles en una página XAML. Ahora te estarás preguntando cómo es que la Vista puede ser capaz de entender cuál es el ViewModel que pobla sus datos. Para entender esto, es necesario introducir el concepto de **DataContext**, que es una propiedad ofrecida por todos los controles XAML. La propiedad *DataContext* define el contexto del binding: cada vez que se configura una clase como el DataContext de un control, se puede acceder a todas sus propiedades públicas. Además, el DataContext es jerárquico: las propiedades no sólo pueden acceder por medio del control, sino que todos los controles hijos también pueden acceder a ellas.

El núcleo de la implementación del patrón MVVM recae en su jerarquía: **la clase que se crea como el ViewModel de una Vista se define como el DataContext de la página entera**. Consecuentemente, cada control que coloquemos en la página XAML podrá acceder a las propiedades del ViewModel y mostrarlas o administrar la información. En una aplicación desarrollada con el patrón MVVM, usualmente, se termina con una declaración en la página como esta:

{% highlight xml %}
    <Page x:Class="Ejemplo.MainPage"
		  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
		  xlmns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
		  DataContext="{Binding Source={StaticResource MainViewModel}}"
		  mc:Ignorable="d">
		  
		  <!-- El contenido de la página va aquí -->
		  
	</Page>
{% endhighlight %}

La propiedad **DataContext** de la clase **Page** se ha conectado a una nueva instancia de la clase **MainViewModel**.

## La interfaz INotifyPropertyChanged
Si intetamos crear una aplicación simple basada en el patrón MVVM aplicando los conceptos que hemos aprendido, rápidamente crearemos un gran problema. Usemos el ejemplo anterior de la página para crear un nuevo pedido y digamos que tenemos, en el ViewModel, una propiedad que usamos para mostrar el nombre del producto, como la siguiente:

{% highlight c# %}
    public string NombreProducto { get; set; }
{% endhighlight %}

De acuerdo a lo que hemos aprendido, esperamos tener un control *TextBlock* en la página para mostrar el valor de esta propiedad, como sigue:

{% highlight xaml %}
    <TextBlock Text="{Binding Path=NombreProducto}" />
{% endhighlight %}

Ahora digamos que, durante la ejecución de la aplicación, el valor de la propiedad **NombreProducto** cambia (por ejemplo, porque se termina una operación de cargado de datos). Notaremos como, a pesar del hecho de que el ViewModel guardará apropiadamente el nuevo valor de la propiedad, el control TextBlock seguirá mostrando la antigüa. La razón de esto es que el binding no es suficiente para manejar la conexión entre la Vista y el ViewModel.

El binding ha creado un canal entre la propiedad *NombreProducto* y el control *TextBlock*  pero ninguno ha notificado al otro lado del canal que el valor de la propiedad ha cambiado. Para este propósito, XAML ofrece el concepto de ***dependency properties***, que son propiedades especiales que pueden definir un comportamiento complejo y, bajo el capó, son capaces de enviar una notificación a ambos lados del canal del binding cada vez que su valor cambia. La mayoría de los controles básicos XAML usan dependency properties (por ejemplo, la propiedad *Text* de un control TextBlock es una dependency property). Sin embargo, definir una nueva dependency property no es algo sencillo y, en la mayoría de los casos, ofrece características que no son necesarias para nuestro escenario MVVM.

Tomemos el ejemplo anterior basado en la propiedad *NombreProducto*, no necesitamos manipular ningún comportamiento o lógica especial, solamente necesitamos que, cada vez que la propiedad NombreProducto cambie, ambos lados del canal del binding reciban una notificación, de forma que el control TextBlock pueda actualizar su diseño visual y mostrar el nuevo valor.

Para esos escenarios, XAML ofrece una interfaz específica llamada ***INotifyPropertyChanged***, que podemos implementar en nuestros ViewModels. De esta forma, si necesitamos notificar a la interfaz cuando cambia el valor de una propiedad, no necesitamos crear una dependency property compleja, sino solamente implementar esta interfaz e invocar el método reacionado cada vez que el valor de la propiedad cambie.

Así es como se ve un ViewModel que implementa esta interfaz:

{% highlight c# %}
    public class MainViewModel: INotifyProperyChanged
    {
	    public event PropertyChangedEventHandler PropiedadHaCambiado;
	    [NotifyPropertyChangedInvocator]
	    protected virtual void OnPropertyChanged([CallerMemberName] string nombrePropiedad = null)
	    {
		    PropiedadHaCambiado?.Invoke(this, new PropertyChangedEventArgs(nombrePropiedad));
		}
	}
{% endhighlight %}

Puedes notar cómo la implementación de esta interfaz permite llamar a un método llamado **OnPropertyChanged()**, que podemos invocar cada vez que el valor de una propiedad cambia.
Sin embargo, para lograr esto, necesitamos cambiar la manera como definimos las propiedades dentro de nuestro ViewModel. Cuando se trata de propiedades simples, usualmente se definen usando sintaxis corta:

{% highlight c# %}
    public string NombreProducto { get; set; }
{% endhighlight %}

Sin embargo, con esta sintaxis no se puede cambiar qué pasa cuando el valor de la propiedad es leído o escrito. Por tal motivo, es necesario ir atrás y usar el enfoque anterior, basado en una variable privada que guarde temporalmente el valor de la propiedad. De esta forma, cuando se escribe el valor, se puede invocar el método **OnPropertyChanged()** y despachar y enviar la notificación. Así es como una propiedad en un ViewModel se ve:

{% highlight c# %}
    private string _nombreProducto;
    public string NombreProducto
    {
	    get { return _nombreProducto; }
	    set
	    {
		    _nombreProducto = value;
		    OnPropertyChanged();
		}
	}
{% endhighlight %}

Ahora la propiedad trabajará como se esperaba: cuando cambia su valor, el control *TextBlock* en binding con ella cambiará su apariencia para mostrarla.

## Comandos (o cómo manejar eventos en MVVM)
Otro escenario crítico cuando se trata de desarrollar una aplicación es manejar las interacciones con el usuario: él puede presionar un botón, escoger un ítem de una lista, etc. En XAML, esos escenarios se manejan usando eventos que son expuestos por varios controles. Por ejemplo, si quieres manejar cuando se presiona un botón, necesitamos suscribirnos  a un evento **Click**, como en el siguiente ejemplo:

{% highlight xml %}
    <Button Content="Haz clic aquí" Click="OnButtonClicked" />
{% endhighlight %}

El evento *Click* es administrado por un *event handler*, que es un método que incluye, entre varios parámetros, alguna información que es útil para entender el contexto del evento (por ejemplo, qué control acciona el evento o qué elemento de la lista ha sido seleccionado), como en el siguiente ejemplo:

{% highlight c# %}
    private void OnButtonClicked(object sender, RoutedEventArgs e)
    {
	    // Hacer algo
	}
{% endhighlight %}

El problema con este enfoque es que los event handlers tienen una fuerte dependencia con la Vista: sólo pueden declararse, de hecho, en la clase del code-behind. Cuando se crea una aplicación usando el patrón MVVM, en cambio, todos los datos y la lógica son usualmente definidos en el ViewModel, así que es necesario encontrar una manera de manejar la interacción con el usuario desde allí.

Para este propósito, XAML tiene los **commands**, que son una manera de expresar una interación del usuario con una propiedad en lugar de usar un event handler. Como es una propiedad simple, se puede romper la fuerte conexión entre la Vista y el event handler y se puede también definir en una clase independiente, como un ViewModel.

El *framework* ofrece la interfaz **ICommand** para implementar comandos: con el enfoque estándar, se termina teniendo una clase separada para cada comando. El siguiente ejemplo muestra como luce un comando:

{% highlight c# %}
    public lass ClickCommand : ICommand
    {
	    public bool CanExecute(object parameter)
	    {
	    }
	    
	    public void Execute(object parameter)
	    {
	    }
	    
	    public event EventHandler CanExecuteChanged;
	    
	}
{% endhighlight %}

El núcleo del comando es el método **Execute()**, que contiene el código que ha de ejecutarse cuando se invoca el comando (por ejemplo, porque el usuario ha presionado un botón). Es el código que, en una aplicación tradicional, se pudo haber escrito dentro de un event handler.

El método **CanExecute()** es una de las características más interesantes que proveen los comandos, porque puede usarse para manejar el ciclo de vida del comando cuando la aplicación está ejecutándose. Por ejemplo, digamos que tenemos una página con un formulario que se puede llenar, con un botón al final de la página que el usuario presiona para enviar el formulario. Como todos los campos en el formularios son obligatorios, queremos que el botón esté desactivado hasta que todos los campos hayan sido llenados. Si manejamos la operación para enviar el formulario con un comando, podemos implementar el método *CanExecute()* de una forma que devuelva *false* cuando haya al menos un campo vacío. De esta forma, el control *Button* que hemos enlazado al comando automáticamente cambiará su estado visual: estará desactivado y el usuario inmediatamente entenderá que no podrá presionarlo.

![](http://i.imgur.com/1UNWXL1.png)

Al final, el comando ofrece un evento llamado **CanExecuteChanged**, que podemos invocar dentro del ViewModel cada vez que cambie la condición que queremos monitorizar para manejar el estado del comando. Por ejemplo, en el ejemplo anterior, podemos llamar al evento *CanExecuteChanged* cada vez que el usuario llene uno de los campos del formulario.

Una vez hemos definido un comando, podemos enlazarlo al XAML gracias a la propiedad **Command**, que es expuesta por cada control capaz de manejar interacciones con el usuario (como Button, RadioButton, etc.)

{% highlight xaml %}
    <Button Content="Haz click aquí" Command="{Binding Path=ClickCommand}" />
{% endhighlight %}

Como veremos más adelante, la mayoría de los *toolkits* y de *frameworks* que implementan el patrón MVVM ofrecen una manera fácil de definir un comando, sin forzar al desarrollador a crear una nueva clase por cada comando que tenga la aplicación. Por ejemplo, el toolkit popular MVVM Light ofrece una clase llamada **RelayCommand**, que puede usarse para definir un comando de la siguiente forma:

{% highlight c# %}
    private RelayCommand _diHola;
    public RelayCommand DiHola
    {
	    get
	    {
		    if (_diHola == null)
		    {
			    _diHola = new RelayCommand(() =>
			    {
				    Mensaje = string.Format("¡Hola {0}", Nombre);
				}, () => !string.IsNullOrEmpty(Nombre));
			}
			
			return _diHola;
		}
	}
{% endhighlight %}

Como se puede ver, no es necesario definir una nueva clase por cada comando sino que, al usar método anónimos, se puede simplemente crear un nuevo objeto **RelayCommand** y pasarle, como parámetros:

1. El código que se quiere ejecutar cuando se invoque el comando
2. El código que evalúe si el comando está habilitado o no

## Cómo implementar el patrón MVVM: toolkits y frameworks

Como hemos mencionado al inicio del post, MVVM es un patrón, que no una librería o un framework. Sin embargo, como hemos aprendido hasta ahora, cuando se crea una aplicación basada en este patrón se necesita especificar un conjunto de procedimientos estándar: implementar la interfaz *INotifyPropertyChanged*, manejar comandos, etc.

Consecuentemente, muchos desarrollados han empezado a trabajar en librerías que pueden ayudar con el trabajo de otros desarrolladores, permitiéndoles enfocarse en el desarrollo de la aplicación misma, en lugar de cómo implementar el patrón. Veamos algunas de las librerías más populares:

### MVVM Light

![](http://www.galasoft.ch/layout/mvvm/MVVM_BlackText_190x147.png)

[MVVM Light](http://www.mvvmlight.net/) es una librería creada por Laurent Bugnion, un MVP longevo y uno de los desarrolladores más populares en el mundo Microsoft. Esta librería es muy popular gracias a su flexibilidad y su simplicidad. MVVM Light, de hecho, ofrece sólo las herramientas básicas para implementar el patrón, como:

- Una clase base, de cual el ViewModel puede heredar, para obtener acceso a algunas características básicas como las notificaciones
- Una clase base para manejar comandos
- Un sistema de mensajería básico, para manejar la comunicación entre clases diferentes (como dos ViewModels)
- Un sistema básico para manejar la inyección de dependencias, que son una forma alternativa de inicializar ViewModels y manejar sus dependencias. Aprenderemos más de este concepto más adelante.

Como MVVM Light es muy básico, puede usarse no solamente en aplicaciones universales UWP, sino también en WPF, Silverlight e incluso Android e iOS gracias a su compatibilidad con Xamarin. Como es extremadamente flexible, es también fácil de adaptar a los requerimientos y usarla como punto de partida. Esta simplicidad, sin embargo, es también la debilidad de MVVM Light. Como veremos más adelante, cuando se crea una aplicación universal UWP usando el patrón MVVM nos enfrentaremos con varios desafíos, debido a que varios conceptos vásicos y características de la plataforma (como la navegación entre diferentes páginas) sólo puede manejarse en una clase code-behind. Desde este punto de vista, MVVM Light no ayuda mucho al desarrollador: como solamente ofrece las herramientas básicas para implementar el patrón, todo lo demás depende del desarrollador. Por estas razones, también se pueden encontrar otras librerías adicionales (como el [*Cimbalino Toolkit*](http://cimbalino.org/)) que extiende MVVM Light y agrega un conjunto de servicios y características que son útiles cuando se trata de desarrollar una aplicación universal UWP.

### Caliburn Micro
[Caliburn Micro](http://caliburnmicro.com/) es un framework originalmente creado por Rob Einsenberg y ahora mantenido por Nigel Sampson y Thomas Ibel. Si MVVM Light es un toolkit, Caliburn Micro es un framework completo, que ofrece un enfoque completamente diferente. Comparado con MVVM Light, de hecho, Caliburn Micro un rico conjunto de servicios y de características que son específicas de resolver algunos de los desafíos que propone la Plataforma Universal de Windows como la navegación, el almacenamiento, los contratos, etc.

Caliburn Micro maneja la mayoría de las características básicas del patrón con convención de nombres: la implementación del binding, los comandos y otros conceptos están ocultos por un conjunto de reglas, basado en los nombres que necesitemos asignar a los varios componentes del proyecto. Por ejemplo, si queremos conectar una propiedad del ViewModel a un control XAML, no necesitamos definir manualmente un binding: podemos simplemente darle al control el mismo nombre de la propiedad y Caliburn Micro aplicará el binding por nosotros. Esto es posible gracias a un *bootstrapper*, que es una clase especial que reemplaza la clase estándar App y se ocupa de inicializar, además de la aplicación misma, la infraestructura Caliburn.

Caliburn Micro es, sin lugar a dudas, muy poderoso, porque inmediatamente te da acceso a todas las herramientas requeridas para desarrollar apropiadamente una aplicación universal UWP usando el patrón MVVM. Sin embargo, en nuestra opinión, no es la mejor opción si eres nuevo en esto del patrón MVVM: como oculta la mayoría de los conceptos básicos que son parte del núcleo del patrón, puede ser complejo para un desarrollador nuevo entender qué está pasando y cómo están conectadas las diferentes piezas de la aplicación.

### Prism
[Prism](http://github.com/PrismLibrary/Prism) es otro framework popular que, al principio, fue [creado y mantenido](http://blogs.msdn.com/b/dotnet/archive/2015/03/19/prism-grows-up.aspx) por la división Pattern & Practises de Microsoft. Ahora, en cambio, se ha convertido en un proyecto comunitario, mantenido por un grupo de desarrolladores independientes y MVPs de Microsoft.

Prism es un framework que usa un enfoque similar al que ofrece Caliburn Micro: ofrece convención de nombres, para conectar diferentes piezas de la aplicación, e incluye un rico conjunto de servicios para resolver los desafíos de la UWP.

Se puede decir que se sitúa en medio de MVVM Light y Caliburn Micro, cuando se trata de complejidad: no es tan simple y flexible como MVVM Light, pero, al mismo tiempo, no usa la conveción de nombres de forma agresiva como lo hace Calibun Micro.

**Qué viene después**
En los próximos posts vamos a convertir lo que hemos aprendido en un proyecto real y vamos a utilizar MVVM Ligth para este propósito: la razón de esto es que, como ya hemos mencionado, creemos que es más fácil de entender, especialmente si eres nuevo en el patrón, porque ayuda a entender todos los conceptos básicos que son parte del núcleo del patrón.

Happy coding!

**El patrón MVVM: Series**
1. [Introducción](https://theshallowbay.github.io/tutoriales/2017/07/02/patron-mvvm-introduccion/)
2. [La práctica](https://theshallowbay.github.io/tutoriales/2017/07/04/patron-mvvm-practica/)
3. [Inyección de dependencias](https://theshallowbay.github.io/tutoriales/2017/08/07/patron-mvvm-inyeccion-de-dependencias/)
4. [Escenarios avanzados](https://theshallowbay.github.io/tutoriales/2017/08/08/mvvm-escenarios-avanzados/)
5. [Servicios, helpers y plantillas](https://theshallowbay.github.io/2017/08/09/mvvm-servicios-helpers-plantillas/)
6. [Datos en tiempo de diseño](https://theshallowbay.github.io/tutoriales/2017/08/09/datos-tiempo-diseno/)
