# ansible.awx installation
# Ref: https://github.com/ansible/awx/blob/devel/INSTALL.md#docker-or-docker-compose

# Pre-Requisites
mkdir -p ansible.awx
cd ansible.awx/

# Install PIP

curl -L "https://raw.githubusercontent.com/dilip682/ansible.awx/master/install-scripts/pip.sh" -o pip.sh
chmod 755 pip.sh
sh pip.sh

# Install Docker

curl -L "https://raw.githubusercontent.com/dilip682/ansible.awx/master/install-scripts/docker.sh" -o docker.sh
chmod 755 docker.sh
sh docker.sh

# Install Ansible
curl -L "https://raw.githubusercontent.com/dilip682/ansible.awx/master/install-scripts/ansible.sh" -o ansible.sh
chmod 755 ansible.sh
sh ansible.sh

# Install GNU Make, Git Requires Version 1.8.4+, Node 8.x LTS version, NPM 6.x LTS

sudo apt-get install nodejs npm  bzip2 -y
sudo apt install build-essential
sudo apt-get install git gettext -y

# Log off and login back for group membership to take effect.
# Test docker command
docker ps
# CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

# Install awx
git clone https://github.com/ansible/awx.git

# Update postgress data directory in inventory file.
# All username and passwords for admin accounts are in inventory
vi ~/ansible.awx/awx/installer/inventory
postgres_data_dir=/opt/pgdocker
docker_compose_dir=/var/lib/awx

sudo mkdir -p /opt/pgdocker
sudo chmod 755 /opt/pgdocker

# Start the build

# Set the working directory to installer
$ cd installer

# Run the Ansible playbook
$ ansible-playbook -i inventory install.yml

# Post build
$ docker ps
CONTAINER ID        IMAGE	          COMMAND	    CREATED             STATUS	PORTS	NAMES
fea286520020        ansible/awx_task:3.0.0       "/tini -- /bin/sh -c…"   3 minutes ago       Up 3 minutes        8052/tcp	    awx_task
1dc1a79ba986        ansible/awx_web:3.0.0        "/tini -- /bin/sh -c…"   3 minutes ago       Up 3 minutes        0.0.0.0:80->8052/tcp awx_web
156af79434c3        memcached:alpine             "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        11211/tcp memcached
4215cd220cad        ansible/awx_rabbitmq:3.7.4   "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        4369/tcp, 5671-5672/tcp, 15671-15672/tcp, 25672/tcp   rabbitmq
9edb062d7af0        postgres:9.6	   "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        5432/tcp	    postgres


# Tail the the awx_task log
$ docker logs -f awx_task
$ docker logs -f awx_web

# Accessing AWX
http://localhost
http://dilip6823c.mylabserver.com/
admin/password
