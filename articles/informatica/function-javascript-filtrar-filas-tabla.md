# Función en Javascrip para filtrar filas de una tabla

Como en la [entrada anterior de Javascrip para crear solapas](/archivo/2012/informatica/funcion-en-javascript-para-crear-solapas-tabs.html), para filtrar las filas de una tabla también existen multitud de ejemplos en la Web, [incluidos plug-ins de jQuery](https://github.com/riklomas/quicksearch).

Pero como a mi me gusta experimentar para aprender, aquí está mi versión del filtrado de filas de una tabla. De momento no es plug-in de jQuery pero, como la vez anterior, no creo que sea demasiado difícil adaptarlo. Y, por supuesto que tiene sus fallos, pero se irán subsanando con el tiempo.

```javascript
function tablefilter(table_selector, input_selector, search_level, colspan) {
    var table = $(table_selector);
    if(table.length == 0)
        return;

    var input = $(input_selector);
    if(input.length == 0)
        return;

    if(search_level == "undefined" || search_level < 1)
        search_level = 3;

    if(colspan == "undefined" || colspan < 0)
        colspan = 2;

    $(input).val("Buscar…");

    $(input).focus(function() {
        if($(this).val() == "Buscar…") {
            $(this).val("");
        }
        $(this).select();
    });

    $(input).blur(function() {
        if($(this).val() == "") {
            $(this).val("Buscar…");
        }
    });

    $(input).keyup(function() {
        if($(this).val().length >= search_level) {
            // Ocultamos las filas que no contienen el contenido del edit.
            $(table).find("tbody tr").not(":contains(\"" + $(this).val() + "\")").hide();
            
            // Si no hay resultados, lo indicamos.
            if($(table).find("tbody tr:visible").length == 0) {
                $(table).find("tbody:first").append('<tr id="noresults" class="aligncenter"><td colspan="' + colspan + '">Lo siento pero no hay resultados para la búsqueda indicada.</td></tr>');
            }
        } else {
            // Borramos la fila de que no hay resultados.
            $(table).find("tbody tr#noresults").remove();
            
            // Mostramos todas las filas.
            $(table).find("tbody tr").show();
        }
    });
}
```

Para hacerlo funcionar basta con llamar a la función con dos, tres o cuatro parámetros (a elegir; si no se pasan se usan los predeterminados). El primer parámetro es el selector de la tabla a aplicar; el segundo es el `input` que se usará para el filtro; el tercero es el número de caracteres mínimos para comenzar la búsqueda (si no se especifica se usan tres); y el cuarto es el número de columnas que se debe extender la fila de información. Un ejemplo de llamada sería:

```javascript
$(document).ready(function() {
  tablefilter("table#mi_tabla", "table thead tr input#filtro", 2, 3);
});
```

Internamente esta función usa el método `contains()` de jQuery para buscar. Este método distingue entre mayúsculas y minúsculas, así que, para hacer mejores búsquedas, se puede indicar que este método no distinga de la siguiente forma:

```javascript
jQuery.expr[':'].contains = function(a, i, m) { 
  return jQuery(a).text().toUpperCase().indexOf(m[3].toUpperCase()) >= 0; 
};
```

En caso de que no funcione correctamente, se debe añadir exactamente lo mismo otra vez pero cambiando "`.contains =`" por "`.Contains =`".

Y, cómo no, podéis [ver un ejemplo del funcionamiento](/demos/table-filter/table-filter-example.html) de la misma así como [descargarlo](/download/table-filter-example.html).
