---
layout: post
title: Rogue, crea una aplicación con el diseño de menú hamburguesa
date: 2016-06-27 01:27:10
summary: Conoce cómo se construyen las plantillas prediseñadas para empezar tus propios diseños de interfaces. Con Rogue haremos una primera aproximación a la arquitectura del patrón de navegación con el menú hamburguesa.
categories: templates
---


#On SplitView control

Una arquitectura de navegación en la aplicación, a veces mal llamada patrón de *menú hamburguesa*. Aprendamos a crearla y diseñarla desde cero, pero también poniendo a disposición una plantilla para facilitar el trabajo y ahorrar tiempo.

Cuando Microsoft presentó públicamente Windows 10 dejó ver la dirección hacia la que se movían las nuevas líneas de diseño del lenguaje *Metro*, o  *Modern UI*, como quieras llamarlo. Con el nuevo sistema, también llegó el programa *Insider*, una nueva versión del Visual Studio, y por supuesto nuevo SDK, que incluía muchas mejoras notables respecto a las versiones anteriores. Nuevas características y nuevos controles aparecieron, tales como SplitView y RelativePanel, que permiten tener un mejor control sobre el diseño de la interfaz.

Del control que nos interesa, Splitiew, como se llama en este contexto, se puede decir mucho. Porque es el fundamento de aquella arquitectura para navegación que muchos llaman *menú hamburguesa*, y que tanto nos gusta odiar. 

Resulta que muchas de las nuevas aplicaciones nativas hacen uso del control, dando unos resultados interesantes. Cada desarrollador / diseñador decide hasta que punto se implementa o se recurre a otras ténicas de arquitectura.

![Aplicaciones que hacen uso de un SplitView](https://66.media.tumblr.com/bcd2286d65a03f7c1b0289ad5d463602/tumblr_inline_o8odp6iQH01r7pczk_540.png)

Varias de las aplicaciones que están en nuestra hoja de ruta también hacen uso del control, por lo menos en los borradores y versiones beta internas que estamos probando.

Así que hemos decidido enfrentarnos a ello y dejarle el camino más fácil a aquellos que se decidan a hacer lo mismo. Vamos a explicar como crear una aplicación para Windows 10 que haga uso del control, y al final, también dejaremos disponible la plantilla para que sea incluso más fácil empezar desde ese punto, ahorrando tiempo y enfocando recursos en otros aspectos como decidir el diseño visual y la lógica de navegación. Aunque ya existen [iniciativas](https://github.com/Windows-XAML/Template10) con la misma idea, hemos tenido varios problemas para usar esas otras plantillas.

> NOTA: En el desarrollo de esta y de las demás aplicaciones hacemos uso de la [versión estable](http://insideten.xyz) más reciente de Windows 10, de Visual Studio y del SDK.

### Calentando motores
Antes de empezar debemos dejar claro que se deben conocer conceptos importantes del contexto en el que nos estamos moviendo, tales como [MVVM](http://www.codeproject.com/Articles/100175/Model-View-ViewModel-MVVM-Explained). Mi explicación favorita del tema es [ésta](http://wp.qmatteoq.com/the-mvvm-pattern-introduction/).

Si indagamos en el [GitHub](https://github.com/Microsoft) de Microsoft encontramos los [ejemplos oficiales](https://github.com/Microsoft/Windows-universal-samples) del Universal Windows Platform,  que podemos clonar en nuestro ordenador para ver cómo funcionan y se comportan los nuevos controles universales.

El [proyecto](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/XamlNavigation) que nos interesa y sobre el cual nos vamos a basar es el llamado *Navegation Menu (XAML) Sample*.

### Comprende cómo funciona
Si uno descarga el repositorio, busca el ejemplo mencionado, abre la solución del Visual Studio y le da al Deploy, ve una aplicación como esta:

![](https://40.media.tumblr.com/c86f290fc2c0e8e571cc0814a42220e2/tumblr_inline_o1j2lu6jlp1r7pczk_540.png)
> A la izquierda, el Pane y a la derecha el Content del control Splitview

Cabe resaltar las propiedades del control Splitview y los otros componentes usados en la interfaz, como el PageHeader donde va el título de la página o el botón mismo del menú hamburguesa. De eso hablamos más adelante.

### Configurando lo básico

Manos a la obra. Vamos a Visual Studio 2015 y seleccionamos **File -> New -> Project** le ponemos el nombre que queremos que tenga nuestra aplicación, lo ubicamos en la ruta que queremos y le damos al botón **OK**

![](https://40.media.tumblr.com/1460343cea09c254dd468623c9bf83e2/tumblr_inline_o1j2deYDfs1r7pczk_540.png)

Esperamos que Visual Studio nos construya la nueva solución con su respectivo único proyecto y, aunque parezca algo sin importancia, vamos a agregar las carpetas para tener organizado el código. Damos clic derecho al proyecto **Add -> New folder** y agregamos las capetas:

- Views, para las vistas que son todas las páginas XAML
- Models, para nuestras clases que sirvan de *Modelos*
- ViewModels, para conectar la interfaz con los modelos
- Styles, para guardar nuestros diccionarios de recursos y estilos que también son páginas XAML
- Controls, para almacenar controles de la interfaz pero que no van en las vistas

El proyecto debería verse algo como:

![](https://41.media.tumblr.com/4c46bbe83337171e7bb89027c8076d1c/tumblr_inline_o1j3dmf4XO1r7pczk_540.png)


Dado que vamos a hacer uso del patrón MVVM, utilizaremos [MVVMLight](http://www.mvvmlight.net/) para adelantar parte del trabajo. Damos clic derecho sobre el proyecto, luego seleccionamos **Manage NuGet packages -> Browse** y buscamos por mvvmlight, lo instalamos y aceptamos la licencia de uso.

Lo siguiente que debemos hacer es borrar la página MainPage.xaml porque recuerda que *todas* las vistas deben ir en la carpeta Views.

Es importante recalcar ahora que, como no tenemos absolutamente ninguna vista en el proyecto si intentamos hacer el Deploy nos va a mostrar un error proveniente de *App.xaml.cs*, que resulta que en esa clase tenemos el código que se ejecuta tan pronto se lanza la aplicación (se crea una nueva instancia de ella).

La sobreescritura (en inglés *override*) del método OnLaunched es la que más nos interesa pues hay que hacer algunas modificaciones interesantes para hacer funcionar el patrón. Si eres una persona observadora, habrás leído el mensaje que aparece en la captura de pantalla de como luce el proyecto de ejemplo que provee Microsoft una vez le damos al Deploy.

Habla de que la raíz de la aplicación es un *AppShell* del tipo Page en lugar de un Frame. Se utiliza un control SplitView para presentar dos cosas: el menú de la navegación de alto nivel (el botón con las tres líneas horizontales de toda la vida junto a un PageHeader) y un Frame para navegar entre las páginas.

### Instanciación de la aplicación

No estoy seguro de si ese verbo exista en español, pero es algo así como cada vez que ejecutamos la aplicación, ésta se *instancia*. Cuando esto ocurre se invoca al método *OnLaunched* ubicado en *App.xaml.cs*.

Vamos a crear nuestro *marco* que de ahora en adelante lo vamos a llamar Shell. Clic derecho en la carpeta Views y seleccionamos **Add -> New Item -> BlankPage** y la llamamos AppShell.xaml. En nuestro Shell es donde van ubicados los controles de la navegación de alto nivel y además tenemos lo que nos interesa, una nueva clase llamada *AppShell.xaml.cs*

### El control Splitview

Tal y como explica Javier Suarez en su [blog](https://javiersuarezruiz.wordpress.com/2015/04/21/windows-10-control-splitview/) este control dispone de las siguientes propiedades:

- **PanePlacement**, posición del panel lateral. Se puede tener a la izquierda o a la derecha
- **PaneBackground**, color de fondo del Pane
- **OpenPaneLength:** ancho en píxeles del Panel abierto.
- **CompactPaneLength:** ancho en píxeles del Panel cerrado.
- **IsPanelOpen:** propiedad de tipo bool que nos permite tanto saber como establecer si el Panel esta abierto no.

Son algunas, pero no todas, propiedades importantes de este control. Una de las propiedades mas destacables es *DisplayMode*, como ilustra la imagen:

![](https://javiersuarezruiz.files.wordpress.com/2015/04/splitview-07.png)

En *AppShell.xaml* agregamos el control, lo llamamos RootSplitView y lo personalizamos:

    <Splitview x:Name="RootSplitView"
			   DisplayMode="Inline"
               OpenPaneLength="256"
               IsTabStop="False">
	</Splitview>

![](http://az648995.vo.msecnd.net/win/2015/09/3_splitView.png)

En Pane, es donde vamos a ubicar una lista de las páginas hacia donde podemos navegar y un control ListView es perfecto para ese propósito. Vamos a crear uno personalizado: clic derecho en la carpeta *Controls* y agregamos una clase pública llamada *NavMenuListView.cs* que herede de ListView.

> Para acelerar el trabajo, puedes simplemente descargar la [clase](https://github.com/Microsoft/Windows-universal-samples/blob/master/Samples/XamlNavigation/cs/Controls/NavMenuListView.cs) *NavMenuListView* y añadirla al proyecto o copiar y pegar todo el código. Ten en cuenta los namespaces.

Antes de empezar a trabajar con la clase que acabamos de crear, vamos a añadir las Views que necesitaremos, en este caso agregaremos 3:

- **BasicPage**, una página sencilla
- **CommandBarPage**, una página con comandos
- **DrillInPage**, una página con más navegación

Y además de eso, vamos a crear el Model para esa lista de items que queremos ubicar en el Pane clic derecho en la carpeta **Models -> Add -> Class** y la llamamos *NavMenuItem.cs*

> Puedes simplemente descargar la [clase](https://github.com/Microsoft/Windows-universal-samples/blob/master/Samples/XamlNavigation/cs/NavMenuItem.cs) *NavMenuItem* y añadirla al proyecto o copiar y pegar todo el código. Ten en cuenta los namespaces.

Vamos de nuevo a *AppShell.xaml* y agregamos el nombre de espacios para poder hacer el binding, de la siguiente manera:

    xmlns:controls="using:menunavigation.Controls"
	xmlns:item="using:menunavigation.Models"

El nombre que aparece después de *using:* es nuestro namespace donde están las clases que hemos creado anteriormente. Hay veces que Visual Studio se vuelve perezoso y nos muestra un error diciendo que no se puede tener acceso al namespace. Una solución a este problema es *construir* la aplicación, damos clic en **Build -> Build Solution**

Ahora necesitamos crear la plantilla para mostrar esos datos. En los recursos de *AppShell.xaml*:

    <DataTemplate x:Key="NavMenuItemTemplate" x:DataType="item:NavMenuItem">
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="48" />
                    <ColumnDefinition />
                </Grid.ColumnDefinitions>             
                <FontIcon x:Name="Glyph" FontSize="16" Glyph="{x:Bind SymbolAsChar}" VerticalAlignment="Center" HorizontalAlignment="Center" ToolTipService.ToolTip="{x:Bind Label}"/>
                <TextBlock x:Name="Text" Grid.Column="1" Text="{x:Bind Label}" />
            </Grid>
        </DataTemplate>

En la anterior porción de código aparece un comentario que invoca lo siguiente:

> Showing a ToolTip and the Label is redundant.  We put the ToolTip on the icon.
It appears when the user hovers over the icon, but not the label which provides value when the SplitView is 'Compact' while reducing the likelihood of showing redundant information when the label is shown.

En nuestra anterior plantilla tenemos un Grid que dividimos en dos columnas: una que mide 48 y otra que es automática. Un control FontIcon que se encarga de mostrar un icono y un control TextBlock que muestra el nombre de la página a donde lleva.

Visual Studio me pidió *construir* el proyecto después de añadir el DataTemplate.

Trabajemos ahora en el code-behind de *AppShell.xaml*, abrimos *AppShell.xaml.cs* porque necesitamos añadir los eventos de navegación, de selección... Para adelantar trabajo:

> Puedes simplemente descargar la [clase](https://github.com/Microsoft/Windows-universal-samples/blob/master/Samples/XamlNavigation/cs/AppShell.xaml.cs) *AppShell.xaml.cs* y reemplazarla en el proyecto o copiar y pegar todo el código. Ten en cuenta los namespaces. Nos aparecerán varios errores, pero los ignoramos mientras vamos al siguiente paso.

### La clase App.xaml.cs

Seguimos trabajando en esta clase. Debemos reemplazar el código por [este](https://github.com/Microsoft/Windows-universal-samples/blob/master/Samples/XamlNavigation/cs/App.xaml.cs) (ten siempre en cuenta los namespaces, es muy importante o aparecerán errores)

### Construcción de la interfaz

Ahora estamos listos para añadir el control en el Pane que se encargará de mostrarnos la lista de páginas navegables. Vamos a nuestro *AppShell.xaml* y añadimos dentro del SplitView:

    <controls:NavMenuListView x:Name="NavMenuList"
                                          TabIndex="3"
                                          Margin="0,48,0,0"
                                          ContainerContentChanging="NavMenuItemContainerContentChanging"
                                          ItemContainerStyle="{StaticResource NavMenuItemContainerStyle}"
                                          ItemTemplate="{StaticResource NavMenuItemTemplate}"
                                          ItemInvoked="NavMenuList_ItemInvoked" />

Inmediatamente nos aparecerá en error indicando que no se puede resolver un recurso. Es porque nos falta un estilo. Vamos a la carpeta Styles clic derecho **Add -> New Item -> XAML View** y lo llamamos *DefaultStyles.xaml*. Borramos todo el código que aparezca y lo reemplazamos por [este](https://github.com/Microsoft/Windows-universal-samples/blob/master/Samples/XamlNavigation/cs/Styles/Styles.xaml) ten en cuenta los namespaces. Algo importante para poder usar el diccionario que acabamos de crear es registrarlo en *App.xaml* por lo que le debemos añadir:

    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Styles/DefaultStyles.xaml"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>

### Botón del menú hamburguesa

Este es botón maestro de nuestro patrón de navegación y debemos colocarlo en *AppShell.xaml* justo antes de cerrar el Grid que contiene toda la página:

    <ToggleButton x:Name="TogglePaneButton"
                      TabIndex="1"
                      Style="{StaticResource SplitViewTogglePaneButtonStyle}"
                      IsChecked="{Binding IsPaneOpen, ElementName=RootSplitView, Mode=TwoWay}"
                      Unchecked="TogglePaneButton_Checked"
                      AutomationProperties.Name="Menu"
                      ToolTipService.ToolTip="Menu" />

Hasta este momento, si lo hemos hecho todo bien, veremos en la vista del diseñador algo como esto:

![](https://40.media.tumblr.com/4f954fb23a2dbb26aaf071ef3a264186/tumblr_inline_o1javuFtXT1r7pczk_540.png)

### Frames

Justo después de cerrar el Splitview.Pane y antes de cerrar el control Splitview debemos agregar el frame que nos permite renderizar las páginas.

            <Frame x:Name="frame"
                   Navigating="OnNavigatingToPage"
                   Navigated="OnNavigatedToPage">
                <Frame.ContentTransitions>
                    <TransitionCollection>
                        <NavigationThemeTransition>
                            <NavigationThemeTransition.DefaultNavigationTransitionInfo>
                                <EntranceNavigationTransitionInfo/>
                            </NavigationThemeTransition.DefaultNavigationTransitionInfo>
                        </NavigationThemeTransition>
                    </TransitionCollection>
                </Frame.ContentTransitions>
            </Frame>

El código anterior es más que todo las transiciones y nombrar el frame como frame. Lo importante son las propiedades Navigating y Navigated.

### Navegación entre páginas


###  Darle tu diseño

Estamos listos para darle al *Deploy*. Hasta aquí tenemos solo lo básico, que es la construcción de la interfaz implementando el patrón de navegación. A partir de aquí puedes empezar a añadir tu toque personal o estilo propio.
 

Happy coding!
----------
