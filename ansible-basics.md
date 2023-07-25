### Basic Ansible
This was done on a home lab running Debian 11. `tesseract` is my control-node.

1. Add Ansible to Sources list
1. Update the OS Sources
1. Install Ansible
1. Create SSH keys
2. Tell ansible to use `ssh-agent` so you don't have to retype passwords
1. Use Ansible to copy the controle node SSH key to the ansible hosts
1. Use an Ansible playbook to ping the devices
1. Use an Ansible playbook to upgrade the devices

##### Add Ansible to Sources list
```
$ echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu focal main" | sudo tee /etc/apt/sources.list.d/ansible.list
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
$ sudo apt update
````

##### Install Ansible
```
$ sudo apt install ansible
```

##### Define hosts, Create Host file
Do not put special characters (like -) into the group names. Hosts should be FQDNs.

```
ariadne@tesseract:~/ansible$ cat /etc/ansible/hosts 
[proxmox]
agatha.haske.org
tiny-1.haske.org
tiny-2.haske.org
tiny-3.haske.org

[debian11]
k8s-control-agatha.haske.org
k8s-control-tiny-1.haske.org
k8s-worker-agatha.haske.org
k8s-worker-2.haske.org
k8s-worker-3.haske.org
tesseract.haske.org
orchard.haske.org
grove.haske.org

[k8s]
k8s-control-agatha.haske.org
k8s-control-tiny-1.haske.org
k8s-worker-agatha.haske.org
k8s-worker-2.haske.org
k8s-worker-3.haske.org
```

##### Define Defaults, Modify ansible.cfg
```
ariadne@tesseract:/etc/ansible$ cat ansible.cfg 
# [output omitted]

[defaults]
host_key_checking = False
remote_user = ariadne
```

##### Create a public SSH key to allow passwordless access

I'm using an internal linux host called `tesseract`. It doesn't use a password, it's a home lab.
```
ariadne@tesseract:~$ ssh-keygen -t rsa -b 4096 -C "ariadne@tesseract.haske.org"
```

##### Tell ansible you don't want to type a password constantly
I'm not sure if this is required. I did it, I'm including the instructions
```
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa
```

##### Write a playbook to copy the SSH keys
```
ariadne@tesseract:~/ansible$ cat copy_ssh_keys_test.yml 
---
- name: Copy SSH key to hosts
  hosts: all
  become: yes

  tasks:
  - name: Set authorized key taken from file
    authorized_key:
      user: ariadne
      state: present
      key: "{{ lookup(file, /home/ariadne/.ssh/id_rsa.pub) }}"
```

##### Run it
```
ariadne@tesseract:~/ansible$ ansible-playbook copy_ssh_keys.yml 

PLAY [Copy SSH key to hosts] ********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [agatha.haske.org]
ok: [tiny-3.haske.org]
ok: [tiny-2.haske.org]
ok: [tiny-1.haske.org]
ok: [k8s-control-agatha.haske.org]
ok: [tesseract.haske.org]
ok: [k8s-control-tiny-1.haske.org]
ok: [k8s-worker-3.haske.org]
ok: [k8s-worker-2.haske.org]
ok: [k8s-worker-agatha.haske.org]
ok: [orchard.haske.org]
ok: [grove.haske.org]

TASK [Set authorized key taken from file] *******************************************************************************************************************************************************************************
ok: [tiny-2.haske.org]
ok: [tiny-1.haske.org]
ok: [tiny-3.haske.org]
ok: [k8s-control-agatha.haske.org]
ok: [agatha.haske.org]
ok: [k8s-control-tiny-1.haske.org]
ok: [k8s-worker-agatha.haske.org]
ok: [k8s-worker-2.haske.org]
ok: [k8s-worker-3.haske.org]
ok: [tesseract.haske.org]
ok: [grove.haske.org]
ok: [orchard.haske.org]

PLAY RECAP **************************************************************************************************************************************************************************************************************
agatha.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
grove.haske.org            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-control-agatha.haske.org : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-control-tiny-1.haske.org : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-2.haske.org     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-3.haske.org     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-agatha.haske.org : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orchard.haske.org          : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tesseract.haske.org        : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tiny-1.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tiny-2.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tiny-3.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

##### Write a Playbook to Upgrade Everything
```
ariadne@tesseract:~/ansible$ cat upgrade-everything.yml 
---
- name: Update and upgrade apt packages
  hosts: all
  become: true
  tasks:
    - name: Update apt cache and upgrade all packages
      apt:
        upgrade: yes
        update_cache: yes
        cache_valid_time: 86400 #One day
```

##### Run it
```
ariadne@tesseract:~/ansible$ ansible-playbook upgrade-everything.yml 

PLAY [Update and upgrade apt packages] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [tiny-2.haske.org]
ok: [tiny-1.haske.org]
ok: [tiny-3.haske.org]
ok: [k8s-control-agatha.haske.org]
ok: [agatha.haske.org]
ok: [tesseract.haske.org]
ok: [k8s-control-tiny-1.haske.org]
ok: [k8s-worker-2.haske.org]
ok: [k8s-worker-3.haske.org]
ok: [k8s-worker-agatha.haske.org]
ok: [orchard.haske.org]
ok: [grove.haske.org]

TASK [Update apt cache and upgrade all packages] ************************************************************************************************************************************************************************
ok: [tiny-3.haske.org]
ok: [tiny-2.haske.org]
ok: [agatha.haske.org]
ok: [k8s-control-agatha.haske.org]
ok: [k8s-worker-agatha.haske.org]
ok: [k8s-worker-2.haske.org]
ok: [k8s-worker-3.haske.org]
ok: [tiny-1.haske.org]
ok: [tesseract.haske.org]
ok: [grove.haske.org]
ok: [orchard.haske.org]
ok: [k8s-control-tiny-1.haske.org]

PLAY RECAP **************************************************************************************************************************************************************************************************************
agatha.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
grove.haske.org            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-control-agatha.haske.org : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-control-tiny-1.haske.org : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-2.haske.org     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-3.haske.org     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-agatha.haske.org : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orchard.haske.org          : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tesseract.haske.org        : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tiny-1.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tiny-2.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
tiny-3.haske.org           : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
##### Sources

https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian
https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html
