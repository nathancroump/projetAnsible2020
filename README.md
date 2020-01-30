# projetAnsible2020
Thomas Brasselet/ Nathan CROUMP

#deploy.yaml
# Create new VM with minimal options
- hosts: all
  tasks:
        - name: "Deploy VM"
          proxmox_kvm:
              api_user     : root@pam
              api_password : itescia
              api_host     : 192.168.58.135
              name         : testVM
              node         : pve
              ide          : '{"ide0":"data:iso/ubuntu-18.04.1-live-server-amd64.iso,media=cdrom"}'
              autostart    : yes
              cores        : 2
              memory       : 512
              net          : '{"net0":"virtio,bridge=vmbr0,rate=200"}'
              virtio       : '{"scsi0":"replica:20,format=raw"}'
              ostype       : l26


        - name: "PowerON"
          proxmox_kvm:
              api_user    : root@pam
              api_password: itescia
              api_host    : 192.168.58.135
              name        : testVM
              node        : pve
              state       : started



#hosts.yaml
#hyperviseur:
[hosts]
pve  ansible_host=192.168.58.135
pve2 ansible_host=192.168.58.137
pve3 ansible_host=192.168.58.138




#iptables.yaml
---

-
  hosts: all
  tasks:
   - name: AllowSSH
     iptables:
          chain: INPUT
          comment: "Accept new SSH connections."
          ctstate: NEW
          destination_port: 22
          jump: ACCEPT
          protocol: tcp

   - name: AllowHTTPS
     iptables:
          chain: INPUT
          comment: "Accept new HTTPS connections."
          ctstate: NEW
          destination_port: 8006
          jump: ACCEPT
          protocol: tcp








#ssh.yaml
-
 hosts: all
 tasks:
  - name: Add the user 'itescia' with a specific uid and a primary group of 'sudo'
    user:
     name: itescia
     comment: itescia
     uid: 1045
     group: sudo
  - name: "SSHKEYGEN"
    authorized_key:
     user: itescia
     state: present
     key: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_rsa.pub')  }}"
