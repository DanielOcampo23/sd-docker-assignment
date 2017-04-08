# TALLER MIRROR, CONTENEDOR, BASES DE DATOS

# Daniel Steven Ocampo
# A00053980

# Objetivo

- Crear un mirror que contenga los archivos de Postgresql.
- Levantar un contenedor que descargue Postgresql desde el mirror creado.
- Dentro del contenedor crear una base de datos para probar el funcionamiento.

# Pasos

Primero creamos una máquina virtual para poder configurar y que quede como mirror, para esto se necesita el siguiente Vagrantfile:

```
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false
  config.vm.define :mirror do |trusty|
    trusty.vm.box = "trusty"
    trusty.vm.network "private_network", ip: "192.168.131.10"
    trusty.vm.network "public_network", bridge: "enp5s0", ip: "192.168.131.11"
    trusty.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024","--cpus", "1", "--name", "mirror" ]
    end
  end
end	
```

Después de tener el Vagrantfile se hace un 
```
vagrant up
```

y entramos a la máquina con un
```
vagrant ssh centos_mirror(Nombre de la máquina especificada en el Vagrantfile)
```

Una vez adentro de la máquina virtual se procede a generar la clave 
```
gpg --gen-key
```

En el proceso degenerar la clave se necesita que una entropia para ayudar a generar la clave
```
cat /dev/urandom
```

Después de generarse la clave la exportamos dentro de la carpeta /tmp/
```
gpg --no-default-keyring --keyring /usr/share/keyrings/ubuntu-archive-keyring.gpg --export | gpg --no-default-keyring --keyring trustedkeys.gpg --import
$ gpg --export --armor > my_key.pub
```

Después de exportar la clave, abrimos una terminal por fuera de la máquina virtual para poder sacarla
```
scp vagrant@192.168.131.10:/tmp/my_key.pub .
```

Después de haber exportado la clave por fuera de la máquina virtual, volvemos a la máquina virtual e instalamos aptly
```
echo deb http://repo.aptly.info/ squeeze main > /etc/apt/sources.list
sudo apt-key adv --keyserver keys.gnupg.net --recv-keys 9E3E53F19C7DE460
$ apt-get update
$ apt-get install aptly
```

Luego de instalar aptly creamos el mirror a traves de aptly con el siguiente comando
```
aptly mirror create -architectures=amd64 -filter='Priority (required) | Priority (important) | Priority (standard) | postgresql' -filter-with-deps xenial-main-postgresql http://mirror.upb.edu.co/ubuntu/ xenial main
```

Y lo actualizamos para que descargue los paquetes necesarios
```
aptly mirror update xenial-main-postgresql
```

Creamos un snaphost para poder exponer los paquetes y que puedan ser descargados desde otras máquinas virtuales y después las publicamos e ingresamos una passphrase
```
$ aptly snapshot create xenial-snapshot-postgresql from mirror xenial-main-postgresql
$ aptly publish snapshot xenial-snapshot-postgresql
```

Por último iniciamos el mirror.
``` Cancel
Contact GitHub API Training Shop Blog About

$ aptly serve
```

# Paso siguiente: crear el contenedor con docker

FOTO 
```
![GitHub Logo] (/images/logo-png)
```

Como lo muestra en la imagen para poder crear la imagen necesitamos un Dockerfile con las siguientes especificaciones
```
FROM ubuntu:16.04
MAINTAINER tebannew@gmail.com	

ADD config/my_key.pub /tmp

#COnfigure repositorio
RUN apt-key add /tmp/my_key.pub && \
    rm -f /tmp/my_key.pub && \
    echo "deb http://192.168.131.11:8080/ xenial main" >  /etc/apt/sources.list && \
    chmod 777 /tmp

#Install packege
RUN apt-get clean -y
RUN apt-get update -y
RUN apt-get install postgresql -y

EXPOSE 5432
CMD postgresql -m http.server 5432
```

Después para poder construir la imagen se hace lo siguiente
```
$ docker build -t ubuntu_postgresql:0.0.1 .
```

Después como lo muestra la estrutura antes mostrada, después de tener la imagen se corre para poder levantar el contenedor esto lo hacemos con el siguiente código Cancel
Contact GitHub API Training Shop Blog About

```
$ docker run -it --rm ubuntu_postgresql:0.0.1 /bin/bash
```

# Paso siguiente : prueba de funcionamiento con la creación de una base de datos

Activación postgres
```
/etc/init.d/postgresql start 
```

Siguiente a esto cambiamos el usuario de postgres y abrimos la consola de Postgresql.
```
su - postgres
psql
```

Creamos la base de datos
```
CREATE DATABASE pruebaFuncionamiento;
```

Nos connectamos a la base de datos que acabamos de crear
```
\c pruebafuncionamiento
```

Y creamos una tabla en la bases de datos
```
CREATE TABLE Cuentas
(
  NumeroCuenta int,
  ClienteId int,
  NombreCliente varchar(255),
  Saldo varchar(255),
  Clave varchar(255),
);
```

E insertamos algunos datos 
```
INSERT INTO cuentas (NumeroCuenta, ClienteId, NombreCliente, Saldo, Clave) VALUES (123, 12345, 'Daniel', 10000000, cuenta123);
INSERT INTO cuentas (NumeroCuenta, ClienteId, NombreCliente, Saldo, Clave) VALUES (456, 67890, 'Pepe', 20000000, micuenta2);
```

y por ultimo podemos realizar una consultar para confirmar el funcionamiento
```
SELECT NumeroCuenta FROM cuentas; 
```
