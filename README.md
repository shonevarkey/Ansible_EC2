# Ansible_EC2

Ansible playbook to create an AWS EC2 instance.

### Requirements

The below requirements are needed on the host that executes this playbook.

- Install python boto boto3 and configure AWS ID and Key on the ansible master server.

```
---
- name: "Ansible Playbook to Create an EC2 Instance"
  hosts: localhost
  connection: local
  gather_facts: False
  vars:
    region: us-east-2
    security_group: ansible-sg
    instance_type: t2.micro
    image: ami-01aab85a5e4a5a0fe
    keypair: ansible-key
    count: 1
  tasks:
   
    - name: "Creating a keypair"
      ec2_key:
        name: ansible-key
        region: "{{ region }}"
        state: present
      register: ansible_private_key

    - name: "Copying Private key to a file"
      when: ansible_private_key.changed == true
      copy:
        content: "{{ ansible_private_key.key.private_key }}"
        dest: "./ansible_login.pem"
        mode: 0400

    - name: "Creating Security group"
      ec2_group:
        name: "{{ security_group }}" 
        description: Security Group for EC2
        region: "{{ region }}"
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 443
            to_port: 443
            cidr_ip: 0.0.0.0/0
        rules_egress:
          - proto: all
            cidr_ip: 0.0.0.0/0
      register: ansible_security

    - name: "Launching EC2 Instance"
      ec2:
        instance_type: "{{ instance_type }}"
        image: "{{ image }}"
        region: "{{ region }}"
        group: "{{ security_group }}"
        key_name: "{{ keypair }}"
        wait: true
        count: "{{ count }}"
        instance_tags:
           Name: IAC-ANSIBLE
      register: ansible_ec2

```

### The EC2 Instance has been created under your AWS account. 
