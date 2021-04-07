# Flag 0 Unauthorized access
Tenemos en la página principal 3 opciones disponibles:

![image](https://user-images.githubusercontent.com/43856126/113806875-aa95f980-9752-11eb-966b-728d7f893806.png)

### Testing
Aquí lo que tenemos es un par de headings con un texto y una opción para editar esa página.

![image](https://user-images.githubusercontent.com/43856126/113807043-f0eb5880-9752-11eb-8f86-bf1112a63290.png)

Lo que más llama la atención es que podemos ver lo que bien podría ser el ID de la página que se esta viendo

![image](https://user-images.githubusercontent.com/43856126/113807216-3c056b80-9753-11eb-908c-3935021def9e.png)

y efectivamente podemos ver otras páginas directamente simplemente al ir incrementando el id, esto es curioso hasta que llegamos al número 5, donde nos devuelve un forbidden:

![image](https://user-images.githubusercontent.com/43856126/113807433-a6b6a700-9753-11eb-8b93-6937bab5de69.png)

Regresando a testing, ahora vemos lo que podemos hacer con la edición de la página, y podemos ver que aquí podemos editar el contenido de la página que teniamos por id, y aquí también podemos ver el id

![image](https://user-images.githubusercontent.com/43856126/113807578-eed5c980-9753-11eb-935a-802415dd8b1d.png)

Así que ahora directamente probamos lo que sería el número 5, que nos había saltado antes para ver si ahora si podemos verlo.

![image](https://user-images.githubusercontent.com/43856126/113807664-188ef080-9754-11eb-9375-4b30578200e7.png)

Y tenemos la primera flag.


# Flag 1 XSS

### Edit this page
Ahora regresando al principio en el apartado de crear una nueva página podemos ver que solo nos pide el titulo y el contenido que llevará la página que queremos crear:

![image](https://user-images.githubusercontent.com/43856126/113808808-53922380-9756-11eb-9cac-a7f19ae3b325.png)

Y con una pequeña prueba podemos ver que escribe directamente lo que pasamos a los campos que nos pedía:

![image](https://user-images.githubusercontent.com/43856126/113808874-7b818700-9756-11eb-814e-a9a5d686807c.png)

Lo que se me ocurre en este punto es probar algo de XSS...
Así que lo primero que probamos son etiquetas HMTL para ver si encontramos algo...

![image](https://user-images.githubusercontent.com/43856126/113808965-abc92580-9756-11eb-965e-444065539200.png)

![image](https://user-images.githubusercontent.com/43856126/113809004-c26f7c80-9756-11eb-91a2-d950ea18f374.png)

Podemos ver que a comparación de un texto normal, sí afecta directamente al texto poniendo etiquetas HTML, así que ahora probamos con un alert:

![image](https://user-images.githubusercontent.com/43856126/113809196-24c87d00-9757-11eb-945b-8b03ede4b6cb.png)

Pero esto no afeta a la salida... Así que el mismo procedimiento ahora lo probamos en el campo para especificar el título, y podemos ver lo siguiente:

![image](https://user-images.githubusercontent.com/43856126/113809274-47f32c80-9757-11eb-9c0d-6c411efc57db.png)

![image](https://user-images.githubusercontent.com/43856126/113809290-53465800-9757-11eb-9122-9a79fc311b98.png)

![image](https://user-images.githubusercontent.com/43856126/113809338-68bb8200-9757-11eb-8a5e-56ce64c17419.png)

Y esto tuvo efecto y obtuvimos otra flag.

# Flag 2 XSS

### Markdown Test
Inspeccionando la segunda página que ya estaba creada podemos ver que existe un botón:

![image](https://user-images.githubusercontent.com/43856126/113809649-1d55a380-9758-11eb-8d23-8a87bb27a001.png)

En este apartado vemos que se pueden añadir botones, así que probamos nuevamente algo pero con el evento a button que le podemos agregar que es un onclick

![image](https://user-images.githubusercontent.com/43856126/113810210-3b6fd380-9759-11eb-8d7c-8a01aab1b582.png)

Después al hacer clic en el botón podemos ver que se efectuo el payload pero no podemos ver la bandera

![image](https://user-images.githubusercontent.com/43856126/113810440-cea90900-9759-11eb-9bc7-08471c4c6e0d.png)

Pero si revisamos el código podemos ver que la bandera fue incrustada en el botón:

![image](https://user-images.githubusercontent.com/43856126/113810512-f5673f80-9759-11eb-9335-990baef286e0.png)

# Flag 3 SQL injection
En esta bandera no tenía mucha idea de como obtenerla, pero al revisar las pistas me percate de algo:

![image](https://user-images.githubusercontent.com/43856126/113810630-33646380-975a-11eb-9673-ec054d9963aa.png)

No había probado por hacer SQLi, así que después de probar de diferentes maneras llegue hasta la edición de las páginas y al hacer lo siguiente:

![image](https://user-images.githubusercontent.com/43856126/113810714-57c04000-975a-11eb-91e0-34ef8885bd3f.png)

Simplemente añadiendo una comilla simple para ver si rompía de alguna manera la consulta si es que fuese algo de SQL.

![image](https://user-images.githubusercontent.com/43856126/113810776-7a525900-975a-11eb-9760-cab4a59637dd.png)

Y así obtuvimos la siguiente y última flag.
