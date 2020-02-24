---
$title: Use AMP como fuente de datos para su PWA
$order: 1
description: Si ha invertido en AMP pero aún no ha creado una aplicación web progresiva, sus páginas AMP pueden simplificar drásticamente el desarrollo de su aplicación web progresiva.
formats:
- sitios web
author: pbakaus
---

Si ha invertido en AMP pero aún no ha creado una aplicación web progresiva, sus páginas AMP pueden simplificar drásticamente el desarrollo de su aplicación web progresiva. En esta guía aprenderá cómo consumir AMP dentro de su aplicación web progresiva y usar sus páginas AMP existentes como fuente de datos.

## De JSON a AMP

En el escenario más común, una aplicación web progresiva es una aplicación de página única que se conecta a una API JSON a través de Ajax. Esta API JSON luego devuelve conjuntos de datos para impulsar la navegación y el contenido real para representar los artículos.

Luego procedería y convertiría el contenido en bruto en HTML utilizable y lo representaría en el cliente. Este proceso es costoso y a menudo difícil de mantener. En cambio, puede reutilizar sus páginas AMP ya existentes como fuente de contenido. Lo mejor de todo es que AMP lo hace trivial en tan solo unas pocas líneas de código.

## Incluya "Shadow AMP" en su aplicación web progresiva

El primer paso es incluir una versión especial de AMP que llamamos "Shadow AMP" en su aplicación web progresiva. Sí, es cierto: carga la biblioteca AMP en la página de nivel superior, pero en realidad no controlará el contenido de nivel superior. Solo "amplificará" las partes de nuestra página que usted le indique.

Incluya Shadow AMP en el encabezado de su página, así:

[código fuente: html]

[/código fuente]

<!-- Asynchronously load the AMP-with-Shadow-DOM runtime library. -->

<script async="" src="https://cdn.ampproject.org/shadow-v0.js"></script>


Le recomendamos que cargue la biblioteca Shadow AMP con el atributo `async` en su lugar. Eso significa, sin embargo, que necesita usar un cierto enfoque para comprender cuándo la biblioteca está completamente cargada y lista para usarse.

### ¿Cómo saber cuándo la API de Shadow AMP está lista para usar?

La señal correcta a observar es la disponibilidad de la variable `AMP` global, y Shadow AMP utiliza un " [enfoque de carga de función asincrónica](http://mrcoles.com/blog/google-analytics-asynchronous-tracking-how-it-work/) " para ayudar con eso. Considera este código:

[código fuente: javascript] (window.AMP = window.AMP || []). push (function (AMP) {// AMP ahora está disponible.}); [/código fuente]

Este código funcionará, y cualquier cantidad de devoluciones de llamada agregadas de esta manera se disparará cuando AMP esté disponible, pero ¿por qué?

Este código se traduce a:

Funciona porque la biblioteca Shadow AMP, después de la carga real, se dará cuenta de que ya hay una serie de devoluciones de llamada en `window.AMP` , luego procesará toda la cola. Si luego ejecuta la misma función nuevamente, seguirá funcionando, ya que Shadow AMP reemplaza `window.AMP` por sí mismo y un método de `push` personalizado que simplemente activa la devolución de llamada de inmediato.

1. "Si window.AMP no existe, cree una matriz vacía para tomar su posición"
2. "luego inserte una función de devolución de llamada en la matriz que debe ejecutarse cuando AMP esté listo"

[tip type = "tip"] **SUGERENCIA:** para que el ejemplo de código anterior sea práctico, recomendamos que lo envuelva en una Promesa, luego use siempre dicha Promesa antes de trabajar con la API de AMP. Mire nuestro [código de demostración React](https://github.com/ampproject/amp-publisher-sample/blob/master/amp-pwa/src/components/amp-document/amp-document.js#L20) para ver un ejemplo. [/propina]

Aún necesitará implementar este paso manualmente. Después de todo, depende de usted cómo presente los enlaces al contenido en su concepto de navegación. ¿Un número de listas? ¿Un montón de cartas?

## Maneje la navegación en su aplicación web progresiva

En un escenario común, obtendría algunos JSON que devuelven URL ordenadas con algunos metadatos. Al final, debe terminar con una función de devolución de llamada que se activa cuando el usuario hace clic en uno de los enlaces, y dicha devolución de llamada debe incluir la URL de la página AMP solicitada. Si tienes eso, estás listo para el paso final.

Finalmente, cuando desee mostrar contenido después de una acción del usuario, es hora de buscar el documento AMP relevante y dejar que Shadow AMP se haga cargo. Primero, implemente una función para recuperar la página, similar a esta:

## Use la API Shadow AMP para representar una página en línea

[código fuente: javascript] función fetchDocument (url) {

// lamentablemente fetch () no admite la recuperación de documentos, // por lo tanto, tenemos que recurrir a la buena XMLHttpRequest. var xhr = new XMLHttpRequest ();

return new Promise (función (resolver, rechazar) {xhr.open ('GET', url, true); xhr.responseType = 'document'; xhr.setRequestHeader ('Accept', 'text / html'); xhr.onload = function () {// .responseXML contiene un objeto de documento listo para usar resolver (xhr.responseXML);}; xhr.send ();}); } [/código fuente]

[tip type = "important"] **IMPORTANTE:** para simplificar el ejemplo de código anterior, omitimos el manejo de errores. Siempre debe asegurarse de detectar y manejar los errores con gracia. [/propina]

Ahora que tenemos nuestro objeto `Document` listo para usar, es hora de dejar que AMP se haga cargo y lo represente. Obtenga una referencia al elemento DOM que sirve como contenedor para el documento AMP, luego llame a `AMP.attachShadowDoc()` , así:

[código fuente: javascript] // Esto puede ser cualquier elemento DOM var container = document.getElementById ('container');

// La página AMP que desea mostrar var url = "https: //my-domain/amp/an-article.html";

// Use nuestro método fetchDocument para obtener el documento fetchDocument (url) .then (function (doc) {// Deje que AMP tome el control y renderice la página var ampedDoc = AMP.attachShadowDoc (container, doc, url);}); [/código fuente]

[tip type = "tip"] **SUGERENCIA:** antes de entregar el documento a AMP, es el momento perfecto para eliminar elementos de la página que tengan sentido al mostrar la página AMP de forma independiente, pero no en modo incrustado: por ejemplo, pies de página y encabezados . [/propina]

¡Y eso es! Su página AMP se representa como un elemento secundario de su aplicación web progresiva general.

Es probable que su usuario navegue de AMP a AMP dentro de su aplicación web progresiva. Al descartar la página AMP renderizada anterior, asegúrese siempre de informarle a AMP al respecto, así:

## Limpia después de ti

[código fuente: javascript] // ampedDoc es la referencia devuelta por AMP.attachShadowDoc ampedDoc.close (); [/código fuente]

Esto le indicará a AMP que ya no está usando este documento y liberará memoria y sobrecarga de la CPU.

[video src = "/ static / img / docs / pwamp_react_demo.mp4" width = "620" height = "1100" loop = "true", controls = "true"]

## Véalo en acción

Puede ver el patrón "AMP in PWA" en acción en la [muestra React](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa) que hemos creado. Demuestra transiciones suaves durante la navegación y viene con un componente React simple que envuelve los pasos anteriores. Es lo mejor de ambos mundos: JavaScript flexible y personalizado en la aplicación web progresiva y AMP para impulsar el contenido.

También puede ver una muestra de PWA y AMP usando Polymer framework. La muestra usa [amp-viewer](https://github.com/PolymerLabs/amp-viewer/) para incrustar páginas AMP.

- Obtenga el código fuente aquí: [https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa)
- Utilice el componente React independiente a través de npm: [https://www.npmjs.com/package/react-amp-document](https://www.npmjs.com/package/react-amp-document)
- Véalo en acción aquí: [https://choumx.github.io/amp-pwa/](https://choumx.github.io/amp-pwa/) (mejor en su teléfono o emulación móvil)

También puede ver una muestra de PWA y AMP usando Polymer framework. La muestra usa [amp-viewer](https://github.com/PolymerLabs/amp-viewer/) para incrustar páginas AMP.

- Obtenga el código aquí: [https://github.com/Polymer/news/tree/amp](https://github.com/Polymer/news/tree/amp)
- Véalo en acción aquí: [https://polymer-news-amp.appspot.com/](https://polymer-news-amp.appspot.com/)
