# Kubernetes Cluster Creator
I got tired of manually creating Kubernetes clusters in my lab so i created this playbook to streamline and speed up the process

This playbook creates a single master node and any number of worker nodes automatically. It will automatically install the `Calico CNI` infrastructure. More will be added later. 

`Ingress` controller is not yet included but will be in the near future.

`Storage` is not included and will need to be configured by you. `Kadalu` will be included as an optional storage infrastructure in the future

## Requirements
### Operating System
- Master and Worker Nodes
    
    Tested and running on `Rocky Linux 8`
    - Redhat 8
    - Centos 8
    - Rocky Linux 8
    - AlmaLinux 8
- Bastion or Workstation host
    - Any host that supports running of Ansible playbooks
### Ansible Support
Tested and running on `ansible-playbook 2.9.6`
- Requires 2 collections to be installed
```
$ ansible-galaxy collection install community.general
$ ansible-galaxy collection install ansible.posix
```
## How does this playbook/roll work?
- Fully updates `Master` and `Worker` nodes
- Installs required packages required to run/install K8s and containerd
- Installs Containerd
- Installs Kubernetes version of your choice
- Configures Kubernetes master node
- Configures Kubernetes worker nodes
## How to Use this playbook

1. Provision your master and worker nodes by deploying appropriately specced and supported operating systems. 
    - See `Operating System` support for details.

2. Clone or download this project to your workstation or bastion host
    ```
    $ git clone https://github.com/****
    ```

3. Configure your `/etc/hosts` file on your `bastion or workstation` host
```
192.168.2.70 k8smaster.example.com k8smaster01

192.168.2.71 k8snode01.example.com k8snode01
192.168.2.71 k8snode02.example.com k8snode02
192.168.2.71 k8snode03.example.com k8snode03
```

4. Add the `short` hostname of each of your master and worker nodes to the `hosts` file in the root of this project.
```
[k8snodes]
k8smaster
k8snode1
k8snode2
k8snode3
```

5. Configure deployment options by editing the variables in `k8s-base.yml`
```
- name: K8S Setup Base
  hosts: k8snodes
  remote_user: root
  become: yes
  become_method: sudo
  vars:
    k8s_version: "1.22.3-0"             # Use the long k8s version numbers. IE: "1.22.3-0"
    selinux_state: permissive           # Using a restrictive selinux may require configs not in playbook
    timezone: "America/Chicago"         # Choose your timezone
    k8s_cni: calico                     # Choose your CNI. NOTE: Calico is only supported at this time
    container_runtime: containerd       # Choose your CR: NOTE: Containerd is only supported at this time
    control_plane_ip: "192.168.2.70"    # Set as the IP address of your Master node
    configure_firewalld: false          # Configure Firewall Rules. NOTE: Leave false for now as it breaks K8s
    kube_purge_first: false             # Purge K8s first. Currently not in use
  roles:
    - kubernetes-stack
```

6. Create SSH key and copy to your `master` and `worker` nodes.
    - Note: I'm using root here. `Terrible idea i know.` I'm working on a better way of doing this securely

7. Test that you can ssh to each machine and get a shell before continuing. 

8. Run this playbook.
```
$ ansible-playbook -i hosts k8s-base.yml
```
9. Configure your storage and enjoy using your Kubernetes cluster.