
# Pasos
sudo ip addr add 192.168.131.55/24 dev enp5s0:0


gpg --gen-key

cat /dev/urandom

gpg --no-default-keyring --keyring /usr/share/keyrings/ubuntu-archive-keyring.gpg --export | gpg --no-default-keyring --keyring trustedkeys.gpg --import

gpg --export --armor > my_key.pub

scp vagrant@ip_solo_anfitrion:/tmp/my_key.pub .


-# echo deb http://repo.aptly.info/ squeeze main > /etc/apt/sources.list

sudo apt-key adv --keyserver keys.gnupg.net --recv-keys 9E3E53F19C7DE460


aptly mirror create -architectures=amd64 -filter='Priority (required) | Priority (important) | Priority (standard) | postgresql' -filter-with-deps xenial-main-postgresql http://mirror.upb.edu.co/ubuntu/ xenial main




# editando......

