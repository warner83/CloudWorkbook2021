# Check hardware virtualization support
sudo apt install cpu-checker
kvm-ok

# Install KVM and libvirt

    sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst

    sudo adduser `id -un` libvirt

# Interact with virsh

    virsh list
    virsh nodeinfo

# Install VM from network

    sudo virt-install \
      --name Ubuntu \
      --description 'Test VM with Ubuntu' \
      --ram=512 \
      --vcpus=1 \
      --os-type=Linux \
      --os-variant=ubuntu18.04 \
      --disk path=/var/lib/libvirt/images/ubuntu1804.qcow2,bus=virtio,size=1 \
      --graphics none \
      --location 'http://us.archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/' \
      --network bridge:virbr0 \
      --console pty,target_type=serial -x 'console=ttyS0,115200n8 serial'

  
# Delete a VM

    virsh destroy Ubuntu
    virsh undefine Ubuntu

# Import a preinstalled VM

    wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
    mv cirros-0.4.0-x86_64-disk.img /var/lib/libvirt/images
    
    sudo virt-install --name Cirros --description 'Cirros' --ram=512 --vcpus=1 --os-type=Linux --os-variant=ubuntu18.04 --disk path=/var/lib/libvirt/images/cirros-0.4.0-x86_64-disk.img,bus=virtio,format=raw --network bridge:virbr0 --graphic none --import

# Interact with virsh part2

    virsh console Cirros
    virsh shutdown Cirros
    virsh suspend Cirros
    virsh autostart Cirros

# Virsh networking

    virsh net-list

# Define a new network

    <network>
      <name>net2</name>
      <bridge name="virbr1"/>
      <forward mode="nat"/>
      <ip address="192.168.2.1" netmask="255.255.255.0">
        <dhcp>
          <range start="192.168.2.2" end="192.168.2.254"/>
        </dhcp>
      </ip>
    </network>


# Create a new network

    virsh net-define net2.xml
    virsh net-autostart net2
    virsh net-start net2
    
    virsh attach-interface --domain Cirros --type network --source net2 --config –-live
    sudo ip addr add IP_ADDRESS/MASK dev eth1
    sudo ip link set up dev eth1

# Storage Pool

    virsh pool-list
    virsh pool-info images
    virsh pool-dumpxml images
    
    virsh vol-create-as images disk2 100M
    virsh vol-list images
    virsh attach-disk Cirros /var/lib/libvirt/images/disk2 vdb --live

# Format and mount the disk inside the VM

    sudo fdisk /dev/vdb
    sudo mkfs.ext4 /dev/vdb1
    sudo mount /dev/vdb1 /mnt

# Clean UP

    virsh destroy Cirros
    virsh undefine Cirros
    virsh net-destroy net2
    sudo apt-get purge qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst
    reboot

