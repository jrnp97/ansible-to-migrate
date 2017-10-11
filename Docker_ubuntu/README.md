# Configuracion de host docker #

En este directorio se encuentra el archivo Dockerfile para la creación de la imagen
la cual permitirá crear los contenedores de docker y el archivo **authorized_keys**
para la Configuracion de la autenticacion por llaves en el servicio SSH de cada contedenor.

**NOTA:** si desean generan sus propias llaves pueden realizarlo con el siguiente
comando `$ ssh-keygen -f /path/to/ansible-to-migrate/Keys/key`, donde lo que cambiaria es la ruta
/path/to/ la cual podria ser absoluta o relativa para mas información consulte [aquí](http://dep.fie.umich.mx/~stinoco/archivos/RutasAbsolutasYRelativas.pdf).

## Creación del contenedor ##

En este paso ya crearemos el contenedor y configuraremos el HOST anfitrion para la identificación correcta de estos.

Para ello realizaremos los siguientes pasos

1. Creación de la imagen en la que se basará el contenedor.

Se ejecuta el siguiente comando para la realización del pasos

```
$ (sudo) docker build -t {{ nombre contenedor }} .

En mi caso el {{ nombre contenedor }} será igual a server

$ (sudo) docker build -t ubuntu_host .

(sudo) -> Quiere decir que es opcional aveces con la instalación en sistemas linux, se requiere
sino se realiza el paso de POST-INSTALLATION mostrado en la documentacion.
```

Y tenemos creado nuestra imagen con ubuntu y ssh activo.

3. Creacion de contenedores.

Como se ha especificado se ha de utilizar 2 contenedores los cuales serán nuestos hosts en el sistema, por lo tanto para poder realizar la conexion correcta con los contenedores,
se les debe configurar un *alias* a cada uno, pero antes se deben crear con las siguientes instrucciones:

```
Nombre del contenedor main = server_main

$ (sudo) docker run -d -P --name ubuntu_main -p 2221:22 -p 80:80 ubuntu_host

Nombre del contenedor mysql = ubuntu_db

$ (sudo) docker run -d -P --name ubuntu_db -p 2222:22 -p 3306:3306 ubuntu_host

```

Los alias seran los nombres de los contenedores

 Alias |
 --- |
 ubuntu_main |
 ubuntu_db |

Entonces ahora registramos los aleas con el siguiente comando

` $ echo "127.0.0.1 ubuntu_main ubuntu_db" | sudo tee -a /etc/hosts `

Y con esto contamos con los contenedores creados y con posibilidad de identificación.

## Conexion por SSH ##

En este paso configuraremos el HOST anfitrion para la conexion por SSH con los contenedores de docker, siguiendo los siguientes pasos

2. Configurando nuestra llave privada.

Para realizar nuestra conexion nuestra llave privada debe contar con los permisos de esta forma **[-xw-------]** lo que equivale a **0600**.

Por tal razon ejecutamos el siguiente comando

```
$ chmod 0600 /path/to/ansible-to-migrate/Keys/key

Cabe resaltar que la ruta data /path/to/ansible-to-migrate/Keys/key puede ser relativa o
absoluta.
```

5. Registrando los contenedores a los host conocidos por ssh.

El servicio ssh cuenda con una llave ECDSA fingerprint para cada host y esta queda registrada en el archivo **known_hosts** que se encuentra en la siguiente ruta

*/home/{{ username }}/.ssh/known_hosts*

Dicha llave asegura que el host al cual se conectara es conocido por el host anfitrion.

Para realizar el registro de nuestros contenedores hay 2 opciones.

**Conectarse a cada uno con el comando correspondiente que seria**
```
$ ssh root@{{ nombre contenedor }} -p {{ puerto de ssh }} -i {{ llave privada }}

> En nuestro caso ambos comandos quedarian de la siguiente forma

$ ssh root@ubuntu_main -p 2221 -i /path/to/ansible-to-migrate/Keys/key

$ ssh root@ubuntu_db -p 2222 -i /path/to/ansible-to-migrate/Keys/key
```

y podriamos apreciar un mensaje como el siguiente (en cada primer intento de conexion)

![alt-text](/img/connection.png)

Al cual al ingresar *yes* se registrara el host, en los conocidos por ssh.

La otra opcion es:

**Registrarlos directamente con SSH**

Para esta opcion solamente basta con ejecutar los siguientes comandos, ya que este registra el host directamente.

```
Estructura
$ ssh -o StrictHostKeyChecking=no root@{{ nombre contenedor }} -p {{ puerto ssh }} -i {{ llave privada }} hostname

En nuestro caso seria lo siguiente

$ ssh -o StrictHostKeyChecking=no root@ubuntu_main -p 2221 -i /path/to/ansible-to-migrate/Keys/key hostname

$ ssh -o StrictHostKeyChecking=no root@ubuntu_db -p 2222 -i /path/to/ansible-to-migrate/Keys/key hostname
```

y podriamos apreciar un mensaje como el siguiente

![alt-text](/img/connection2.png)

Realizado los pasos anteriores, se procede a probar la conexion a los contenedores y podria apreciarse lo siguientes
![alt-text](/img/connection3.png).
