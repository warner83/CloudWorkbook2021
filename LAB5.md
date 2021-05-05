# OpenStack installation

# Before starting, mark the VMs that you have in order to assign one of the following roles: Manager, Controller, Compute1, Compute2, Compute3
# Contoller MUST be the VM assigned to ypur group (it has more RAM)

# [ON ALL] Configure network connection, modify /etc/netplan/01-netcfg.yaml to add the following
    vlans:
     vlan2:
      id: 2
      link: eth0
      dhcp4: no
      dhcp6: no

# [ON ALL] Apply the configuration
    netplan apply
    
# [ON MANAGER] Install JuJu
    sudo snap install juju --classic
    
# [ON MANAGER] Create an ssh key to connect without password
    ssh-keygen

# [ON MANAGER] Copy the ssh key to all the machines
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@MANAGER
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@CONTROLLER
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@COMPUTE1
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@COMPUTE2
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@COMPUTE3
    
# [ON MANAGER] Create a new managed cloud
    juju add-cloud
    
# [ON MANAGER] Bootstrap the cloud
    juju bootstrap mycloud manual-controller
    juju status
    
# [ON MANAGER] Add machine (it is important that the controller is added as first)
    juju add-machine ssh:root@CONTROLLER
    juju add-machine ssh:root@COMPUTE1
    juju add-machine ssh:root@COMPUTE2
    juju add-machine ssh:root@COMPUTE3
    
# [ON MANAGER] Configure the network (skipping this step might result in a disaster)
    juju model-config fan-config=172.16.0.0/16=252.0.0.0/8
    juju model-config container-networking-method=fan
    juju model-config | egrep 'fan-config|container-networking-method'

# [ON CONTROLLER] Configure the controller to use the second hard drive to store module data
    fdisk /dev/sdb (option n to create a new partition and then option w to write changes)
    mkfs.ext4 /dev/sdb1
    mkdir -p /var/lib/lxd/storage-pools/default/
    mount /dev/sdb1 /var/lib/lxd/storage-pools/default/
    rm -rf /var/lib/lxd/storage-pools/default/lost+found
    ADD IN THE /etc/fstab file: /dev/sdb1 /var/lib/lxd/storage-pools/default ext4 errors=remount-ro 0 1

# [ON MANAGER] Deploy nova
# Create a new file compute.yaml
    nova-compute:
      enable-live-migration: True
      enable-resize: True
      migration-auth-type: ssh
      virt-type: qemu
      openstack-origin: cloud:bionic-train
# Deploy
    juju deploy --to 1 --config compute.yaml nova-compute
    juju add-unit --to 2 nova-compute

# [ON MANAGER] Deploy neutron compute
# Create a new file neutron.yaml
    neutron-gateway:
        bridge-mappings:         physnet1:br-ex
        data-port:               br-ex:vlan2
        openstack-origin: cloud:bionic-train

     neutron-api:
        flat-network-providers:  physnet1
        overlay-network-type: gre
        neutron-security-groups: True
        openstack-origin: cloud:bionic-train
        
    neutron-openvswitch:
       bridge-mappings:         physnet1:br-ex
       data-port:               br-ex:vlan2
       firewall-driver:         openvswitch

# Deploy
    juju deploy --to 0 --config neutron.yaml neutron-gateway
    juju deploy --to lxd:0 --config neutron.yaml neutron-api
    juju deploy --config neutron.yaml neutron-openvswitch
    
    juju add-relation neutron-api:neutron-plugin-api neutron-gateway:neutron-plugin-api
    juju add-relation neutron-api:neutron-plugin-api neutron-openvswitch:neutron-plugin-api
    juju add-relation neutron-openvswitch:neutron-plugin nova-compute:neutron-plugin

# [ON MANAGER] Deploy MySQL
# Create a new file mysql.yaml

    mysql:
      max-connections: 20000
      source: cloud:bionic-train
# Deploy
    juju deploy --to lxd:0 --config mysql.yaml percona-cluster mysql
    juju add-relation neutron-api:shared-db mysql:shared-db
    
# [ON MANAGER] Deploy Keystone
# Create a new file keystone.yaml

    keystone:
      admin-password: openstack
      openstack-origin: cloud:bionic-train

# Deploy

    juju deploy --to lxd:0 --config keystone.yaml keystone
    juju add-relation keystone:shared-db mysql:shared-db
    juju add-relation keystone:identity-service neutron-api:identity-service

# [ON MANAGER] Deploy RabbitMQ
# Deploy
    juju deploy --to lxd:0 rabbitmq-server

    juju add-relation rabbitmq-server:amqp neutron-api:amqp
    juju add-relation rabbitmq-server:amqp neutron-openvswitch:amqp
    juju add-relation rabbitmq-server:amqp nova-compute:amqp
    juju add-relation rabbitmq-server:amqp neutron-gateway:amqp

# [ON MANAGER] Deploy Nova Controller
# Create a new file controller.yaml (change the IP address with the IP address of the CONTROLLER)

    nova-cloud-controller:
      network-manager: Neutron
      console-access-protocol: novnc
      console-proxy-ip: 172.16.3.26
      openstack-origin: cloud:bionic-train


# Deploy
    juju deploy --to lxd:0 --config controller.yaml nova-cloud-controller
    juju add-relation nova-cloud-controller:shared-db mysql:shared-db
    juju add-relation nova-cloud-controller:identity-service keystone:identity-service
    juju add-relation nova-cloud-controller:amqp rabbitmq-server:amqp
    juju add-relation nova-cloud-controller:quantum-network-service neutron-gateway:quantum-network-service
    juju add-relation nova-cloud-controller:neutron-api neutron-api:neutron-api
    juju add-relation nova-cloud-controller:cloud-compute nova-compute:cloud-compute

# [ON MANAGER] Deploy Placement
# Create a new file placement.yaml

    placement:
      openstack-origin: cloud:bionic-train
      
# Deploy

    juju deploy --to lxd:0 --config placement.yaml placement

    juju add-relation placement:shared-db mysql:shared-db
    juju add-relation placement:identity-service keystone:identity-service
    juju add-relation placement:placement nova-cloud-controller:placement


# [ON MANAGER] Deploy Dashboard
# Create a new file dashboard.yaml (change the IP address with the IP address of the CONTROLLER)
    openstack-dashboard:
      openstack-origin: cloud:bionic-train
      os-public-hostname: 172.16.3.26
# Deploy
    juju deploy --to 0 --config dashboard.yaml openstack-dashboard

    juju add-relation openstack-dashboard:identity-service keystone:identity-service
    
    
# [ON MANAGER] Deploy Glance
# Create a new file glance.yaml 
    glance:
      openstack-origin: cloud:bionic-train

# Deploy
    juju deploy --to lxd:0 --config glance.yaml glance

    juju add-relation glance:image-service nova-cloud-controller:image-service
    juju add-relation glance:image-service nova-compute:image-service
    juju add-relation glance:shared-db mysql:shared-db
    juju add-relation glance:identity-service keystone:identity-service
    juju add-relation glance:amqp rabbitmq-server:amqp

# [ON MANAGER] Deploy Cinder
# Create a new file cinder.yaml 

    cinder:
      glance-api-version: 2
      block-device: None 
      openstack-origin: cloud:bionic-train

# Deploy
    juju deploy --to lxd:0 --config cinder.yaml cinder

    juju add-relation cinder:cinder-volume-service nova-cloud-controller:cinder-volume-service
    juju add-relation cinder:shared-db mysql:shared-db
    juju add-relation cinder:identity-service keystone:identity-service
    juju add-relation cinder:amqp rabbitmq-server:amqp
    juju add-relation cinder:image-service glance:image-service

# [ON MANAGER] Deploy Ceph OSD
# Create a new file ceph-osd.yaml
    ceph-osd:
      osd-devices: /dev/sdb
      source: cloud:bionic-train

# Deploy
    juju deploy --to 1 --config ceph-osd.yaml ceph-osd
    juju add-unit --to 2 ceph-osd
    juju add-unit --to 3 ceph-osd

# [ON MANAGER] Deploy Ceph MON
# Create a new file ceph-mon.yaml
    ceph-mon:
        source: cloud:bionic-train


# Deploy
    juju deploy --to lxd:1 --config ceph-mon.yaml ceph-mon
    juju add-unit --to lxd:2 ceph-mon
    juju add-unit --to lxd:3 ceph-mon

    juju add-relation ceph-mon:osd ceph-osd:mon
    juju add-relation ceph-mon:client nova-compute:ceph
    juju add-relation ceph-mon:client glance:ceph
    juju add-relation cinder ceph-mon
    
# [ON MANAGER] Deploy NTP
# Deploy
    juju deploy ntp

    juju add-relation ceph-osd:juju-info ntp:juju-info


