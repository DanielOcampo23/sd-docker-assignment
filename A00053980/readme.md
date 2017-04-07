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

Después de tener el VagrantFile se hace un 
```
vagrant up
```

y entramos a la máquina con un
```
vagrant ssh  centos_mirror(Nombre de la máquina especificada en el Vagrantfile)
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



# editando......

