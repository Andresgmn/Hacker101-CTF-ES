Tenemos una página en donde podemos crear nuestros propios post, públicos o privados.

![image](https://user-images.githubusercontent.com/43856126/115179232-6267c700-a0c2-11eb-856d-78e37994b286.png)

En sign in traté de hacer un pooco de SQL injection sin éxito. Así que nos vamos por el lado de Sign up, donde nos podemos crear una cuenta.

Al crear la cuenta y acceder a ella vemos secciones nuevas en la barra de herramientas y en el contenido de la página principal en el cual podemos observar algunos post públicos:

![image](https://user-images.githubusercontent.com/43856126/115179390-c12d4080-a0c2-11eb-8dd4-34d39ac93378.png)

![image](https://user-images.githubusercontent.com/43856126/115179422-cdb19900-a0c2-11eb-8eef-5f21d64d4b1f.png)

# Flag 0

El primer apartado que me fuí es a My profile, en donde encontramos el nombre de usuario y el conteo de post realizados:

![image](https://user-images.githubusercontent.com/43856126/115179544-15d0bb80-a0c3-11eb-831d-6fa470b3eab9.png)

Lo interesante de esta parte es que en la URL se puede ver un tipo de identificador, en este caso es una simple letra d:

![image](https://user-images.githubusercontent.com/43856126/115179635-4a447780-a0c3-11eb-9421-c61196b0bfd4.png)

Así que ahora a probar diferentes tipos de letra, se puede cambiar el panel y parece que nos dejaría ver el panel de perfil de otros usuarios, comenzando por la letra a e ir probando con las demás letras, podemos encontrar otro usuario:

![image](https://user-images.githubusercontent.com/43856126/115179701-6e07bd80-a0c3-11eb-9b59-f1192a73c5ff.png)

![image](https://user-images.githubusercontent.com/43856126/115179779-a0191f80-a0c3-11eb-81b1-51160c57cd3f.png)

Este nos dice que el administrador tiene un post privado.

![image](https://user-images.githubusercontent.com/43856126/115179832-bf17b180-a0c3-11eb-8b61-521a926195ca.png)

Al revisar este post privado podemos verlo y podemos obtener nuestra primera bandera.

# Flag 1

Ahora nos vamos a la sección de crear un nuevo post, y tenemos un form con dos campos:

![image](https://user-images.githubusercontent.com/43856126/115180280-be334f80-a0c4-11eb-9df7-64afd2cb5f88.png)

Ahora podemos crear un post de prueba, pero antes de darle al botón de create post, le damos a inspeccionar elemento para ver si se esta enviando algún tipo de id o algo similar.

![image](https://user-images.githubusercontent.com/43856126/115180403-0488ae80-a0c5-11eb-9582-ed7a8edc2ecb.png)

Efectivamente podemos ver que se envía un id que bien podría ser el número de identificador de un usuario. Pero no sabemos los demás indetificadores de los otros usuarios. Así que nos vamos a revisar nuevamente la página principal a los post existentes a ver si ahí se especifica algún número de usuario.

![image](https://user-images.githubusercontent.com/43856126/115180510-4154a580-a0c5-11eb-80cc-fe61386e8335.png)

Aquí podemos ver que el post de hola mundo fue hecho por el admin, y tiene un id de 1, así que podríamos tratar de mandar ese id desde el creador de post, y ver si afecta de alguna manera y podemos crear un post como si fueramos el usuario admin.

Para esto deberíamos quitar la etiqueta de hidden en post:

![image](https://user-images.githubusercontent.com/43856126/115180611-8d074f00-a0c5-11eb-9538-88e0a31e000f.png)

Y ahora solo cambiamos ese id al que nosotros queremos enviar, que sería el 1 que corresponde a admin.

Al enviar el post modificado podemos ver que se activo lo que hicimos y podemos obtener otra bandera:

![image](https://user-images.githubusercontent.com/43856126/115180714-d3f54480-a0c5-11eb-983a-4b9fc10c764f.png)


