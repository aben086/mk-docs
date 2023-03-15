---
layout: default
title: Prácticas 13.4
parent: Prácticas 13
nav_order: 4
---
# IAW - Práctica 13.4 Ansible
![](../assets/pr134/ansible.png)
En esta práctica se realizó la creación de la infraestructura necesaria para desplegar las aplicaciones web propuetas en la práctica 7 y 9 utilizando las funcionalidades de **Ansible**, de manera que, a través de AWS, se realice el aprovisionamiento de sus infraestructuras usando las plantillas **Ansible**.
## Estructura
```txt
.
├── practica7
│   ├── playbook
│   │   ├── eliminar_practica7.yaml
│   │   └── montar_practica7.yaml
│   └── variables
│       └── vars.yaml
├── practica9
│   ├── playbook
│   │   ├── eliminar_practica9.yaml
│   │   └── montar_practica9.yaml
│   └── variables
│       └── vars.yaml
└── README.md
```
Ambos directorios, **practica7** y **practica9** contienen un playbook tanto para montar la infrastructura necesaria como para desmontarla automáticamente, además de un archivo de **variables** en caso de que sea necesario cambiarlas.

# Práctica 7
```yaml
---
- name: Playbook Infrastructura Práctica 7
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:

    - name: Llamamos las variables
      ansible.builtin.include_vars:
        ../variables/vars.yaml

# Creación del grupo de seguridad de las máquinas FrontEnd
    - name: Creación de grupo de seguridad FrontEndSecurityGroup
      amazon.aws.ec2_group:
        name: "{{ frontend_sg }}"
        description: "{{ frontend_sg_desc }}"
        rules:
          - proto: tcp
            from_port: "{{ ssh_port }}"
            to_port: "{{ ssh_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ http_port }}"
            to_port: "{{ http_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ https_port }}"
            to_port: "{{ https_port }}"
            cidr_ip: "{{ network_01 }}"
      register: frontend_sec_grp

# Creación del grupo de seguridad de las máquinas BackEnd
    - name: Creación de grupo de seguridad BackEndSecurityGroup
      amazon.aws.ec2_group:
        name: "{{ backend_sg }}"
        description: "{{ backend_sg_desc }}"
        rules:
          - proto: tcp
            from_port: "{{ ssh_port }}"
            to_port: "{{ ssh_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ mysql_port }}"
            to_port: "{{ mysql_port }}"
            cidr_ip: "{{ network_01 }}"
      register: backend_sec_grp

# Creación de la instancia FrontEnd
    - name: Creación de la máquina FrontEnd
      amazon.aws.ec2_instance:
        name: "{{ instancia_frontend }}"
        key_name: "{{ key_name }}"
        security_group: "{{ frontend_sg }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ ami }}"
        state: present
      register: ec2_frontend


# Creación de la instancia BackEnd
    - name: Creación de la máquina FrontEnd
      amazon.aws.ec2_instance:
        name: "{{ instancia_backend }}"
        key_name: "{{ key_name }}"
        security_group: "{{ backend_sg }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ ami }}"
        state: present
      register: ec2_backend

    - name: Mostramos el contenido de las variables ec2
      ansible.builtin.debug:
        msg: "FrontEnd: {{ ec2_frontend }} \
              BackEnd: {{ ec2_backend }}"
```
# Práctica 9
```yaml
---
- name: Playbook Infrastructura Práctica 7
  hosts: aws
  become: true

  tasks:

    - name: Llamamos las variables
      ansible.builtin.include_vars:
        ../variables/vars.yaml

# Creación del grupo de seguridad del balanceador de carga
    - name: Creación de grupo de seguridad LoadBalancerSecurityGroup.
      amazon.aws.ec2_group:
        name: "{{ balancer_sg }}"
        description: "{{ balancer_sg_desc }}"
        rules:
          - proto: tcp
            from_port: "{{ ssh_port }}"
            to_port: "{{ ssh_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ http_port }}"
            to_port: "{{ http_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ https_port }}"
            to_port: "{{ https_port }}"
            cidr_ip: "{{ network_01 }}"
      register: balancer_sec_grp

# Creación del grupo de seguridad del servidor NFS
    - name: Creación de grupo de seguridad NFSServerSecurityGroup.
      amazon.aws.ec2_group:
        name: "{{ nfs_sg }}"
        description: "{{ nfs_sg_desc }}"
        rules:
          - proto: tcp
            from_port: "{{ ssh_port }}"
            to_port: "{{ ssh_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ nfs_port }}"
            to_port: "{{ nfs_port }}"
            cidr_ip: "{{ network_01 }}"
      register: nfs_sec_grp

# Creación del grupo de seguridad de las máquinas FrontEnd
    - name: Creación de grupo de seguridad FrontEndSecurityGroup.
      amazon.aws.ec2_group:
        name: "{{ frontend_sg }}"
        description: "{{ frontend_sg_desc }}"
        rules:
          - proto: tcp
            from_port: "{{ ssh_port }}"
            to_port: "{{ ssh_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ http_port }}"
            to_port: "{{ http_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ https_port }}"
            to_port: "{{ https_port }}"
            cidr_ip: "{{ network_01 }}"
      register: frontend_sec_grp

# Creación del grupo de seguridad de las máquinas BackEnd
    - name: Creación de grupo de seguridad BackEndSecurityGroup.
      amazon.aws.ec2_group:
        name: "{{ backend_sg }}"
        description: "{{ backend_sg_desc }}"
        rules:
          - proto: tcp
            from_port: "{{ ssh_port }}"
            to_port: "{{ ssh_port }}"
            cidr_ip: "{{ network_01 }}"
          - proto: tcp
            from_port: "{{ mysql_port }}"
            to_port: "{{ mysql_port }}"
            cidr_ip: "{{ network_01 }}"
      register: backend_sec_grp


# Creación de la instancia LoadBalancer
    - name: Creación de la máquina balanceador de carga.
      amazon.aws.ec2_instance:
        name: "{{ instancia_balancer }}"
        key_name: "{{ key_name }}"
        security_group: "{{ balancer_sg }}"
        instance_type: "{{ instance_type_med }}"
        image_id: "{{ ami }}"
        state: running
      register: ec2_balancer

# Creación de la instancia NFS Server
    - name: Creación de la máquina servidor NFS.
      amazon.aws.ec2_instance:
        name: "{{ instancia_nfs }}"
        key_name: "{{ key_name }}"
        security_group: "{{ nfs_sg }}"
        instance_type: "{{ instance_type_med }}"
        image_id: "{{ ami }}"
        state: running
      register: ec2_nfs

# Creación de la instancia FrontEnd01
    - name: Creación de la máquina FrontEnd01.
      amazon.aws.ec2_instance:
        name: "{{ instancia_frontend_01 }}"
        key_name: "{{ key_name }}"
        security_group: "{{ frontend_sg }}"
        instance_type: "{{ instance_type_sma }}"
        image_id: "{{ ami }}"
        state: running
      register: ec2_frontend01

# Creación de la instancia FrontEnd02
    - name: Creación de la máquina FrontEnd02.
      amazon.aws.ec2_instance:
        name: "{{ instancia_frontend_02 }}"
        key_name: "{{ key_name }}"
        security_group: "{{ frontend_sg }}"
        instance_type: "{{ instance_type_sma }}"
        image_id: "{{ ami }}"
        state: running
      register: ec2_frontend02

# Creación de la instancia BackEnd
    - name: Creación de la máquina FrontEnd.
      amazon.aws.ec2_instance:
        name: "{{ instancia_backend }}"
        key_name: "{{ key_name }}"
        security_group: "{{ backend_sg }}"
        instance_type: "{{ instance_type_sma }}"
        image_id: "{{ ami }}"
        state: running
      register: ec2_backend

    - name: Mostramos el contenido de las variables ec2.
      ansible.builtin.debug:
        msg: "FrontEnd01: {{ ec2_frontend01 }} \
              FrontEnd01: {{ ec2_frontend02 }}
              BackEnd: {{ ec2_backend }} \
              Load Balancer: {{ ec2_balancer }} \
              NFS Server: {{ ec2_nfs }}"
```
