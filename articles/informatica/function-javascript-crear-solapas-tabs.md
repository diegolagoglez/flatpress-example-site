# Función en Javascript para crear solapas (tabs)

Existen multitud de *[snippets](http://es.wikipedia.org/wiki/Snippet)* y [*plug-ins* de jQuery](https://www.google.es/search?ie=UTF-8&q=jquery+tabs) para hacer solapas con diferentes partes de una misma página, pero ninguno de ellos con las características que he implementado en esta función (que —todavía— no es un *plug-in* de [jQuery](http://jquery.com/)). La principal característica que quería conseguir era la simplicidad.

La mayoría de estos *plug-ins* o funciones requieren que se haga un poco de marcado en [HTML](http://es.wikipedia.org/wiki/HTML) para delimitar tanto el conjunto de solapas como cada una de ellas, con lo que, al menos, habrá que marcar un `<div>` general y un `<div>` por cada solapa.

Con esta función lo que se pretende, como he comentado antes, es que ese marcado adicional sea innecesario usando sólo lo que ya existe dentro de la página para crear las solapas.

Es por esto que esta función lo que hace es, dentro de una caja (o del cuerpo completo del HTML), convertir en solapa todo el contenido que hay entre diferentes encabezados del mismo nivel (`<h1>..<h6>`), es decir, si existe una página bien formada con sus correspondientes encabezados, mediante esta función es trivial mostrarla como solapas.

El código de la función —y, repito, no es un *plug-in* para jQuery pero no sería demasiado difícil adaptarla— es el siguiente:

```javascript
function Tabs(element, tabElement) {
    // Si no existe la caja de donde coger los elementos, nos salimos.
    if($(element).length == 0)
        return;

    // Si ya las hemos creado, no lo volvemos a hacer. De momento sólo se soportan unas tabs por página.
    if($(element).find("ul#tabs").length > 0)
        return;

    // Creamos una lista con los elementos de las solapas (serán los botones).
    var tabList = $("<ul>", { id: "tabs" });

    // Cogemos lo que hay entre cada 'tabElement' y lo metemos en un div
    var counter = 1;
    $(element).find(tabElement).each(function() {

        // Creamos una solapa para cada elemento y asignamos el evento "click".
        $(tabList).append($("<li>", { text: $(this).html(), id: "tab_" + counter }).click(function() {
            // Cuando hacemos click en la solapa, quitamos la selección.
            $(element).find("ul#tabs li").removeClass("selected");
            $(this).addClass("selected");
            var id = $(this).attr("id").substring(4);   // de "tab_1" nos quedamos con "1".
            $(element).find("div.tab_content").removeClass("selected");
            $(element).find("div.tab_content#tab_content_" + id).addClass("selected");

            // Reemplazamos la URL para que se refleje el cambio y, si se recarga la página, se recargue en el sitio correcto.
            var newurl = window.location.pathname;
            newurl = newurl.substring(newurl.indexOf("#") + 1) + "#tab_" + id;
            window.location.replace(newurl);

            // Volvemos a la parte de arriba de la página para que no queden ocultas las solapas por culpa de Drupal y su barra de administrador.
            // Otra opción podría ser volver un poco, en lugar de al principio. Por ejemplo: $(document).scrollTop($(document).scrollTop() - 10);
            $(document).scrollTop(0);
        }));

        // Metemos lo que hay entre los 'tabElement' en un div.
        $(this).nextUntil(tabElement).wrapAll("<div class='tab_content' id='tab_content_" + counter + "'></div>");

        // Borramos el elemento.
        $(this).remove();

        // Estos divs se ocultan con CSS.
        counter++;
    });

    // Añadimos la lista al div principal (pasado como parámetro).
    $(element).prepend(tabList);

    // Obtenemos el fragment (o hash) de la URL.
    var hash = window.location.hash;

    // Obtenemos el número de tab que sabemos está detrás de "_".
    hash = hash.substring(hash.indexOf("_") + 1);

    // Mostramos la primera solapa con el primer contenido (div) si no hay fragment. Si hay fragment
    // mostramos el que toca según el mismo (el fragment es #tab_x donde x es el número de tab [1..n]).
    if(hash != "" && $(tabList).find("li#tab_" + hash).length != 0) {
        $(tabList).find("li#tab_" + hash).addClass("selected");
        $(element).find("div.tab_content#tab_content_" + hash).addClass("selected");
    } else {
        $(tabList).find("li:first").addClass("selected");
        $(element).find("div.tab_content:first").addClass("selected");
    }
}
```

Como las solapas se crean con una lista (`<ul>`), se debe poner un poco de [CSS](http://es.wikipedia.org/wiki/CSS) para que se adapte, igual que se debe poner CSS para ocultar las solapas que no están visibles (en lugar de hacerlo con el propio Javascript modificando los atributos directamente, prefiero hacerlo con CSS añadiendo y/o quitando clases). El CSS básico (con un poco de estilo gráfico también) es el siguiente:

```javascript
ul#tabs {
    margin: 10px 0;
    padding: 0;
    border-bottom: 1px solid silver;
    line-height: 29px;
}

ul#tabs li {
    display: inline-block;
    margin: 0 0 -1px -1px;
    padding: 0 10px;
    cursor: pointer;
    border: 1px solid silver;
    background: #DDDDDD;
    color: gray;
}

ul#tabs li.selected {
    background: white;
    color: #444444;
}

ul#tabs li:hover {
    background: #EEEEEE;
}

div.tab_content {
    display: none;
}

div.tab_content.selected {
    display: block;
}
```

Podéis [ver un ejemplo de su funcionamiento](/demos/tabs/tabs-example.html) y también podéis [descargarlo](/download/tabs-example.html) para vuestro uso y disfrute (sin olvidarse de [la licencia](/acerca-de/licencia), claro).

**Actualización 2012-06-07**: [Antonio me comenta](/archivo/2012/informatica/funcion-en-javascript-para-crear-solapas-tabs.html/comment-page-1#comment-23761) que en IE9 (cómo no) no se ven bien las solapas. La lista que hará las solapas, al tener el CSS `inline-block` para que se muestren en una sola fila pero no se corten las solapas por la mitad, no se interpreta bien, por lo que salen en una lista normal. La solución sería que floten dichas solapas a la izquierda (anulando todo lo anterior del `inline-block`), más o menos así:

```javascript
ul#tabs { overflow: auto; }
ul#tabs li { float: left; }
```
