# Vistas

## Creación de vistas por código

Hasta ahora hemos visto como crear la interfaz visualmente con Xcode, pero todo lo que se puede hacer con dicha herramienta se puede hacer también de forma programática, ya que lo único que hace el entorno es crear objetos de la API de Cocoa Touch \(o definidos por nosotros\) y establecer algunas de sus propiedades.

### Ventanas

Las aplicaciones iOS tienen una única ventana. Podríamos crear la ventana principal desde nuestro código instanciando un objeto de tipo `UIWindow` e indicando el tamaño del marco de la ventana:

```objectivec
UIWindow  *window = [[UIWindow alloc]
        initWithFrame: [[UIScreen mainScreen] bounds]];
```

Lo habitual será inicializar la ventana principal con un marco del tamaño de la pantalla, tal como hemos hecho en el ejemplo anterior.

Para mostrar la ventana en pantalla debemos llamar a su método makeKeyAndVisible \(esto se hará normalmente en el método `application:didFinishLaunchingWithOptions:` del delegado de UIApplication\). Esto se hará tanto cuando la ventana ha sido cargada de un NIB como cuando la ventana se ha creado de forma programática.

```objectivec
[self.window makeKeyAndVisible];
```

Es posible que necesitemos acceder a esta ventana, por ejemplo para cambiar la vista que se muestra en ella. Hacer esto desde el delegado de UIApplication es sencillo, porque normalmente contamos con la propiedad `window` que hace referencia a la pantalla principal, pero acceder a ella desde otros lugares del código podría complicarse. Para facilitar el acceso a dicha ventana principal, podemos obtenerla a través del singleton `UIApplication`:

```objectivec
UIWindow *window  = [[UIApplication sharedApplication] keyWindow];
```

Si podemos acceder a una vista que se esté mostrando actualmente dentro de la ventana principal, también podemos obtener dicha ventana directamente a través de la vista:

```objectivec
UIWindow *window = [vista window];
```

### Vistas genéricas

También podemos construir una vista creando un objeto de tipo `UIView`. En el inicializador deberemos proporcionar el marco que ocupará dicha vista. El marco se define mediante el tipo `CGRect` \(se trata de una estructura, no de un objeto\). Podemos crear un nuevo rectángulo con la macro `CGRectMake`. Por ejemplo, podríamos inicializar una vista de la siguiente forma:

```objectivec
UIView *vista = [[UIView alloc]
        initWithFrame:CGRectMake(0,0,100,100)];
```

Como alternativa, podríamos obtener el marco a partir del tamaño de la pantalla o de la aplicación:

```objectivec
// Marco sin contar la barra de estado
CGRect marcoVista = [[UIScreen mainScreen] applicationFrame];
// Limites de la pantalla, con barra de estado incluida
CGRect marcoVentana = [[UIScreen mainScreen] bounds]
```

Normalmente utilizaremos el primero para las vistas que queramos que ocupen todo el espacio disponible en la pantalla, y el segundo para definir la ventana principal. Hemos de destacar que `UIWindow` es una subclase de `UIView`, por lo que todas las operaciones disponibles para las vistas están disponibles también para las ventanas. Las ventanas no son más que un tipo especial de vista.

Las vistas \(`UIView`\) también nos proporcionan una serie de métodos para consultar y modificar la jerarquía. El método básico que necesitaremos es `addSubview`, que nos permitirá añadir una subvista a una vista determinada \(o a la ventana principal\):

```objectivec
    //En el application delegate
    [self.window addSubview: vista];
```

Con esto haremos que la vista aparezca en la pantalla.

También podemos añadir una vista usando la propiedad `view` del _view controlller_ “activo”. Esta propiedad representa la vista “raíz” de la jerarquía de vistas manejadas por él.

```objectivec
    //En el view controller
    [self.view addSubview: vista];
```

Otra posibilidad es sobreescribir el `loadView` del controlador, que es el método encargado por defecto de crear las vistas, usualmente desde un NIB o _storyboard_, y en él crear las vistas necesarias, asignando la vista raíz a `self.view`.

Podemos eliminar una vista enviándole el mensaje `removeFromSuperview` \(se le envía a la vista hija que queremos eliminar\). Podemos también consultar la jerarquía con los siguientes métodos:

* `superview`: Nos da la vista padre de la vista destinataria del mensaje.
* `subviews`: Nos da la lista de subvistas de una vista dada.
* `isDescendantOfView:` Comprueba si una vista es descendiente de otra.

Como vemos, una vista tiene una lista de vistas hijas. Cada vista hija tiene un índice, que determinará el orden en el que se dibujan. El índice 0 es el más cercano al observador, y por lo tanto tapará a los índices superiores. Podemos insertar una vista en un índice determinado de la lista de subvistas con `insertSubview:atIndex:`.

Puede que tengamos una jerarquía compleja y necesitemos acceder desde el código a una determinada vista por ejemplo para inicializar su valor. Una opción es hacer un _outlet_ para cada vista que queramos modificar, pero esto podría sobrecargar nuestro objeto de _outlets_. También puede ser complejo y poco fiable el buscar la vista en la jerarquía. En estos casos, lo más sencillo es darle a las vistas que buscamos una etiqueta \(_tag_\) mediante la propiedad `Tag` del inspector de atributos \(debe ser un valor entero\), o asignando la propiedad `tag` de forma programática. Podremos localizar en nuestro código una vista a partir de su etiqueta mediante `viewWithTag`. Llamando a este método sobre una vista, buscará entre todas las subvistas aquella con la etiqueta indicada:

```objectivec
UIView *texto = [self.window viewWithTag: 1];
```

> La jerarquía de vistas de una pantalla determinada de nuestra aplicación puede llegar a ser muy compleja. Es por eso que en Xcode 6 se ha añadido una opción que nos permite mostrar un “despiece” visual en 3D de las vistas que componen la pantalla actual. Dicha opción está disponible en `Debug > View Debugging`.En modo texto podemos usar la propiedad `recursiveDescription` para [imprimir la descripción textual](https://developer.apple.com/library/ios/technotes/tn2239/_index.html#//apple_ref/doc/uid/DTS40010638-CH1-SUBSECTION34) de las vistas que contiene una vista dada.

## Propiedades de una vista

A continuación vamos a repasar las propiedades básicas de las vistas, que podremos modificar tanto desde Xcode como de forma programática.

### Disposición

Entre las propiedades más importantes en las vistas encontramos aquellas referentes a su disposición en pantalla. Hemos visto que tanto cuando creamos la vista con Xcode como cuando la inicializamos de forma programática hay que especificar el **marco** que ocupará la vista en la pantalla.

Cuando se crea de forma visual, el marco se puede definir pulsando con el ratón sobre los márgenes de la vista y arrastrando para así mover sus límites. En el código estos límites se especifican mediante el tipo `CGRect`, en el que se especifica posición `(x,y)` de inicio, y el ancho y el alto que ocupa la vista. Estos datos se especifican en el sistema de coordenadas de la supervista.

El sistema de coordenadas tiene su origen en la esquina superior izquierda. Las coordenadas no se dan en _pixels_, sino en **puntos**, una medida que nos permite independizarnos de la resolución en pixels de la pantalla. Las coordenadas en puntos son reales, no enteras.

En los modelos de iPhone/iPod Touch de 3.5’’ la resolución de pantalla en puntos es de 320x480 \(aun en los de _retina display_, que tiene un número de pixels mucho mayor\). Los dispositivos de 4’’ usan una resolución de 320x568 puntos. El iPhone 6 devuelve 375x667 y el 6 plus 736x414. Podéis consultar una [tabla muy completa](http://www.iosres.com) con muchos más datos

> Otros frameworks de iOS definen sistemas de coordenadas distintos. Los de gráficos \(Core Graphics y OpenGL ES\) ponen el origen en la esquina inferior izquierda con el eje Y apuntando hacia arriba.

Algunos ejemplos de cómo obtener la posición y dimensiones de una vista:

```objectivec
    // Limites en coordenadas locales
    // Su origen siempre es (0,0)
    CGRect area = [vista bounds];
    // Posición del centro de la vista en coordenadas de su supervista
    CGPoint centro = [vista center];
    // Marco en coordenadas de la supervista
    CGRect marco = [vista frame]
```

A partir de `bounds` y `center` podremos obtener `frame`

Sin embargo, en muchas ocasiones nos interesa que el tamaño no sea fijo sino que se adapte al área disponible. De esta forma nuestra interfaz podría adaptarse de forma sencilla a distintas orientaciones del dispositivo \(horizontal o vertical\) o a distintas resoluciones de la pantalla. Esto lo podemos conseguir mediante el uso de la tecnología de _autolayout_, que calcula de manera automática el _frame_ de cada vista basándose en un conjunto de restricciones. Veremos esta tecnología en sesiones posteriores.

### Transformaciones

Podemos también aplicar una transformación a las vistas, mediante su propiedad `transform`. Por defecto las vistas tendrán aplicada la transformación identidad `CGAffineTransformIdentity`.

La transformación se define mediante una matriz de transformación 2D de dimensión 3x3. Podemos crear transformaciones de forma sencilla con macros como `CGAffineTransformMakeRotation` o `CGAffineTransformMakeScale`.

> Si nuestra vista tiene aplicada una transformación diferente a la identidad, su propiedad `frame` no será significativa. En este caso sólo deberemos utilizar `center` y `bounds`.

### Otras propiedades

En las vistas encontramos otras propiedades que nos permiten determinar su color o su opacidad. En primer lugar tenemos `backgroundColor`, con la que podemos fijar el color de fondo de una vista. En el inspector de atributos \(sección `View`\) podemos verlo como propiedad `Background`. El color de fondo puede ser transparente, o puede utilizarse como fondo un determinado patrón basado en una imagen.

De forma programática, el color se especifica mediante un objeto de clase `UIColor`. En esta clase podemos crear un color personalizado a partir de sus componentes \(rojo, verde, azul, alpha\), o a partir de un patrón.

Por otro lado, también podemos hacer que una vista tenga un cierto grado de transparencia, o esté oculta. A diferencia de `backgroundColor`, que sólo afectaba al fondo de la vista, con la propiedad `alpha`, de tipo `CGFloat`, podemos controlar el nivel de transparencia de la vista completa con todo su contenido y sus subvistas. Si una vista no tiene transparencia, podemos poner su propiedad `opaque` a `YES` para así optimizar la forma de dibujarla. Esta propiedad sólo debe establecerse a `YES` si la vista llena todo su contendo y no deja ver nada del fondo. De no ser así, el resultado es impredecible. Debemos llevar cuidado con esto, ya que por defecto dicha propiedad es `YES`.

Por último, también podemos ocultar una vista con la propiedad `hidden`. Cuando hagamos que una vista se oculte, aunque seguirá ocupando su correspondiente espacio en pantalla, no será visible ni recibirá eventos.

## Controles básicos de interfaz de usuario

A lo largo de los ejemplos que hemos ido haciendo en las sesiones anteriores ya hemos probado bastantes de los controles básicos de interfaz de usuario que nos proporciona iOS: botones, etiquetas, imágenes, campos de texto,… Vamos a ver aquí algunas de las características de los controles, aunque solo vamos a dar unas pinceladas, ya que una descripción exhaustiva de cada propiedad sería imposible y tediosa. Se remite al lector a la documentación de Apple, en concreto el [UIKit User Interface Catalog](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIKitUICatalog.pdf), excelente y bastante exhaustivo.

> Aunque aquí hablemos de controles indistintamente para referirnos a las etiquetas, botones, … en realidad este término tiene un significado más preciso en iOS. La clase `UIControl` es de la que heredan los controles más “interactivos” como los botones, mientras que las etiquetas lo hacen de `UIView` \(no obstante todos los`UIControl` son también vistas ya que a su vez esta clase hereda de `UIView`\).

### Campos de texto

Un campo de texto nos proporciona un espacio donde el usuario puede introducir y editar texto. Se define en la clase `UITextField`, y pertenece a un grupo de vistas denominados controles, junto a otros componentes como por ejemplo los botones. Esto es así porque permiten al usuario interactuar con la aplicación. No heredan directamente de `UIView`, sino de su subclase `UIControl`, que incorpora los métodos para tratar eventos de la interfaz mediante el patrón _target-action_ como hemos visto anteriormente.

Sus propiedades se pueden encontrar en la sección `Text Field` del inspector de atributos. Podremos especificar un texto por defecto \(`Text`\), o bien un texto a mostrar sombreado en caso de que el usuario no haya introducido nada \(`Placeholder Text`\). Esto será útil por ejemplo para dar una pista al usuario sobre lo que debe introducir en dicho campo.

Si nos fijamos en el inspector de conexiones del campos de texto, veremos la lista de eventos que podemos conectar a nuestra acciones. Esta lista de eventos es común para cualquier control. En el caso de un campo de texto por ejemplo nos puede interesar el evento `Value Changed`.

### Botones

Al igual que los campos de texto, los botones son otro tipo de control \(heredan de `UIControl`\). Se definen en la clase `UIButton`, que puede ser inicializada de la misma forma que el resto de vistas.

Si nos fijamos en el inspector de atributos de un botón \(en la sección `Button`\), vemos que podemos elegir el tipo de botón \(atributo `Type`\). Podemos seleccionar una serie de estilos prefedinidos para los botones, o bien darle un estilo propio \(`Custom`\).

El texto que aparece en el botón se especifica en la propiedad `Title`, y podemos configurar también su color, sombreado, o añadir una imagen como icono.

En el inspector de conexiones, el evento que utilizaremos más comúnmente en los botones es `Touch Up Inside`, que se producirá cuando levantemos el dedo tras pulsar dentro del botón. Este será el momento en el que se realizará la acción asociada al botón.

### Alertas

Se usan para informar al usuario de eventos importantes. Pueden simplemente informar de algo o además pedir al usuario que elija uno entre varios cursos de acción. No se crean gráficamente en Xcode sino por código

```objectivec
UIAlertView *alert = [[UIAlertView alloc]
                              initWithTitle:@"Saludo"
                              message:@"Hola usuario"
                              delegate:self
                              cancelButtonTitle:@"OK"
                              otherButtonTitles: nil];
[alert show]
```

Como mínimo tendremos un botón \(`cancelButtonTitle`\), para que la alerta desaparezca.

> Aunque se llame `cancelButtonTitle`no quiere decir que sea el botón de cancelar, de hecho habitualmente será “Aceptar” u “OK”.

Podemos tener más botones, como una lista terminada en `nil`. No obstante, las “iOS Human Interface Guidelines” [recomiendan como máximo dos](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/Alerts.html#//apple_ref/doc/uid/TP40006556-CH14-SW2), y en caso que necesitemos más usar un _“Action Sheet”_.

El `delegate` es el objeto que se ocupará de qué hacer cuando la alerta se hace desaparecer o qué curso tomar en función del botón elegido. Debe implementar el protocolo `UIAlertViewDelegate`. Por ejemplo `alertView:didDismissWithButtonIndex:` se envía cuando desaparece la alerta y nos dice qué número de botón se ha pulsado \(0 el `cancelButtonTitle` y el resto por el orden de definición comenzando en 1\).

```objectivec
    -(void) alertView:(UIAlertView *)alertView didDismissWithButtonIndex:(NSInteger)buttonIndex {
        NSLog(@"Se ha pulsado el boton nº %d", buttonIndex);
    }
```

### Action Sheets

Dan a los usuarios la posibilidad de ofrecer datos adicionales ante una acción a realizar. Por ejemplo confirmar o cancelar la información.

> Las aparición de una alerta suelen ser inesperada para el usuario. Sin embargo el usuario sabe cuándo es probable que aparezca un Action Sheet. Por ejemplo cuando realiza una acción destructiva como eliminar una foto.

No se crean gráficamente sino por código. El API es prácticamente idéntico al de las alertas.

> Dese iOS 8, `UIAlertController` uniformiza el interfaz de `UIAlertView` y`UIActionSheet`. Ver por ejemplo [http://nshipster.com/uialertcontroller/](http://nshipster.com/uialertcontroller/)

## Teclado en pantalla

Cuando un campo de texto adquiere el foco porque el usuario hace _tap_ sobre él, automáticamente aparece el teclado software _on screen_. El problema es que por defecto no desaparece salvo que escribamos algo de código.

Si queremos que desaparezca cuando pulsamos sobre la tecla de “Aceptar” tenemos que escribir un _action_ que responda al evento `Did end on exit` del campo de texto. En teoría dentro de este _action_ debemos hacer que el campo deje de ser el _first responder_ para que el teclado deje de mostrarse

```objectivec
- (IBAction)pulsadoIntro:(id)sender {
    [sender resignFirstResponder];
}
``

> Aunque en muchas fuentes aparece el método del `resignFirstResponder` como solución, curiosamente *parece bastar con tener un action que responda al `Did end on exit`, aunque no haga nada* para ocultar el teclado.*

Por desgracia, no todos los tipos de teclado en pantalla tienen un botón de “intro” y por tanto no disparan el evento `Did end on exit`, por ejemplo el teclado numérico no lo hace. En ese caso hay que usar una solución algo más rebuscada. Por ejemplo, es muy típico que al pulsar sobre cualquier parte de la pantalla que no sea el campo el teclado se oculte. Esto podemos conseguirlo detectando el evento de toque sobre la vista. Si recordamos el procesamiento de eventos básico, si estos eventos no se procesan antes llegan hasta el *view controller*, por lo que en él podríamos hacer:

```objectivec
-(void) touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event {
NSLog(@"touch en la pantalla!!");
  //Suponiendo que al campo de texto le hayamos dado el tag=1
  [[self.view viewWithTag:1] resignFirstResponder];
}
```

## Ejercicios de vistas

### Creación de ventana/controlador/vista de modo manual

Ya hemos visto que por defecto las aplicaciones de Xcode usan el _storyboard_. Este _storyboard_ lo carga automáticamente el Application Delegate, haciendo además las siguientes tareas:

* Crear una ventana con el tamaño de la pantalla
* Cargar el controlador asociado a la pantalla inicial, y asignarle a su propiedad `view` las subvistas de esta pantalla
* Hacer visible la ventana, que ahora contiene al controlador

Vamos a hacer lo mismo pero de modo manual, para comprender un poco mejor todo el proceso

* Cread un nuevo proyecto en la carpeta `sesion2` de las plantillas llamado `VistaManual`
* Lo primero es desactivar el _storyboard_. Para ello en el navegador de Xcode hacemos clic sobre el icono del proyecto. Aparecerán los distintos apartados de configuración. Elegimos `info` y eliminamos una propiedad que pone “Main Storyboard name” pulsando sobre el botón de ‘-‘. Ahora el proyecto ya no usa el storyboard
* En el `applicationDidFinishLaunching:withOptions` del AppDelegate tenemos que hacer lo siguiente

```objectivec
self.window = [[UIWindow alloc]
                      initWithFrame: [[UIScreen mainScreen] bounds]];
self.window.rootViewController = [[ViewController alloc] init];
UIButton *boton = [[UIButton alloc] init];
[self.window makeKeyAndVisible];
```

* No obstante, en pantalla no se verá nada, ya que la vista del `ViewController` está vacía. Probad a hacer que aparezca algo para comprobar que todo funciona:
  * Tened en cuenta que la vista es `self.window.rootViewController.view`
  * cambiad el color, propiedad `backgroundColor` de la vista. Para especificar color lo más sencillo es usar una serie de métodos de clase \(estático\) de `UIColor`: `[UIcolor greenColor]`, `[UIcolor redColor]`,…
  * cread un botón \(instanciad un objeto de la clase UIButton\*\), especificando el título \(método `setLabel:forState`\) y unas dimensiones

```objectivec
[boton setTitle:@"Soy un botón manual" forState:UIControlStateNormal];
[boton setFrame:CGRectMake(100,100,100,40)];
```

## Uso de controles de interfaz de usuario

En un nuevo proyecto, llamado `Controles`, probad el uso de algunos de los controles de interfaz de usuario de iOS que todavía no hemos visto. Para cualquier duda, consultad la documentación de Xcode o la [referencia de controles de UIKit](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIControl.html)

* Cread un _slider_ cuya escala vaya desde 0 hasta 100, y un _label_ al lado. Al mover el slider, el _label_ debería cambiar para reflejar su valor actual.
  * Cread un _outlet_ para poder manipular la etiqueta
  * Crear un “action” con `ctrl+arrastrar` desde el _slider_ al código de `ViewController.m`. En este _action_ podéis obtener el valor actual del _slider_ \(propiedad `value` del parámetro `sender` y fijarlo como texto de la etiqueta. Tened en cuenta que el valor del slider es un número y el texto es un `NSString*`, podéis usar `[NSString stringWithFormat]` para convertir como ya hemos hecho varias veces.
* cread un botón que cuando se pulse aparezca un `UIActionSheet` dando varias opciones con texto inventado por vosotros. Detectad qué opción se ha mostrado e imprimid un mensaje con `NSLog` indicándolo. Tendréis que consultar la [documentación de `UIActionSheet`](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIActionSheet.html#//apple_ref/doc/uid/TP40012857-UIActionSheet-SW1). Mirad el apartado “Behavior of action sheets”. Básicamente la idea es que al crear el _action sheet_ se especifica qué objeto es su “delegate” \(lo más sencillo es que sea el controller\) y este tiene que
  * Especificar que implementa el protocolo `UIActionSheetDelegate`
  * Implementar el método `actionSheet:clickedButtonAtIndex:` que nos dirá el número de opción elegida, comenzando en 0.

