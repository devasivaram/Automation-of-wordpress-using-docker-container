# WordPress-with-docker-container-using-Ansible

## Description

Here we will build a multi-container WordPress installation using docker container with Ansible. Which includes a MySQL database container and WordPress container.

## Prerequisites

1. Need 2 Amazon LInux instances, one with ansible installed (Manager)
~~~sh
sudo amazon-linux-extras install ansible2 -y
~~~
2. Add second instnace (Worker) private IP to hosts file (Inventory file)
3. Purchase a domain name or point you domain name to server IP. You can use this to purchase new [domain_name](http://www.freenom.com/en/index.html).
4. Need to point A record for example.com pointing to your server’s (Worker) public IP address.

## Procedure

## 1. To configure Hosts file (Inventory file)

The orginal Hosts file for Ansible is "/etc/ansible/hosts", but am not using this file as my Hosts file, instead am creating new file with name "inventory". Here I used grouping in host file by declaring a variable in "[ variable ]".

~~~sh
vim inventory
~~~
>Added this
~~~
[amazon]
server-private-ip ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="private-key-file-name.pem"
~~~

>Make sure that you add the private ssh key file for your worker instance to the manager server and change its permission to "400"!

And run this command for checking SSH is working on worker server/instance:
~~~sh
ansible -i inventory amazon -m ping
~~~
Result:
>![image](https://user-images.githubusercontent.com/100773863/164958708-bcc24449-3660-4914-a771-3a4f8447ab3d.png)


## 2. Create playbook

We need to write the yml file for creating wordpress site using docker container:

~~~sh
vim docker-container
~~~

Added:
~~~
---
- name: "creating wordpress using ansible"
  become: true
  hosts: all
  vars:
    packages:
       - docker
       - python2-pip
    user: ec2-user
    db_volume: db_data
    wp_volume: wp_data
  tasks:
    - name: "Installing Docker"
      yum:
        name: "{{ packages }}"
        state: present
    
    - name: "Installing docker client for python"
      pip:
        name: docker-py
        state: present

    - name: "Restarting and enabling docker service"
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "Adding user "{{ user }}" to group docker"
      user:
        name: "{{ user }}"
        group: docker
        append: yes


    - name: "Launching mysql container"
      docker_container:
        name: database
        image: mysql:5.7
        recreate: true
        volumes:
          - "{{ db_volume }}:/var/lib/mysql"
        restart: true
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: test
          MYSQL_USER: wordpress
          MYSQL_PASSWORD: wordpress

    - name: "Launching wordpress container"
      docker_container:
        name: wordpress
        image: wordpress:latest
        recreate: true
        volumes:
          - "{{wp_volume}}:/var/www/html"
        restart: true
        env:
          WORDPRESS_DB_HOST: db
          WORDPRESS_DB_USER: wordpress
          WORDPRESS_DB_PASSWORD: wordpress
          WORDPRESS_DB_NAME: test
        ports:
          - "8080:80"

    - name: "network creation and adding container to network"
      docker_network:
        name: wpnet
        connected:
          - database
          - wordpress
~~~

## 3. Check syntax and execute playbook

Once the playbook is created, we need to check the syntax and execute the playbook:

~~~sh
# ansible-playbook -i hosts docker-container --syntax-check
# ansible-playbook -i hosts docker-container
~~~

Once those are successfull, we can access the site example.com.

## Conclusion

This is how we create wordpress site using docker-container with Ansible. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>









