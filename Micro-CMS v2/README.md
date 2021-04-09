Tenemos lo que parece ser el mismo sitio de la versión uno:

![image](https://user-images.githubusercontent.com/43856126/114126795-a7347680-98e8-11eb-9836-a8504543979a.png)

Con el cambio de que ahora si queremos editar o crear una nueva página nos redirige a un login:

![image](https://user-images.githubusercontent.com/43856126/114126865-be736400-98e8-11eb-8cae-749fa0d3e71e.png)

# Flag 0
Aquí lo primero que se me ocurre al ver un login es proba algo de SQL injection, así que con una prueba básica:

![image](https://user-images.githubusercontent.com/43856126/114126962-faa6c480-98e8-11eb-8530-2467ca3f768b.png)

Y esto nos dice que la contraseña no es la correcta:

![image](https://user-images.githubusercontent.com/43856126/114126995-07c3b380-98e9-11eb-8b3e-bb3d1f6bc3f0.png)

En este caso podríamos tratar de hacer un ataque de diccionario para ver si podemos encontrar la constraseña, pero a modo de prueba tratamos de romper la consulta simplemente agregando una comilla simple:

![image](https://user-images.githubusercontent.com/43856126/114127356-cd0e4b00-98e9-11eb-81d5-8810c660fe4a.png)

Y esto nos da un error:

![image](https://user-images.githubusercontent.com/43856126/114127406-e0211b00-98e9-11eb-98e2-c7fafbd74617.png)

Traté con algunos payloads para error based, pero lo que parece ser aquí esque tendríamos que usar un poco de blind SQL, así, que probando con un sleep se puede ver que se puede disparar el sleep
 
![image](https://user-images.githubusercontent.com/43856126/114129537-427c1a80-98ee-11eb-9a6b-652fd75e7c88.png)

