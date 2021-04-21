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

    


