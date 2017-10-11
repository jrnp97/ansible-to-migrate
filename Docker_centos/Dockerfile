FROM centos:centos6.7

#Configuracion de host con ssh y autenticacion por llaves

# Actualizando lista de paquetes
RUN yum update -y

# Instalando python, pip, ssh
RUN yum install -y openssh-server python python-pip

#Configurando llave publica
# - Creando directorio donde se almacenará el socker del servicio ssh
RUN mkdir /var/run/sshd
# - Anexando nuestra llave publica
COPY authorized_keys /root/.ssh/authorized_keys
# - Proporcionando los permisos pertinentes a la llave [-xw-------]
RUN ["chmod","0600","/root/.ssh/authorized_keys"]

CMD ["/usr/sbin/sshd", "-D"]
