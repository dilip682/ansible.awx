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
# Your Inventory should look like this...
root@dilip6823c:/home/ubuntu/ansible.awx/awx/installer# cat inventory |grep -v "#"

localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python"
[all:vars]
dockerhub_base=ansible
dockerhub_version=latest

awx_task_hostname=awx
awx_web_hostname=awxweb

postgres_data_dir=/opt/pgdocker
host_port=80
docker_compose_dir=/var/lib/docker-compose-awx/
pg_username=awx
pg_password=awxpass
pg_database=awx
pg_port=5432

rabbitmq_user=awxuser
rabbitmq_password=awxpass
rabbitmq_erlang_cookie=cookiemonster

admin_user=admin
admin_password=password

create_preload_data=True

secret_key=awxsecret
project_data_dir=/var/lib/awx/projects

# Fix for RabbitMQ guest auth error.
vi ./ansible.awx/awx/installer/roles/local_docker/defaults/main.yml
rabbitmq_default_username: "awxuser"
rabbitmq_default_password: "awxpass"

vi ./ansible.awx/awx/installer/inventory
rabbitmq_user=awxuser
rabbitmq_password=awxpass

vi ./ansible.awx/awx/installer/roles/local_docker/templates/docker-compose.yml.j2
From:
rabbitmq_user=awxuser
rabbitmq_password=awxpass
To:
rabbitmq_user=awxuser
rabbitmq_password=awxpass


# Start the build
# Set the working directory to installer
$ cd installer

# Run the Ansible playbook
$ ansible-playbook -i inventory install.yml -vv

# Post build
$ docker ps
CONTAINER ID        IMAGE	          COMMAND	    CREATED             STATUS	PORTS	NAMES
fea286520020        ansible/awx_task:3.0.0       "/tini -- /bin/sh -c…"   3 minutes ago       Up 3 minutes        8052/tcp	    awx_task
1dc1a79ba986        ansible/awx_web:3.0.0        "/tini -- /bin/sh -c…"   3 minutes ago       Up 3 minutes        0.0.0.0:80->8052/tcp awx_web
156af79434c3        memcached:alpine             "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        11211/tcp memcached
4215cd220cad        ansible/awx_rabbitmq:3.7.4   "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        4369/tcp, 5671-5672/tcp, 15671-15672/tcp, 25672/tcp   rabbitmq
9edb062d7af0        postgres:9.6	   "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        5432/tcp	    postgres


# Tail the the awx_task log
$ docker logs -f awx_web
$ docker logs -f awx_task
Look for below messages in awx_task container.
>>> <User: admin>
>>> Default organization added.
Demo Credential, Inventory, and Job Template added.
Successfully registered instance awx
(changed: True)
Creating instance group tower
Added instance awx to tower
(changed: True)
...


# Accessing AWX
http://localhost
http://dilip6823c.mylabserver.com/
admin/password

# Maintenance using docker-compose
Maintenance operations with docker-compose can be done by using the docker-compose.yml file created at the location pointed by docker_compose_dir.

docker_compose_dir=/var/lib/awx

Among the possible operations, you may:

Stop AWX : docker-compose stop
Upgrade AWX : docker-compose pull && docker-compose up --force-recreate

# How to Un-Install AWX.
# List all containers (only IDs)
docker ps -aq

# Stop all running containers.
docker stop $(docker ps -aq)

# Remove all containers.
docker rm $(docker ps -aq)

# Remove all images.
docker rmi $(docker images -q)

# If required Remove old Postgress data & Old projects.
sudo rm -Rf /opt/pgdocker/
sudo rm -Rf /var/lib/awx/projects/


# Misc Notes:
Add user to wheen group for avoiding password prompt.
usermod -a -G wheel cloud_user
