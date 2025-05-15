```
snap install multipass
apt install qemu-kvm virt-manager libvirt-daemon libvirt-daemon-system
snap connect multipass:libvirt
multipass set local.driver=libvirt

# Cloud-Init to set Ubuntu User
cat <<'EOF'> multipass.yaml
#cloud-config
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: false
  bridges:
    br0:
      dhcp4: true
      interfaces: [ ens3 ]
      parameters:
        stp: false
        forward-delay: 0
manage_etc_hosts: False
system_info:
  default_user:
    name: ubuntu
    gecos: Ubuntu
password: pass
chpasswd: { expire: False }
ssh_pwauth: True
ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFOvXax9dNqU2unqd+AZQ+VSe2cZZbGMVRuzIW4Hl6Ji69R0zkWih0vuP2psRA/uWTg1XqFKisCp9Z1XQcBbH2WLhnIWhykeLOHtBdEQqUApKj+BrKnyDmBbCourUwAcuUQSRPeRBOg5hwReviIebwvELmwc8ab1r0X+nbCDwVdohTpwNnxHp5MTO0WADLdP0oDQy2hhVaiParCWdVvgfDauQ2IpgeN6tE5sUvsDyYLaYp/dIhddA/Dwh9sWEFfN7ERMSHJw/A/3GsQ49a8+w6lamgcfNDKK7hE9F5vn95fzhge0jj6Yl8NTXOzoMfpvPo3Q+uCbu+GRMlRAK3hcHP wilton.pem
runcmd:
  - |
    # Fix Oracle Linux Serial Console
    if [ -e /etc/system-release ]; then
      if [ "$(sed 's, release .*$,,g' /etc/system-release)" == "Oracle Linux Server" ]; then 
        if ! grep ttyS0 /etc/default/grub; then
          sed -i "s/console=tty0/console=ttyS0,115200n8 console=tty0/g" /etc/default/grub;
          grub2-mkconfig -o /boot/grub2/grub.cfg;
          grep 'Linux release 9' /etc/redhat-release && grub2-mkconfig -o /boot/grub2/grub.cfg --update-bls-cmdline;
          systemctl enable serial-getty@ttyS0.service;
          echo "Serial Console Fixed... Rebooting...";
          reboot;
        fi
      fi
    fi
# # Ubuntu
# multipass launch -n primary -v -c 1 -m 1G -d 10G --cloud-init multipass.yaml 24.04
# multipass exec primary -- bash

# # Debian
# debian_img=http://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
# multipass launch -n debian -v -c 2 -m 2G -d 10G --cloud-init multipass.yaml $debian_img
# multipass exec debian -- bash

# # Oracle Linux 9
# oracle_linux_9_img=https://yum.oracle.com/templates/OracleLinux/OL9/u5/x86_64/OL9U5_x86_64-kvm-b253.qcow2
# multipass launch -n oracle-linux-9 -v -c 2 -m 2G -d 40G --cloud-init multipass.yaml $oracle_linux_9_img
# multipass exec oracle-linux-9 -- bash

# # Oracle Linux 8
# oracle_linux_8_img=https://yum.oracle.com/templates/OracleLinux/OL8/u10/x86_64/OL8U10_x86_64-kvm-b237.qcow2
# multipass launch -n oracle-linux-8 -v -c 2 -m 2G -d 40G --cloud-init multipass.yaml $oracle_linux_8_img
# multipass exec oracle-linux-8 -- bash

# # Oracle Linux 7
# oracle_linux_7_img=https://yum.oracle.com/templates/OracleLinux/OL7/u9/x86_64/OL7U9_x86_64-olvm-b86.qcow2
# multipass launch -n oracle-linux-7 -v -c 2 -m 2G -d 40G --cloud-init multipass.yaml $oracle_linux_7_img
# multipass exec oracle-linux-7 -- bash

# # CentOS 7
# centos_img=https://cloud.centos.org/altarch/7/images/CentOS-7-x86_64-GenericCloud.qcow2
# multipass launch -n centos -v -c 1 -m 1G -d 10G --cloud-init multipass.yaml $centos_img
# multipass exec centos -- bash

# # Cleanup
# multipass delete primary debian oracle-linux-8 oracle-linux-7 centos
# multipass purge

# # Install
# sudo snap install multipass
# sudo apt install qemu-kvm virt-manager libvirt-daemon libvirt-daemon-system
# sudo snap connect multipass:libvirt
# multipass set local.driver=libvirt
# multipass set client.gui.autostart=false
# # Restart
# sudo snap restart multipass.multipassd
# sudo snap connect multipass:libvirt
EOF
```
