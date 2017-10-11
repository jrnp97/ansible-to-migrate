# Arquitectura #

La Arquitectura base del sistema con CentOS o RHEL del sistema es la siguiente

![alt-text](/img/structure.png)

tomando en cuenta la explicado en el README principal previo exactamente en la siguiente linea

![alt-text](/img/Prev_README.png)

Cabe aclarar que los host se encuentran en el mismo segmento de red es lo que se puede deducir ya que no se especifica ninguna ip del Server de la base de datos.

**Aunque tambien puede darse el acaso que el host *bensible* sea el mismo localhost pero con un alias.**

## Caracteristicas del Server ##

En el servidor web se realiza la instalación del servicio httpd y ntp, los cuales funcionan como servidor web y administrador de fechas en equipos respectivamente para la instalación de estos en los hosts con ansible se utilizaron los siguientes modulos

- **yum:** permite el uso del manejador de repositorios de Centos, para mas documentacion [aquí](http://docs.ansible.com/ansible/latest/yum_module.html).
- **lineinfile:** funciona como la funcion sed en bash, permite el reemplazo o adicion de contenido a archivos buscando este por una expeción regular si la encuentra realiza lo estipulado en el atributo **line**, cabe aclara que en el atributo **regexp** se especifica la expresion regular a buscar, para mas documentacion [aquí](http://docs.ansible.com/ansible/latest/lineinfile_module.html).
- **service:** modulo que permite manejar los servicios en un host, para mas información [aquí](http://docs.ansible.com/ansible/latest/service_module.html).
- **seboolean:** permite la configuración de variables en el modulo seguro SELinux cambiando su estado a true o false dependiendo de su uso, para más información [aquí](http://docs.ansible.com/ansible/latest/seboolean_module.html).
- **git:** Modulo que permite el uso de git en el host que lo ejecuta.
- **template:** Modulo que permite copiar archivos de configuración o normales personalizados en el host.

### Uso de modulos ###

Explicare un poco para que se utilizan los modulos en los playbooks y su posible equivalencia en ubuntu (si se aplica esta).

**yum**

El modulo **yum** se ha utilizado para instalar los siguientes paquetes en el host

- httpd
- php
- php-mysql
- git
- libsemanage-python
- libselinux-python

Los cuales permiten el uso del servicio **httpd** y dependencias de 3ros, git para el control de versiones, **php y php-mysql** para hacer posible la conexion al motor de base de datos MySQL al cual se conectará y los paquetes **libsemanage-python y libselinux-python** para poder hacer uso del modulo **seboolean**.

Del modulo yum en ubuntu su equivalencia seria el gestionador de paquetes de ese sistema **apt**, manteniendo los mismos atributos.

**lineinfile**

El modulo **lineinfile** lo que realiza es la agregación de reglas iptables para permitir las conexiones externas, es decir, permitir que el servicio **httpd** reciba peticiones del exterior.

A diferencia que en CentOS, si miramos desde la perpestiva de docker los contenedores basados en ubuntu no tienen iptables, pero los VPS o hosting alquilados si cuentan con ello, por ende al **realizar prueba de los playbooks en contenedores** no seria necesario utilizar dicha instruccion ya que docker cuenta con sus propios metodos para permitir conexiones desde el exterior.

Pero en el caso que se desee inplementar la solucion en un VPS o cualquier otro dispositivo se podria hacer uso del modulo **iptables**, para no editar directamente el archivo.

**service**

El modulo **service** se utiliza para iniciar el servicio httpd y habilitarlo (insertarlo en la lista de programas que se ejecutan al iniciar el host).

En el sistema ubuntu se puede realizar el mismo modulo sin modificación.

**seboolean**

El modulo **seboolean** permite la maniipulación se SELinux el cual se encuentra deshabilitado en ubuntu, el cual podría mantenerse del mismo, es decir dicho paso en ubuntu podría muy bien no realizarse.

**git**

El modulo **git** se utiliza primordialmente para realizar la clonación de una aplicación en el directorio web del servicio httpd **/var/www/html/**.

**template**

El modulo **template** copia el archvio index.php personalizado el directorio web del servicio httpd.

Antes de terminar con las Caracteristicas del servidor cabe aclarar que el servicio httpd es el equivalente al servicio apache2 en ubuntu pero sus configuraciónes permanecen parecidad, por ende se instalaria apache2 en vez de httpd.


## Caracteristicas del DB ##

Para la configuración del host con el motor de base de datos se instala con Ansible el motor MySQL y se realiza el uso de modulos ya explicados en las Caracteristicas del Server que son como:

- **yum**
- **lineinfile**
- **service**
- **seboolean**

Los anteriores modulos ya fueron explicados y sus equivalencias en ubuntu si son posibles, los nuevos modulos observados son

- **mysql_db:** este modulo permite gestionar las bases de datos en el motor MySQL, para mas información [aquí](http://docs.ansible.com/ansible/latest/mysql_db_module.html)
- **mysql_user:** este modulo permite gestionar los usuarios de una base de datos MySQL, para mas informacion [aquí](http://docs.ansible.com/ansible/latest/mysql_user_module.html).

La equivalencia de estos ultimos 2 modulos no es de discución ya que estos interactuan directamente con el servicio MySQL no con el OS en el host.

Por lo tanto su uso será igual.

## Rol common ##

En este rol se busca instalar el servicio ntp, que permite sincronizar el tiempo y fecha de distintos dispositivo es decir proporcionarles el mismo UTC.

Los modulos usados en este rol ya se han explicado y sus equivalencias.

## Migración de scripts ##

Para la migración de script a ubuntu se ha preparado la siguiente Arquitectura

![alt-text](/img/structure0.png)

Ya que se ha de realizar con docker, las tareas relacionadas con **iptables** se les hará caso omiso por lo explicado en su momento anterior a esta sección al igual que las tareas que utilizan SELinux por el anuncio dado en la web oficial de inpreciso, puede observarlo [aquí](https://wiki.ubuntu.com/SELinux).

Dadas las razones anteriores se procederá a migrar los scripts.

Como los playbooks funcionan con ansible 1.2 y ya ansible cuenta con la versión 2.4, acciones donde se utilizaba **include** para importar tareas se ha despreciado y se utilizan **import_tasks** para importar tareas estaticas o **include_tasks** para importar tareas dinamicas.

Una pequeña definicion de tareas dinamicas e estaticas son las siguientes

- En las importaciones estaticas las opciones de la tarea padre serán copiadas a todas las tareas hijas contenidas en el modulo de referenciación.

- En las importaciones dinamicas, las opciones de la tarea padre no son heredadas a las tareas hijas.

Para mas información en la documentación [aquí](http://docs.ansible.com/ansible/latest/playbooks_reuse.html).

En nuestro caso utilizaremos importaciones staticas para que las tareas hijas hereden las opciones del padre.

Al ejecutar el playbook general con el siguiente comando

```
ansible-playbook -i hosts site.yml

Asumiendo que se encuentre en el directorio ansible-to-migrate
```

Lo cual debe generar algo parecido a la siguiente salida

![alt-text](/img/Migrate_status.png)

y al ingresar en la url http://localhost/ se podrá apreciar el contenido de la app.

![alt-text](/img/Home_apache.png)

Ademas de las siguientes 2 rutas

- **http://localhost/index.php**

 ![alt-text](/img/Home_apache2.png)

- **http://localhost/about.html**

 ![alt-text](/img/About_apache.png)

Y listo nuestra migración esta finalizada con tecnologia docker, cabe aclarar que para realizar la migración y los hosts son "reales" podría incluirse la configuración de iptables o manejar directamente un firewall (puede ser firewalld el cual cuenta con modulo en ansible).
