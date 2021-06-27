## Create Easy Virtual machines from cloud images using Ansible playbooks

Following this [tutorial](https://medium.com/@art.vasilyev/use-ubuntu-cloud-image-with-kvm-1f28c19f82f8) for deploying virtual machines I created an ansible Playbook that allows create virtual machines easy way.
Operating System Details
Fedora 32 with ansible and libivirt installed.

~~~yaml
---
- name: Create virtual machine in cloud format
  hosts: localhost
  become: yes
  vars:
    base_dir: /var/lib/libvirt/images/cloud-images/base/
    target_dir: /var/lib/libvirt/images/cloud-images/
    rsa_key_location: /home/user/.ssh/id_rsa.pub
  vars_prompt:
    - name: vm_name
      private: no
    - name: base_image
      private: no
    - name: username
      private: no
    - name: hostname
      private: no
    - name: os_variant
      private: no
    - name: disk_size
      private: no
      default: 20G
    - name: ram_size
      private: no
      default: 4096
    - name: vcpus
      private: no
      default: 6
  tasks:
    - name: create disk
      command: qemu-img create -f qcow2 -o backing_file={{base_dir}}/{{base_image}} {{target_dir}}/{{vm_name}}.qcow2
    - name: resize disk
      command: qemu-img resize {{target_dir}}/{{vm_name}}.qcow2 {{disk_size}}
    - name: create meta-data
      raw: |
        cat >meta-data <<EOF
        local-hostname: {{hostname}}
        EOF
    - name: export pub key
      command: cat {{rsa_key_location}}
      register: pub_key
    - name: create user-data
      raw: |
        cat >user-data <<EOF
        #cloud-config
        users:
          - name: {{username}}
            ssh-authorized-keys:
              - {{pub_key.stdout}}
            sudo: ['ALL=(ALL) NOPASSWD:ALL']
            groups: sudo
            shell: /bin/bash
        runcmd:
          - echo "AllowUsers {{username}}" >> /etc/ssh/sshd_config
          - restart ssh
        EOF
    - name: generate iso disk with cloud configuration
      command: genisoimage  -output {{target_dir}}/{{vm_name}}-cidata.iso -volid cidata -joliet -rock user-data meta-data
    - name: install virtual machine
      command: >
        virt-install --connect qemu:///system --virt-type kvm --name {{vm_name}} --ram {{ram_size}} --vcpus={{vcpus}} --os-type linux 
        --os-variant {{os_variant}} --disk path=/{{target_dir}}/{{vm_name}}.qcow2,format=qcow2 
        --disk {{target_dir}}/{{vm_name}}-cidata.iso,device=cdrom --import --network network=default --noautoconsole
    - pause:
        seconds: 30
    - name: get ip address from vm created
      command: virsh domifaddr {{vm_name}}
      register: vm_ip
    - debug:
        msg: "{{vm_ip}}"
~~~

Change the params base_dir, target_dir and rsa_key_location where are located your pool storage from libvirt.

Save this file with extensión YAML and execute in this way.

ansible-playbook create-vm-ansible.yaml -b -K

The current mandatory params are:

    vm_name: name of virtual machine
    base_image: Base cloud image for machine
    username: username that will be created
    hostname: hostname to use
    os_variant: variant of operating system, to know all variants use the command osinfo-query os


For example to create a centos 7 machine:
List the current base images in our storage:

~~~bash
└─▪ ls /var/lib/libvirt/images/cloud-images/base/
bionic-server-cloudimg-amd64.qcow2     manjaro-i3-20.0-200426-linux56.iso
CentOS-7-x86_64-GenericCloud.qcow2     ubuntu-18.04.4-live-server-amd64.iso
Fedora-Cloud-Base-31-1.9.x86_64.qcow2  ubuntu-20.04-desktop-amd64.iso
Fedora-Cloud-Base-32-1.6.x86_64.qcow2  Win10_1909_English_x64.iso
kali-linux-2020-W13-live-amd64.iso
~~~

Execute the playbook:

~~~bash
└─▪ ansible-playbook create-vm-ansible.yaml -b -K
BECOME password: 
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'
vm_name: centos
base_image: CentOS-7-x86_64-GenericCloud.qcow2
username: ventos
hostname: centos.lan
os_variant: centos7.0
disk_size [20G]: 
ram_size [4096]: 
vcpus [6]: 
PLAY [Create virtual machine in cloud format] ************************************************************
TASK [Gathering Facts] ***********************************************************************************
ok: [localhost]
TASK [create disk] ***************************************************************************************
changed: [localhost]
TASK [resize disk] ***************************************************************************************
changed: [localhost]
TASK [create meta-data] **********************************************************************************
changed: [localhost]
TASK [export pub key] ************************************************************************************
changed: [localhost]
TASK [create user-data] **********************************************************************************
changed: [localhost]
TASK [generate iso disk with cloud configuration] ********************************************************
changed: [localhost]
TASK [install virtual machine] ***************************************************************************
changed: [localhost]
TASK [pause] *********************************************************************************************
Pausing for 30 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [localhost]
TASK [get ip address from vm created] ********************************************************************
changed: [localhost]
TASK [debug] *********************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": true,
        "cmd": [
            "virsh",
            "domifaddr",
            "centos"
        ],
        "delta": "0:00:00.025770",
        "end": "2020-05-10 15:52:57.745066",
        "failed": false,
        "rc": 0,
        "start": "2020-05-10 15:52:57.719296",
        "stderr": "",
        "stderr_lines": [],
        "stdout": " Name       MAC address          Protocol     Address\n-------------------------------------------------------------------------------\n vnet0      52:54:00:c6:d4:21    ipv4         192.168.122.55/24",
        "stdout_lines": [
            " Name       MAC address          Protocol     Address",
            "-------------------------------------------------------------------------------",
            " vnet0      52:54:00:c6:d4:21    ipv4         192.168.122.55/24"
        ]
    }
}
PLAY RECAP ***********************************************************************************************
localhost                  : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

At the end the command virsh domifaddr shows us the ip from the virtual machine created.
Log in via ssh and check that vm is created successfully

~~~bash
└─▪ ssh ventos@192.168.122.55
The authenticity of host '192.168.122.55 (192.168.122.55)' can't be established.
ECDSA key fingerprint is SHA256:+kyBu52gBpRt+v05CY2fjBxkswdqg10Q+3SHisaLUGE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.55' (ECDSA) to the list of known hosts.
Enter passphrase for key '/home/user/.ssh/id_rsa': 
[ventos@centos ~]$ uname -a
Linux centos.lan 3.10.0-957.27.2.el7.x86_64 #1 SMP Mon Jul 29 17:46:05 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
[ventos@centos ~]$ 
~~~
