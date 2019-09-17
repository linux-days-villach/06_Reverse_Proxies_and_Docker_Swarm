# KVM server base setup ########################################################

This example is split up into snapshots.

## lbase #######################################################################
    ssh root@0.0.0.0
    ln -sfn /usr/share/zoneinfo/UTC /etc/localtime
    localectl set-locale LANG=en_US.UTF-8
    echo 'linuxday' > /etc/hostname
    reboot
    ssh root@0.0.0.0
    yum distro-sync
    yum install epel-release
    yum install epel-release
    yum install fish
    adduser username -c 'Full Name' -G users && passwd username
    usermod root -s '/usr/bin/fish'
    sed -i 's/.*PermitRootLogin ...\?$/PermitRootLogin no/g' /etc/ssh/sshd_config
    poweroff

## ldocker #####################################################################
    ssh username@0.0.0.0
    su -
    yum install -y yum-utils device-mapper-persistent-data lvm2
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install docker-ce-18.09.1 docker-ce-cli-18.09.1 containerd.io
    systemctl start docker.service
    systemctl enable docker.service
    poweroff

## lds #########################################################################
    ssh username@0.0.0.0
    su -
    docker swarm init --advertise-addr 0.0.0.0
    yum install git
    git clone https://gist.github.com/2b3bfd8ff886015bbce8.git /tmp/docker-shell
    install /tmp/docker-shell/docker-shell /usr/local/sbin/
    rm -rf /tmp/docker-shell
    docker network create --driver overlay --subnet=185.143.7.0/27 traefik-net
    docker service create --name traefik --network traefik-net --publish mode=host,target=80,published=80 --publish 8080:8080 --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock --constraint=node.role==manager traefik --docker.swarmMode --docker.domain=wetter.host --docker.watch --api --docker
    docker service create --name portainer --network traefik-net --label "traefik.enable=true" --label "traefik.port=9000" --label "traefik.docker.network=traefik-net" --label "traefik.frontend.rule=Host:p2.wetter.host" --label "traefik.backend=portainer" --constraint 'node.role == manager' --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock --mount type=volume,src=portainer_data,dst=/data portainer/portainer:1.18.1 -H unix:///var/run/docker.sock
    poweroff
