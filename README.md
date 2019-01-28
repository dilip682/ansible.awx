# ansible.awx

mkdir -p ansible.awx
cd ansible.awx/
curl -L "https://raw.githubusercontent.com/dilip682/ansible.awx/master/install-scripts/docker.sh" -o docker.sh
chmod 755 docker.sh
sh docker.sh
